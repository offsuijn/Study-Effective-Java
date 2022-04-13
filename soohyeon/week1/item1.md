# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스는 생성자와 별도로 정적 팩터리 메서드 (static factory method)를 제공한다. 

## 장점

### 1 이름을 가질 수 있다.

> 생성자에 제공하는 파라미터가 거기에서 반환하는 객체를 잘 설명하지 못하는 경우에 이름을 가질 수 있는 static 팩토리를 사용하는 것이 알아보기 보다 더 쉽고 읽기 편하다.
> 

예를 들어,

```java
public Foo(String name) {
		this.name = name;
}

public static void main(String[] args){
		Foo foo = new Foo("keesun");
}
```

이렇게 public 생성자를 사용해서 객체를 생성하는 방식을 사용하면,  foo라는 변수가 어떤 것을 의미하는지 정확하게 알기 어렵다. 

```java
public static Foo withName(String name) {
		return new Foo(name);
}

public static void main(String[] args){
		Foo foo2 = Foo.withName("keesun");
}
```

따라서 위 코드와 같이 이름을 가질 수 있는 정적 팩토리 메서드를 사용해주면 “keesun”이라는 string이 어떤 것을 의미하는지 이름으로 바로 알 수 있다.

또한 하나의 시그니처로는 하나의 생성자만 만들 수 있다. 입력 매개변수를 달리해서 생성자를 추가할 수 있지만, 옳지 않은 방법이다. 

```java
String name;
String address;

public Foo(String name) {
		this.name = name;
}
public Foo(String address) {
		this.addresss= address;
}
```

따라서 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸어 특징을 살린 이름을 지어준다.

```java
public static Foo withName(String name) {
		return new Foo(name);
}
public static Foo withAddress(String address) {
		return new Foo(address);
}
```

### 2 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다

> 불변(immutable) 클래스(아이템 17)인 경우나 매번 새로운 객체를 만들 필요가 없는 경우에는 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
> 
> 
> 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할 지를 철저히 통제할 수 있다. 
> 

<aside>
💡 **왜 인스턴스를 통제할까?**
1. 인스턴스를 통제하면 클래스를 싱글턴(아이템3)으로도, 인스턴스화 불가(아이템4)로도 만들 수 있다. 
2. 불변 값 클래스(아이템17)에서 동치인 인스턴스가 단 하나뿐임으로 보장할 수 있다.
3. 플라이웨이트 패턴의 근간이 된다.
4. 열거타입은 인스턴스가 하나만 만들어짐을 보장한다.

</aside>

뭔소리일까? 이해가 안간다.

```java
public Foo(String name) {
		this.name = name;
}

public static void main(String[] args){
		Foo foo = new Foo("keesun");
}
```

위에서 봤던 예시를 다시 한번 보자. 현재 foo라는 생성자로 호출을 하게 되면 매번 새로운 객체를 리턴 받게 된다. 즉, 호출할 때마다 새로운 인스턴스가 만들어지는 것이다. 

하지만 아래와 같이 정적팩토리 메서드로 선언해주게 되면

```java
private static final Foo GOOD_NIGHT = new Foo();

//정적 팩토리 메서드로 선언
public static Foo getFoo(){
		return GOOD_NIGHT;
}

//foo 호출
public static void main(String[] args){
		Foo foo = Foo.getFoo();
}

```

몇 번을 호출해도 정적 팩토리 메서드인 GOOD_NIGHT이 리턴된다. 

### 3 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다

> 이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성’을 선물한다.
리턴 타입의 하위 타입의 인스턴스를 만들어줘도 되니까, 리턴 타입은 인터페이스로 지정하고 그 인터페이스의 구현체는 API로 노출시키지 않지만 그 구현체의 인스턴스를 만들어 줄 수 있다는 말이다.
> 
1. API를 작게 유지할 수 있다.
> 만들 때 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 
2. API를 사용할 때 알아야 할 개념의 개수와 난도 즉, 개념적인 무게 또한 줄일 수 있다. 

뭔가 감이 오는데 잘 모르겠다

코드를 통해서 한번 보자

```java
public static Foo withName(String name) {
		return new Foo(name);
}
```

