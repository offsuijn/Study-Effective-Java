## 아이템 22. 인터페이스는 타입을 재정의하는 용도로만 사용하라

인터페이스는 타입을 정의하는 용도로만 사용해야한다.
- 인터페이스를 구현하는 클래스를 만들게 되면, 그 인터페이스는 해당 클래스의 객체를 참조할 수 있는 자료형 역할을 하게 된다.
- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 말해주는 것이다.

### 상수를 공개하는 수단으로 사용해서는 안 된다!
#### 예시: 상수 인터페이스
```java
public interface PhysicalConstants {

    // 아보가드로 수
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // 볼츠만 상수
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // 전자 질량
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
- 메서드 없이 상수를 뜻하는 static final 필드로만 가득찬 인터페이스
- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부 구현이다.
   - 따라서 내부 구현을 클래스의 API로 노출하는 행위가 된다.

### 상수를 공개하는 대안
#### 1. 상수가 특정 클래스나 인터페이스에 강하게 연관되어 있다면 그 클래스나 인터페이스 자체에 추가해야 한다.
ex) Integer, Double의 MIN_VALUE, MAX_VALUE 상수

#### 2. Enum 타입
#### 3. 유틸리티 클래스
```java
public class PhysicalConstants {

    private PhysicalConstants(){} // 객체 생성을 막음

    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
- 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 한다.
   - ex) Integer.MAX_VALUE
- 유틸리티 클래스의 상수를 빈번히 사용한다면 정적 임포트(static import)하여 클래스 이름을 생략할 수 있다.
