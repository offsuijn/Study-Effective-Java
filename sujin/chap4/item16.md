## 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### public 필드 사용
```java
class Point {
    public double x;
    public double y;
}
```
- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.


### 접근자와 변경자 메소드 사용
```java
class Point {
  private double x;
  private double y;

  public Point2(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() {
    return x;
  }

  public double getY() {
    return y;
  }

  public void setX(double x) {
    this.x = x;
  }
  public void setY(double y) {
    this.y = y;
  }
}
```
- private field를 사용하여 직접적인 접근을 막는다.
- getter와 setter를 통해 내부 표현 방식의 유연성을 얻는다.
- package-private(default class) 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.

## 정리
- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. (item 15의 배열의 경우)
- package-private 클래스나 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.
