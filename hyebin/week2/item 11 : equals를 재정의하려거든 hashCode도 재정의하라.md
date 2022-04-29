## [아이템 11] equals를 재정의하려거든 hashCode도 재정의하라

`equals`를 재정의한 클래스 모두에서 `hashCode`도 재정의해야 한다. 

논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여 규약과 달리 서로 다른 값을 반환한다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707,867,5309),"제니"); 

//이후 
m.get(new PhoneNumber(707,867,5309)) // null 반환 
```

위 코드와 같이 논리적으로 같은 객체이지만 물리적 객체가 다르면 hashCode가 다르기 때문에 key 값으로서의 역할을 못하게 된다. Hashmap 입장에서는 그저 다른 객체 두 개일 뿐인 것이다. 

이런 문제를 해결하기 위해 HashCode 메서드를 수정해야 한다. 

### HashCode 작성하는 요령

1. `int` 변수 `result`를 선언한 후 값을 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다. (이때 핵심 필드란 equals 메서드에서 비교할 필드를 말한다.)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면 Type.hashCode(f)를 수행한다. 
        2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals 를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null 이면 0을 사용한다. 
        3. 필드가 배열이라면 핵심 원소 각각을 별도의 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음 단계 2.b 방식으로 갱신한다. **모든 원소가 핵심 원소라면 Arrays.hashCode**를 사용한다. 
    2. 단계 2.a 에서 계산한 해시코드로 c를 result로 갱신한다. 코드는 다음과 같다.
    
    ```java
    result = 31 * result + c;
    ```
    
3. result를 반환한다.

```java
@Override public int hashCode(){
//PhoneNumber의 filde는 short 타입의 areaCode, prefix, lineNum
	 int result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		return result;
}
```

**성능에 민감하지 않다면 코드가 더 깔끔한 `hash`를 사용할 수도 있음**

```java
@Override public int hashCode(){
	return Objects.hash(lineNum, prefix, areaCode);
}
```

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다. 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다. 

캐싱은 아예 해당 인스턴스의 필드로 hashCode를 인스턴스가 갖고 있게 하는 방식으로 구현한다.

```java
private int hashCode; 

@Override public int hashCode(){
	int result = hashCode;
	if( result = 0){ // 해시코드 생성 안됨
	  result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}

```

**성능을 높인답시고 핵심 필드를 해시코드 계산에서 생략하면 안된다**

AutoValue를 사용하면 `equals` 뿐만 아니라 `hashCode` 메서드까지 자동생성된다.

릴리즈할 때는 hashCode 메서드를 클라이언트에게 공표하지 말아야 한다.
