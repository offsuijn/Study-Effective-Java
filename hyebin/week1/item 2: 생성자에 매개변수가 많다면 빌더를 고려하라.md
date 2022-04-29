## [아이템 2] 생성자에 매개변수가 많다면 빌더를 고려하라

매개변수가 많을 때 해결 방법 

1. 점층적 생성자 패턴 - **확장하기 어렵다!!!** 

이 경우 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 코드를 읽을 때 각 값의 의미가 무엇인지 헷갈리고 매개변수가 몇 개인지 주의해서 세어야 한다. 또한 매개변수가 늘어져 있으면 버그 찾기도 어렵고 매개변수 순서가 틀리는 오류도 자주 범한다. 

```java
public class NutritionFacts{
	private final int servingSize; //필수
	private final int servings; //필수
	private final int calories; //선택
	private final int fat; //선택
	private final int sodium; //선택
	private final int carbohydrate; //선택

public NutritionFacts(int servingSize, int servings){
this(servingSize, servings, 0);}
public NutritionFacts(int servingSize, int servings, int calories){
this(servingSize, servings, calories, 0);
}
...

```

1. 자바 빈스 패턴 - 일관성이 깨지고 불변으로 만들 수 없다.

매개변수가 없는 생성자를 만들고 setter로 매개변수 입력을 대신한다. 

이 경우 객체 하나를 만들려면 메서드(setter) 여러 개를 호출해야 하고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 일관성이 깨진 객체가 만들어지면 버그를 심은 코드와 그 버그 때문에 런타임에 문제를 겪는 코드가 물리적으로 떨어져 있을 것이므로 디버깅이 어렵다. 

또한 이렇게 일관성이 무너지는 문제 때문에 클래스를 불변으로 만들 수도 없다. 

1. 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취함

**클라이언트는 필요한 객체를 직접 만드는 대신 필수 매개변수만으로 생성자(혹은 정적 팩터리 메소드)를 호출해 빌더 객체를 얻는다.** 그런 다음 빌더 객체가 제공하는 일종의 setter 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 `build()`메서드를 호출해서 필요한 (보통 불변인) 객체를 리턴하도록 한다.

**보통 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어둔다.**

```java
public class NutritionFacts{
	private final int servingSize; //필수
	private final int servings; //필수
	private final int calories; //선택
	private final int fat; //선택
	private final int sodium; //선택
	private final int carbohydrate; //선택

	public static class Builder{
		//필수 매개변수
		private final int servingSize;
		private final int servings;
	
		//선택 매개변수 - 기본값으로 초기화
		private int calories = 0;
		private int fat = 0;
		..
		
		public Builder(int servingSize, int servings){
			this.servingSize = servingSize;
			this.servings = servings;
		}
		public Builder calories(int val){
			calories = val;
			return this;
		}
		.. //나머지 요소의 setter 도 같은 패턴으로 구현
		public NutritionFacts build(){
			return new NutritionFacts(this);
		}
	}
private NutritionFacts(Builder builder){
	servingSize = builder.servingSize;
	servings = builder.servings;
	....
}
}
```

이 코드의 사용은 아래와 같이 할 수 있다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).build();
```

<aside>
💡 **불변**은 어떠한 변경도 허용하지 않는다는 뜻
**불변식**은 프로그램이 실행되는 동안 혹은 정해진 기간 동안 반드시 만족해야 하는 조건.
변경을 허용할 수는 있으나 주어진 조건 내에서만 허용된다.

</aside>

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다**

각 계층의 클래스에 관련 빌더를 멤버로 정의한다. 

```java
public abstract class Pizza{
	public enum Topping {HAM, ONION, PEPPER}
	final Set<Topping> toppings;

	abstract static class Builder<T extends Builder<T>>{
		EnumSet<Topping> toppings = EnumSet.noneOf(Toppings.class);
		public T addTopping(Topping topping){
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}
	abstract Pizza build();

	//하위 클래스는 이 메서드를 재정의하여 this를 반환하도록 한다 
	protected abstract T self();
}

Pizza(Builder<?> builder){
	toppings = builder.toppings.clone(); 
}
}
```

```java
public class NyPizza extends Pizza{
	public enum Size {SMALL, MEDIUM, LARGE}
	private final Size size;

	public static class Builder extends Pizza.Builder<Builder>{
		private final Size size;
		
		public Builder(Size size){
			this.size = Object.requireNonNull(size);
		}
	
		@Override
		public NyPizza build(){
		return new NyPizza(this);
		}
		@Override
		protected Builder self(){ return this;}
}
private NyPizza(Builder builder){
	super(builder);
	size = builder.size;
}
}
```

각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.

빌더는 점층적 생성자 패턴보다는 코드가 장황하기 때문에 매개변수가 4개이상 되어야 가치가 있다. 하지만 개발을 하다보면 매개변수는 점점 늘어나기 때문에 애초에 빌더로 시작하는 편이 나을 때가 있다. 
