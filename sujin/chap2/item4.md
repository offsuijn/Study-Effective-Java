
## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
인스턴스로 만들어 쓰려고 설계하지 않은 클래스에 생성자를 명시하지 않으면 자동으로 기본 생성자가 만들어진다.
클래스의 인스턴스화를 막기 위해서는 private 생성자를 추가해야 한다.

```java
public class SujinUtil {
    private SujinUtil() {
        throw new AssertionError();
    }
}
```
클래스 안에서 실수로라도 생성자를 호출하지 않도록 오류를 던져준다.

이 방식을 사용하면 상속도 방지할 수 있다.
