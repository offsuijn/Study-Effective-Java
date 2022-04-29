## [아이템 10] equals는 일반 규약을 지켜 재정의하라

equals는 단순히 객체를 비교하는 때 뿐만 아니라 `contains` 메서드가 작동할 때도 작동하므로 주의해주어야 한다.

**재정의하지 않는 것이 좋은 상황** 

1. 각 인스턴스가 본질적으로 고유한 경우 : 값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스
2. 인스턴스의 논리적 동치성을 검사할 일이 없는 경우 : 애초에 논리적으로 같은지 여부를 판단할 필요가 없는 인스턴스의 클래스는 재정의할 필요가 없음
3. 상위 클래스에서 재정의한 equals를 하위 클래스에서 사용하면 되는 경우: 굳이 하위 클래스에서 또 재정의하지 않는다.
4. 클래스가 private 이거나 package private 이고 equals 메서드를 호출할 일이 없을 때 

```java
@Override public boolean equals(Object o){
	throw new AssertionError(); 
}
```

아예 예외 처리를 해서 equals 메서드 호출을 막아버린다.

**equals를 재정의해야 하는 상황** 

**객체 식별성(물리적으로 같은지)이 아닌 논리적 동치성(갖고 있는 값이 같은지) 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때**이다. 

주로 **값 클래스**들이 여기에 해당한다. `Integer`나  `String`같이 특정 값을 표현하는 클래스들을 말한다. 이 두 인스턴스를 `equals`로 비교할 때는 객체가 같은 객체인지를 보고자 하는 것이 아니라 객체가 담고 있는 내용이 같은지를 판단하고 싶을 때이다.

**값 클래스여도 같은 인스턴스가 여러개 만들어지지 않도록 정적 팩터리 메서드를 사용하는 클래스거나 Enum 인 경우 재정의할 필요가 없다.**

**equals 규약 지키기**

equals 규약을 지키지 않으면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 알 수 없다. 

1. **대칭성** 

대칭성의 핵심은 **두 객체가 서로에 대한 동치 여부에 똑같이 답해야 한다는 것**이다. 다시 말해 객체 A와 객체 B가 있다고 할 때, `A.equals(B)`와 `B.equals(A)`는 모두 같은 값을 내야 한다.

이를 어기는 예시 코드는 아래와 같다.

```java
public final class CaseInsensitiveString{
	private final String s;

	public CaseInsensitiveString(String s){
		this.s = Objects.requireNonNull(s);
	}
 //대칭성 위배
	@Override public boolean equals(Object o){
		if(o instance of CaseInsensitiveString) return s.equalsIgnoreCase{
		((CaseInsensitiveString) o).s);
		if(o instance of String) // 한방향으로만 작동
		return s.equalsIgnoreCase((String) o);
		return false;
}
}
```

위 코드는 `caseInsensitiveString.equals(string)`은 true를 반환하지만 `string.equals(caseInsensitiveString)`은 false를 반환하게 된다. 

CaseInsensitiveString 클래스의 equals는 string을 같은 값으로 인식하도록 재정의 되었지만 String 클래스의 equals는 그렇지 않기 때문이다.

따라서 이 경우 **다른 타입의 클래스끼리 equals로 비교하는 것은 웬만하면 시도하지 않는 것이 좋다.** 특히 String과 같은 사용자 정의 클래스가 아닌 클래스는 다른 타입과 equals로 값을 비교하는 것이 불가능하다.

```java
@Override public boolean equals(Object o){
	return o instanceof CaseInsensitiveString && 
	((CaseInsensitiveString)o)s.equals.IgnoreCase(s);
}
```

위의 코드처럼 equals는 CaseInsensitiveString 클래스끼리만 비교하도록 설계하는 것이 좋다.

1. **추이성** 

추이성은 A=B이고 B=C 이면 C=A 가 성립해야 한다는 뜻이다. **상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황에서 추이성을 어기기 쉽다.**

```java
public class Point {
	private final int x;
	private final int y;

	public Point (int x, int y){
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o){
	if(!(o instanceof Point) return false;
	Point p = (Point) o;
	return p.x == x && p.y == y;
}
}
```

```java
public class ColorPoint extends Point{
	private final Color color;

	public ColorPoint(int x, int y, Color color){
		super(x,y);
		this.color = color;
	}
}
```

여기서 만약 ColorPoint 의 eqauls 메서드를 재정의 안하고 그대로 둔다면 ColorPoint의 color 는 무시한 채 비교하게 될 것이다. 

그래서 ColorPoint 인 경우는 color까지 비교하도록 equals를 짠다면 아래와 같아질 것이다.

```java
@Override public boolean eqauls(Object o){
	if(!(o instanceof ColorPoint)) return false;
		return super.eqauls(o) && ((ColorPoint)o).color == color;
}
```

이런 경우 point.equals(colorpoint) 는 true, colorpoint.equals(point)는 false를 반환하는 대칭성 위배 문제가 생긴다.

ColorPoint가 equals에서 Point인 경우 color를 무시하게 코드를 짤 경우 추이성이 깨진다.

```java
@Override public boolean equals(Object o){
	if(!(o instanceof Point)) return false;
	if(!(o instanceof ColorPoint)) return o.eqauls(this);
	return super.equals(o)&&((ColorPoint)o).color == color;
}
```

같은 위치의 세 점 A, B,C 중 A는 그냥 Point고 B와 C의 색이 다르다면 A = B이고 A=C이지만 B≠C이기 때문이다.

**해결 방법은 없다, 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재X
무조건 같은 클래스끼리만 equals로 비교해야 함**

이 모든 규약을 지키며 **상위 클래스 컬렉션의 `contain`메서드에서 하위클래스도 유효하려면?**

