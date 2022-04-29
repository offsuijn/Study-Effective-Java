## [아이템 3] private 생성자나 열거 타입으로 싱글턴임을 보증하라

`**싱글턴(singleton)**` : **인스턴스를 오직 하나**만 생성할 수 있는 클래스 

**싱글턴 클래스의 Mocking을 가능하게 하기 위해서는?** 

Mocking이란 클라이언트가 코드를 테스트 할 때 주로 쓰는 방법으로 테스트할 코드의 가짜 인스턴스를 생성하여 해당 인스턴스로 테스트를 진행하는 방법이다. 

만약 인스턴스를 오직 하나만 생성하도록 막아버리면 이러한 Mocking이 어려워지는데, 이를 해결하기 위해서는 **타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴**을 제공하도록 해야한다. 이렇게 하면 클라이언트가 Mocking시 인터페이스를 구현하여 Mock 객체를 만들어낼 수 있다.

### 싱글턴을 만드는 방법

1. `**public static final` 필드 방식의 싱글턴** 

클라이언트가 접근할 수 있는 유일한 `public` 형태는 미리 정의된 인스턴스 하나 뿐이다. 해당 인스턴스에 대한 생성자는 `private` 접근제한을 걸고 클래스가 초기화될 때 딱 한 번 호출된다. 

```java
public class Elvis{
	public static final Elvis Instance = new Elvis();
	private Elvis() {..} // 생성자
}
```

**장점**

이 방법의 장점은 **해당 클래스가 싱글턴임이 API에 명백히 드러난다는 것** 이다. 또한 코드가 간결하다는 장점이 있다. 

이렇게 만들면 클라이언트는 인스턴스를 조작할 수 없게 되는데 한 가지 예외가 있다.

**예외) 리플렉션 API `AccessibleObject.setAccessible`**

이 메서드는 클라이언트로 하여금 **private 생성자를 호출할 수 있도록**한다. 

```java
public static void checkAndFixAccess(Member paramMember) {
    AccessibleObject localAccessibleObject = (AccessibleObject) paramMember;
    try {
        localAccessibleObject.setAccessible(true);
        return;
    } catch (SecurityException localSecurityException) {
        while (localAccessibleObject.isAccessible())
            ;
```

이러한 리플렉션을 방지하기 위해서는 생성자에 `final`인스턴스 외의 인스턴스를 생성하려고 하면 Exception을 발생시키도록 코드를 구현하면 된다. → 생성자에 final 인스턴스가 null 이 아닌 경우에는 Exception을 낸다. 

1. **정적 팩터리 방식의 싱글턴** 

이 방법은 위의 `public final static` 인스턴스를 `public`이 아닌 `private` 로 접근권한을 변경하고 그 대신 유일한 `public`접근 허용으로 `getInstance()`메서드를 통해 private인 인스턴스를 리턴하여 주는 것이다.

이 방법도 마찬가지로 리플렉션을 방어하기 위한 처리는 따로 해주어야 한다.

```java
public class Elvis{
	private static final Elvis Instance = new Elvis();
	private Elvis(){...} // 생성자
	public static Elvis getInstance(){return Instance;}
}
```

**장점**

이 방법은 **API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다**는 점이다. 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다. 

또한 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. 

```java
public class GenericFactoryMethod { 
	public static final Set EMPTY_SET = new HashSet(); 

	public static final <T> Set<T> emptySet() { 
		return (Set<T>) EMPTY_SET; } 
	}
}
```

**마지막으로 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.** 

참고) Supplier는 인자를 받지 않고 Type T 객체를 리턴하는 함수형 인터페이스이다.

```java
public interface Supplier<T> {
    T get();
}
```

→ Q. 왜 공급자로 사용하는게 장점이 되는가?

이러한 정적 펙토리 메서드를 사용하는 장점이 필요가 없다면 public 필드로 인스턴스를 받는 것이 좋다.

**위 두 방법에서의 직렬화**

위 두 방법은 직렬화할 때 처리가 까다롭다. 

우선 기존의 방법으로 직렬화를 하는 것으로 해결이 안되고 모든 인스턴스 필드를 `trainsient` 선언해서 Serialize 하는 과정에서 제외시켜야 한다. (그렇지 않으면 private 처리된 필드까지 모두 데이터화 되어서 private의 의미가 없어지는 듯,,?)

또한 `readReslove`처리도 해주어야 한다. 역직렬화 과정에서 인스턴스가 새로 만들어지는 것을 방지하고 기존에 생성된 싱글톤 인스턴스를 반환하도록 해야 한다. 

따라서 직렬화를 위해서는 아래와 같이 코드를 짜야한다.

참고) `trainsient`는 `static`과 마찬가지로 접근제어자 다음에 선언해주면 된다. 

```java
private Object readReslove(){
	//진짜 Elvis만 반환, 가짜는 가비지 컬렉터로 자동으로 맡겨짐
	return INSTANCE;
}
```

1. **열거 타입 방식의 싱글턴 - 바람직한 방법**

enum 타입으로 싱글톤을 만들 수 있다.

```java
public enum SingletonEnum {
    INSTANCE;
    int value;
    
    public int getValue() {
        return value;
    }
    public void setValue(int value) {
        this.value = value;
    }
}
public class EnumDemo {
    
    public static void main(String[] args) {
        SingletonEnum singleton = SingletonEnum.INSTANCE;
        
        System.out.println(singleton.getValue());
        singleton.setValue(2);
        System.out.println(singleton.getValue());
    }
}
```

이 경우 아예 생성자 자체가 없기 때문에 싱글톤 형태가 항상 유지되며 추가 노력없이 직렬화할 수 있고 리플렉션 공격도 방어할 수 있다.

**따라서 이 방법이 가장 추천되는 방법이다**

그러나 만들려는 싱글턴이 Enum 외의 클래스를 상속하게 된다면 이 방법을 사용할 수 없다. 
