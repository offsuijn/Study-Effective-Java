# 3장 모든 객체의 공통 메서드
Objects의 non-final 메서드에 대해 알아봅니다.

## 아이템 10. equals는 일반 규약을 지켜 재정의하라

### equals를 재정의하지 않아야하는 경우
- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 '논리적 동치성'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 privated이거나 package-private이고 equals 메서드를 호출할 일이 없다.

### equals를 재정의해야하는 경우
'논리적 동치성'을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때

### equals 메소드 재정의할 때 지켜야 할 일반 규약
#### 1. 반사성
객체는 자기 자신과 같아야 한다.
x.equals(x) = True

#### 2. 대칭성
두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
x.equals(y) = True, y.equals(x) = True
 
ex) CaseInsensitiveString.equals(String) != String.equals(CaseInsensitiveString)
 
#### 3. 추이성
x.equals(y) = True, y.equals(z) = True, x.equals(z) = True

ex) ColorPoint1.equals(Point) = True, Point.equals(ColorPoint2) = True, ColorPoint1.equals(ColorPoint2) = False

**구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**
**-> 상속 대신 컴포지션을 사용하면 가능하다.**

#### 4. 일관성
두 객체가 같다면 앞으로도 영원히 같아야 한다.
x.equals(y) = True.. True.. True...

ex) URL의 equals는 비교대상인 IP주소를 알기위해서 네트워크를 통과해야 하므로 일관성을 지키지 못할 수도 있다.

#### 5. null-아님
모든 객체가 null과 같지 않아야 한다.
x.equals(null) = False

**instanceof를 사용하면 null일 때 false를 반환해주므로 null 검사를 명시적으로 하지 않아도 된다.**

### 양질의 equals 메서드 구현 방법
- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
   - 자기 자신이면 true를 반환한다. 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.
- instanceof 연산자로 입력이 올바른 타입인지 확인한다.
   - 가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수도 있다.
   - 이런 인터페이스를 구현한 클래스라면 equals에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.
   - ex) Set, List, Map, Map.Entry 등 컬렉션 인터페이스들
- 입력을 올바른 타입으로 형변환 한다.
   - 2번에서 instanceof 연산자로 입력이 올바른 타입인지 검사 했기 때문에 이 단계는 100% 성공한다.
- 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
   - 모두 일치해야 true를 반환한다.
   
```java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this) {
            return true;
        }

        if(!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```

## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라
그렇지 않으면 인스턴스를 HashMap 이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

### hashCode 일반 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
   (애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
   논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
   → hashCode 재정의를 잘못 했을 때 문제가 되는 조항
- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
   하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### 좋은 hashCode 작성 요령
1. int 변수인 result 를 선언한 후 값을 c로 초기화한다.
	- 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
	- 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
	1. 해당 필드의 해시코드 c 를 계산한다.
		- 기본 타입 필드라면, Type.hashCode(f) 를 수행한다. 여기서 Type은 해당 기본타입의 박싱클래스다.
		- 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals 를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode 를 재귀적으로 호출한다.
		- 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 모든 원소가 핵심 원소라면 Arrays.hashCode 를 사용한다.
	2. 단계 2.1에서 계산한 해시코드 c로 result 를 갱신한다.
		- result = 31 * result + c;
3. result 를 반환한다.

```java
@Override
public int hashCode() {
	int result = Integer.hashCode(areaCode); // 1번
	result = 31 * result + Integer.hashCode(prefix); // 2번
	result = 31 * result + Integer.hashCode(lineNum); // 2번
	return result;
}
```

31을 곱하는 이유는 비슷한 필드가 여러개일 때 해시효과를 크게 높여주기 위해서다.
비슷한 값들이 여러개 있을때 그냥 더하면 같은 값이 나올 확률이 높다.
그래서 31을 곱해 큰수로 만들어 해시효과를 증가시킨다.


### hashCode의 캐싱과 지연 초기화
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 한다.
→ 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.

- 해시의 키로 사용되지 않는 경우라면 hashCode 가 처음 불릴 때 계산하는 지연 초기화를 사용하는 것이 좋다.
→ 필드를 지연 초기화 하려면 그 클래스가 thread-safe가 되도록 동기화에 신경 쓰는 것이 좋다.

## 아이템 12. toString을 항상 재정의하라
Object의 기본 toString 메서드는 PhoneNumber@adbbd처럼 단순히 `클래스_이름@16진수로_표시한_해시코드`를 반환한다.
toString의 일반 규약에 따르면 '간결하고 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야한다.
toString의 규약은 '모든 하위 클래스에서 이 메서드를 재정의하라'고 합니다.

### toString 재정의
- toString을 잘 구현한 클래스는 사용하기 편하고 디버깅하기 쉽다.
   - {Jenny=PhoneNumber@adbbd} 보다는 {Jenney=707-867-5309}라는 메세지가 유용하다.
   
- 실전에서 toString 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
   - 하지만 객체가 거대하거나 객체의 상태가 문자열로 표현하기 어려운 경우는 제외한다.

- toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다.
   - 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
   - 하지만 포맷을 한번 명시하면, 평생 그 포맷에 얽매이게 된다.
   
- 포맷을 명시하든 아니든 주석을 이용하여 개발자의 의도는 명확히 밝혀야한다.

- 포맷 명시 여부와 상관없이 toString이 반환한 값에 대해 포함된 정보를 얻어올 수 있는 API를 제공해야한다.
   - Getter 제공하지 않으면 개발자들은 toString을 파싱해서 사용해야한다.

### toString 재정의가 필요치 않는 경우
- 정적 유틸리티 클래스 - 제공할 이유가 없음
- enum Type - 이미 제공하고 있음
- 대다수의 컬렉션 구현체 - 추상 컬렉션 클래스의 toString을 상속해 쓰고있음
- 구글의 @Autovalue, Lombok의 @ToString
