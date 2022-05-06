## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
싱글톤(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.

**싱글턴을 만드는 세 가지 방법**
### 1. public static final 필드 방식
private 생성자는 Sujin.INSTANCE를 초기화할 때 딱 한번만 호출된다.
```java
public class Sujin {
    public static final Sujin INSTANCE = new Sujin();

    private Sujin() { }
}
```
### 2. 정적 팩터리 방식
```java
public class Sujin {
    private static final Sujin INSTANCE = new Sujin();
    private Sujin() { }
    public static Sujin getInstance() { return INSTANCE; }
}
```
장점
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 정적 팩터리를 제너릭 싱글턴 팩터리로 만들 수 있다.

### 3. 열거 타입 방식
public 방식과 비슷하지만 간결하고 직렬화 문제도 막아준다.
```java
public enum Sujin {
    INSTANCE;
}
```
👉 **대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다!**
