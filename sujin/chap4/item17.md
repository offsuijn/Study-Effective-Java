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

### 예시: 불변 복소수 클래스
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
- 불변 객체는 **Thread-safe**하여 따로 동기화할 필요가 없다는 장점이 있다.
