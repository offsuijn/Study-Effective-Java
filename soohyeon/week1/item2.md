# 아이템 2 생성자에 매개변수가 많다면 빌더를 고려하라

> static 팩토리 메소드와 public 생성자 모두 매개변수가 많이 필요한 경우에 불편해진다. 
선택 매개변수가 많을 때 활용할 수 있는 대안들을 확인해보자.
> 

## 해결책

### 1 생성자

설정하고 싶은 매개변수를 최소한으로 사용하는 생성자를 사용해서 인스턴스를 만든다. 

```java
NutritionFacts cocaCola =
new NutritionFacts(240, 8, 100, 0, 35, 27);
```

보통의 프로그래머는 점층적 생성자 패턴을 사용하지만, **매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다**. 또한 사용자가 설정하기 원치 않는 매개변수까지 포함하기 쉽다. 

### 2 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다. 

```java
//매개변수가 없는 생성자로 객체 생성
NutritionFacts cocaCola = new NutritionFacts(); 
//세터 메서드 호출로 원하는 매개변수 값 설정
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

이 방법의 단점은 최종적인 객체를 하나 만들려면 여러 번의 호출을 거쳐야 하기 때문에 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 여지가 있다. 

*이러한 일관성의 문제 때문에 자바빈즈 패턴에서는 클래스를 불변(아이템17)으로 만들 수 없다. 

- 어떻게 한다는건지 이해는 안가지만... 당연한거 아님??
    
    

이러한 단점을 보완하기 위해서 사용하기 전 freezing을 쓴다는데.... 다루는 방법이 어려워 실전에서는 거의 쓰이지 않는다. 세마포어랑 비슷한 느낌??

<aside>
💡 **불변**
어떠한 변경도 허용하지 않는다는 뜻으로 주로 변경을 허용하는 가변 객체와 구분하는 용도로 쓰인다. 대표적으로 String 객체는 한번 만들어지면 절대 값을 바꿀 수 없는 불변 객체다. 
**불변식**
프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다. 주어진 조건 내에서만 변경이 허용된다. 가변 객체에도 불변식은 존재할 수 있으며, 넓게 보면 불변은 불변식의 극단적인 예라 할 수 있다.

</aside>

설명을 보니 좀 더 이해가 되는 것 같기도 하고... 일단 넘어가자..

### 3 빌더

> 앞의 1 점층적 생성자 패턴의 안전성과 2 자바빈즈의 가독성을 겸비한 빌더패턴이다. 
1. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수적인 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 
2. 그 후 빌더 객체가 제공하는 일종의 세터 메서드로 원하는 선택 매개변수들을 설정한다. (부가적 필드)
3. 매개변수가 없는 build 메서드를 호출해 필요한 객체(보통은 불변)를 얻는다.
> 

![불변인 NutritionFacts 클래스](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bc1d3a12-f029-4814-8349-190a118286af/Untitled.png)

불변인 NutritionFacts 클래스

빌더의 세터 메서드는 빌더 자신을 반환한다. 때문에 연쇄적인 호출이 가능하다. 이러한 호출 방식을 따서 플루언트 API 혹은 메서드 연쇄라고 부른다. 

그럼 위와 같은 빌더를 이용한 클래스의 클라이언트 코드를 한번 보자.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

1,2를 합쳐놓은 것으로 보인다! 매개변수의 값을 설정하면서 생성자를 선언할 수 있다. 

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.** 

왜?

추상 빌더를 가지고 있는 추상 클래스를 만들고 하위 클래스에서는 추상 클래스를 상속받으며 각 하위 클래스용 빌더도 추상 빌더를 상속받아 만들 수 있다. 

예시를 통해 한번 알아보자. 

피자의 다양한 종류를 표현한 계층구조의 루트에 놓인 추상 클래스이다. 

```java
public abstract class Pizza {

    public enum Topping {
        HAM, MUSHROOM, ONION, PEPPER, SAUSAGE
    }

    final Set<Topping> toppings;

//자기자신의 하위타입을 받는 빌더
    abstract static class Builder<T extends  Builder<T>> { // `재귀적인 타입 매개변수`
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class); //비어있는 셋

			//addTopping제공
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

			//Pizza의 하위타입을 만들게 됨
        abstract Pizza build(); // `Convariant 리턴 타입`을 위한 준비작업

			// 추상메서드인 `self-type` 개념을 사용해서 형변환 없이도 메소드 체이닝이 가능케 함
        protected abstract T self(); 
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }

}
```

이 피자를 상속받는 ny피자를 만들어봅니다. 

```java
public class NyPizza extends Pizza {

    public enum Size {
        SMALL, MEDIUM, LARGE
    }

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this); //빌더 자체를 넘겨받음
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

칼조네 피자

```java
public class Calzone extends Pizza {

    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauseInside = false;

        public Builder sauceInde() {
            sauseInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder); 
	//상위 빌더를 호출해도 해당 클래스의 타입을 반환한다. 즉 pizza가 아닌 Calzone를 반환함
        sauceInside = builder.sauseInside;
    }

}
```

여기서 각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다. 즉, NyPiazza.Builder는 NyPizza를 반환하고, Calzone.Builder는 Calzone를 반환한다는 뜻이다. 다시 말해서, **하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환타입이 아닌 그 하위타입을 반환하는 기능을 공변 반환 타이핑**이라 한다. 

<aside>
💡 **결론**
생성자나 정적 팩터리가 처리해야할 매개변수가 많다면 빌더패턴을 선택하는 것이 더 유연하다.

</aside>