하위 클래스는 상위 클래스를 상속하는 대신 `final` 필드로 상위 클래스의 객체를 가진다. 

또 상위 클래스 필드만 리턴해주는 View 메서드를 구현한다.

```java
public class ColorPoint{
	private final Point point;
	private final Color color;

	public ColorPoint(int x, int y, Color color){
		point = new Point(x,y);
		this.color = Objects.requireNonNull(color);
	}

public Point asPoint(){ return point;}

@Override public boolean equals(Object o){
	if(!(o instanceof ColorPoint)) return false;
	ColorPoint cp = (ColorPoint)o;
	return cp.point.equals(point)&&cp.color.equals(color);
}
```

1. **일관성**

equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다 : 예컨대 `java.net.URL`의 `equals`메서드는 호스트 IP에 따라 값이 달라지기 때문에 쓰지 않는 것이 좋다.

### ⭐⭐Equals 메서드 재정의하는 방법 ⭐⭐ (사실 이거만 봐도 될 듯)

1. `==`연산자를 활용해서 입력이 자기 자신의 참조인지 확인한다. 
    
    성능 향상의 효과를 기대할 수 있다.
    
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다. 
    
    그렇지 않다면 false를 반환한다. equals 재정의 규약에 따라 반드시 같은 타입(클래스)의 인스턴스만 비교하도록 한다.
    
    ```java
    if(!(o instanceof ColorPoint)) return false;
    ```
    
3. 입력을 올바른 타입으로 형변환한다.
    
    앞서 `instanceof`를 했기 때문에 여기서 오류가 날 가능성은 없다.
    
    ```java
    ColorPoint cp = (ColorPoint)o;
    ```
    
4. 입력 객체와 자기 자신의 대응되는 **핵심 필드**들이 모두 일치하는지 **하나씩 검사한다.** 
    
    비교해야 하는 필드를 하나씩 모두 검사하라는 이야기이다.
    
    ```java
    return cp.point.equals(point)&&cp.color.equals(color);
    ```
    

**인스턴스 비교 주의사항** 

`float`와 `double`을 제외한 기본 타입 필드는 `==`연산자로 비교한다.

참조 타입 필드는 각각의 `equals`메서드로 비교한다.

`float`와 `double`필드는 각각 정적 메서드인 `Float.compare(float,float)` , `Double.compare(double,double)`로 비교한다.

배열 비교의 경우 배열의 모든 원소가 핵심 필드라면 `Arrays.equals` 메서드를 사용한다.

간혹 null 값을 정상 값으로 취급하는 참조 타입 필드도 있는데, 이 경우에는 `Objects.equals(Object, Object)`로 비교해서 NullPointerException을 예방해야 한다.

어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 하기 때문에 **다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교한다.**

### 추가적인 주의사항

- equals를 재정의할 때는 hashCode도 반드시 재정의하기
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 않기 : `@Override` 어노테이션을 일관되게 사용하기 위함이다.

`**@AutoValue` 사용하기** 

[https://dahye-jeong.gitbook.io/java/java/advanced/2020-02-02-autovalue](https://dahye-jeong.gitbook.io/java/java/advanced/2020-02-02-autovalue)

AutoValue는 **불변 객체 클래스**를 편리하게 생성해주는 프레임워크로 위의 equals 메서드도 손쉽게 만들어낸다. 사용은 아래와 같이 한다. 

```java
@AutoValue
public abstract class Product {
    public abstract String name();
    public abstract BigDecimal price();

    @AutoValue.Builder
    public abstract static class Builder{
        public abstract Builder name(String name);
        public abstract Builder price(BigDecimal price);
        public abstract Product build();
    }

    public static Product.Builder builder(){
        return new AutoValue_Product.Builder();
    }
}
```

위 코드를 컴파일하면 자동으로 AutoValue_Product 클래스가 생성된다.

```java
import java.math.BigDecimal;

final class AutoValue_Product extends Product {
    private final String name;
    private final BigDecimal price;

    private AutoValue_Product(String name, BigDecimal price) {
        this.name = name;
        this.price = price;
    }

    public String name() {
        return this.name;
    }

    public BigDecimal price() {
        return this.price;
    }

    public String toString() {
        return "Product{name=" + this.name + ", price=" + this.price + "}";
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Product)) {
            return false;
        } else {
            Product that = (Product)o;
            return this.name.equals(that.name()) && this.price.equals(that.price());
        }
    }

    public int hashCode() {
        int h = 1;
        int h = h * 1000003;
        h ^= this.name.hashCode();
        h *= 1000003;
        h ^= this.price.hashCode();
        return h;
    }

    static final class Builder extends ch3.dahye.item11.Product.Builder {
        private String name;
        private BigDecimal price;

        Builder() {
        }

        public ch3.dahye.item11.Product.Builder name(String name) {
            if (name == null) {
                throw new NullPointerException("Null name");
            } else {
                this.name = name;
                return this;
            }
        }

        public ch3.dahye.item11.Product.Builder price(BigDecimal price) {
            if (price == null) {
                throw new NullPointerException("Null price");
            } else {
                this.price = price;
                return this;
            }
        }

        public Product build() {
            String missing = "";
            if (this.name == null) {
                missing = missing + " name";
            }

            if (this.price == null) {
                missing = missing + " price";
            }

            if (!missing.isEmpty()) {
                throw new IllegalStateException("Missing required properties:" + missing);
            } else {
                return new AutoValue_Product(this.name, this.price);
            }
        }
    }
}
```

코드에서 보다시피 AutoValue 는 필드와 클래스를 `final`로 제한하기 때문에 불변객체를 생성할 때만 사용해야 한다. `@AutoValue`가 붙는 클래스는 추상클래스로 작성해야 한다.