이러한 인터페이스가 있을 때 밖, 즉 인터페이스를 사용하는 개발자의 입장에서는 Foo라는 인터페이스의 타입만을 알 수 있다. 실제 안에 있는 구현체가 어떤 것인지는 모른다. 

이러한 점을 이용하여 java.util.collections는 45개에 달하는 구현체를 갖고 있지만 그 모든 구현체는 non-public이다. 따라서 개발자가 알아야 할 개념의 수가 줄어들게 된다는 것이다. 

하지만 자바 9부터는 java.util.collections 대신 인터페이스에 private static method를 추가할 수 있게 되었다.

### 4 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

3과 같은 이유로 객체의 타입은 다를 수 있다. Foo로 감싸고 있다고 해서 그 안의 구현체가 반드시 같은 형태의 타입으로 리턴하지 않아도 된다는 뜻이다. 

```java
public static Foo withName(String name) {
		return new Foo(name); //얘가 꼭 Foo 타입을 리턴해도 되지 않아도 된다는 뜻
}
```

만약 foo의 하위타입이 있다면 그 타입을 리턴해도 된다. 

그럼 다른 타입을 리턴하는 예시를 한번 보자.

```java
//foo의 하위타입
static class BarFoo extends Foo{
}

public static Foo getFoo(boolean flag){
	return flag ? new Foo() : new BarFoo(); //BarFoo라는 새로운 타입
}
```

여기에서는 Foo 타입이 아닌 BarFoo 타입을 리턴할 수도 있게 되는 것이다. 

이럴 때 3과 마찬가지로 클라이언트는 이 두 클래스의 존재를 모른다. 알 필요도 없는 것이다. 

자바에서는 EnumSet 클래스는 생성자 없이 allOf(), of()를 제공하는데, 그 안에서 리턴하는 객체의 타입은 enum 타입의 개수에 따라 다른 함수, RegularEnumSet(), JumboEnumSet()을 제공한다. ~~몰라도 된다.~~

+ 밖으로 노출될 필요가 없는 함수들은 private static을 사용하여 감춘다

### *5 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. TODO : issue에 모르는 것 올리기

> 이 또한 유연함을 제공한다. 이러한 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다. 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.
> 

어떻게..? 

```java
//foo의 하위타입 정의 x
//static class BarFoo extends Foo{
//}

public static Foo getFoo(boolean flag){
	Foo foo = new Foo();
	//TODO 어떤 특정 약속이 되어있는 텍스트 파일에서 FOo의 구현체의 FQCN을 읽어온다. 
	//* FQCN : 클래스가 속한 패키지명을 모두 포함한 이름을 말한다. Fully Qualified Class Name
	//TODO FQCN 에 해당하는 인스턴스를 생성한다. 
	//TODO foo 변수를 해당 인스턴스를 가리키도록 수정한다. 
	return foo;
}
```

이렇게 코드를 작성할 때, Foo가 호출되는 당시 해당 텍스트 파일에 뭐가 적혀 있느냐에 따라 다른 객체를 리턴하게 된다. 

*즉, 해당 메서드를 작성한 후에도 foo를 상속받은 다른 메서드를 나중에도 만들 수 있다는 뜻이다. 

*이게 왜 용이할까?

해당 내용은 좀 더 공부해보고 다시 와서 왜 필요한지, 언제 쓰이는지 확인해야 할듯

서비스 로드에 대해서 공부를 좀 더 해보면 이해가 잘 된다고 한다. 의존성 주입과 비슷하다고 한다는데...

---

## 단점

### 1 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

collections 프레임워크에서 제공하는 편의성 구현체 (java.util.Collections)는 상속할 수 없다. 뭔소리일까. 어차피 상속을 할 필요가 없는 클래스이므로 단점도 아니다. 

### 2 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자는 javadoc 상단에 모아서 보여주지만 static 팩토리 메소드는 API 문서에서 특별히 다뤄주지 않는다. 따라서 클래스나 인터페이스 문서 상단에 중요한 팩토리 메소드에 대한 문서를 제공하는 것이 좋다. 

<aside>
💡 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.

</aside>
