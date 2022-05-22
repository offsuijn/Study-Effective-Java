# 아이템 34. int 상수 대신 열거 타입을 사용하라

자바에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언해서 사용하는 정수 열거 패턴을 사용했다.
하지만 정수 열거 패턴에는 많은 단점이 존재한다.

## 정수 열거 패턴의 단점
#### 예제: 정수 열거 패턴
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
1. 타입 안정성을 보장할 수 없다.
    - 컴파일러 입장에서는 APPLE_FUJI나 ORANGE_NAVEL은 모두 같은 0을 나타내기 때문에 동등 연산자(==)로 비교하더라도 아무런 경고를 발생시키지 않는다. 
    
    - 즉, APPLE_FUJI가 전달되어야 할 값에 ORANGE_NAVEL가 전달되어도 아무런 문제 없이 컴파일된다는 뜻이다.
    
    - 따라서 타입 안정성을 보장할 수 없다.

2. 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.
    - 정수 열거 패턴은 상수를 나열한 것뿐이라 컴파일 후 상수의 값이 바뀐다면 다시 컴파일해줘야 한다.

3. 문자열로 출력하기 까다롭다.
    - 값을 출력하거나 디버깅할 때 단지 숫자로 값이 표현되기 때문에 그다지 도움이 되지 않는다.
    
## 열거 타입(Enum Type)

#### 예제: 가장 단순한 열거 타입
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
Java의 열거 타입은 완전한 형태의 클래스이다.
상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

### 열거 타입의 장점
1. 인스턴스가 하나씩만 존재함을 보장한다.

    - 생성자를 제공하지 않으므로 사실상 final이다.
  
   - 따라서 인스턴스를 생성하거나 확장할 수 없으니, 인스턴스가 하나씩만 존재함을 보장할 수 있다.

2. 타입 안정성을 제공한다.
   - Apple 타입을 매개변수로 받는 메서드는 Apple 타입의 값만 넘겨받을 수 있다. 다른 타입을 넘기려 하면 컴파일 오류가 발생한다.
   
3. namespace를 제공한다.
   - 열거 타입에는 각자의 이름 공간(namespace)이 있어서 이름이 같은 상수도 공존할 수 있다.
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { FUJI, PIPPIN, GRANNY_SMITH }
```

4. 임의의 메서드나 필드를 추가할 수 있고, 인터페이스를 구현할 수 있다.
```java
public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS(4.869e+24,6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23,3.393e6),
    JUPITER(1.899e+27,7.149e7),
    SATURN(5.685e+26,6.027e7),
    URAUS(8.683e+25,2.556e7),
    NEPTUNE(1.024e+26,2.477e7);

    // 임의의 필드
    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면중력

    //중력상수
    private static final double G = 6.67300E-11;

	//생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }
    
    // 임의의 메서드
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```
5. 열거 타입의 상수를 제거해도 참조하지 않는 클라이언트는 아무 영향이 없다.

### 상수별 메서드 구현
상수마다 동작이 달라져야 하는 상황이라면?
#### 예제: 상수별 메서드 구현
```java
public enum Operation {
    PLUS{
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    public abstract double apply(double x, double y);
}

```
apply라는 추상 메서드로 인해 새로운 상수가 추가되면 재정의를 강제한다.
재정의되지 않으면 컴파일 오류로 알려준다. 

#### 예제: 상수별 데이터와 결합하는 방법
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```
위의 코드에서 toString을 상수의 이름이 아닌 연산기호를 반환하도록 재정의한 것을 볼 수 있다. 

### 전략 열거 타입 패턴
상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.
이 단점을 보완하려면 전략 열거 타입 패턴을 사용하자.

전략 열거 타입 패턴은 상수를 추가할 때 전략을 선택하도록 하는 패턴이다.
private 중첩 열거 타입을 만들고 계산을 위임한다.
그리고 바깥 열거 타입 생성자에서 전략을 인자로 받게 하면 된다.

#### 예제: 전략 열거 타입 패턴
```java
// 잔업 수당을 계산해준다.
// 주중에 오버타임, 주말은 무조건 잔업 수당이 발생한다!
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    // 전략 열거 타입, private 중첩 열거 타입
    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

### 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
