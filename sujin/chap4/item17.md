## 아이템 17. 변경 가능성을 최소화하라

### 불변 클래스
   - 인스턴스의 내부 값을 수정할 수 없는 클래스를 말한다.
   - 대표적으로 String, BigInteger, BigDecimal 등이 이에 속한다.
   - 사용 이유
      - 설계, 구현, 사용이 용이하다.
      - Thread-safe하고 오류가 생길 여지가 적다.

### 불변 클래스를 만들 때 지켜야할 5가지 규칙
#### 1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.

#### 2. 클래스를 확장할 수 없도록 한다.

  - 객체의 상태를 변하게 만드는 사태를 막아준다.
  - 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이다.

#### 3. 모든 필드를 final로 선언한다.

  - 필드의 수정을 막겠다는 설계자의 의도를 드러내는 방법이다.

#### 4. 모든 필드를 private으로 선언한다.

  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근하여 수정하는 일을 막아 준다.

#### 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

#### 예시: 불변 복소수 클래스
```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re * c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
- 실수부와 허수부 값을 반환하는 접근자 메서드가 정의되어 있고, 사칙연산을 위한 메서드들이 제공되고 있다.
- 사칙연산 메서드들은 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다.
- 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로이다.(함수형 프로그래밍)
   - 해당 메서드가 객체의 값을 변경하지 않는다.

### 불변 객체의 장점
- 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- Thread-safe하여 따로 동기화할 필요가 없다.
- 안심하고 공유할 수 있다.

### 불변 객체의 단점
값이 다르면 반드시 독립된 객체로 만들어야 한다.
- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용이 들게 된다.

### 불변 클래스를 만드는 또 다른 설계 방법
-> 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩토리를 제공한다.

```java
public class Complex {
    private final double re;
    private final double im;
    
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    
    //나머지 코드 생략
}
```
- final 클래스를 없애고, 생성자를 private으로 바꾼 뒤 정적 팩토리 메서드를 통해 불변 객체를 제공하고 있다.
- 패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final 클래스와 이다.
   - public이나 protected 생성자가 없으니 다른 패키지에서 이 클래스를 확장할 방법이 없기 때문이다.


## 정리
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

- **다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.**
