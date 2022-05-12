## 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라
#### 태그 달린 클래스
- 두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스

### 태그 달린 클래스의 단점
- 열거 타입 선언, 태그 필드 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합돼 있어서 가독성이 나쁘다.
- 다른 의미를 위한 코드도 항상 함께하니 메모리도 많이 사용한다.
- 또 다른 의미를 추가하려면 코드를 수정해야 한다.

**<태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적이다.>**

### 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법

#### 1. 계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
#### 2. 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가한다.
#### 3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.

```java
// 1번
abstract class Figure {
    abstract double area();
}

// 3번
class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

// 3번
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}

```

### 클래스 계층구조의 장점
- 간결하고 명확하며, 쓸데없는 코드가 사라졌다.
- 루트 클래스의 코드를 건드리지 않고도 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다.
