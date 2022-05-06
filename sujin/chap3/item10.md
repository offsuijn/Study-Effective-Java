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
