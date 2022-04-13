> 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라
> 
> 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
> 
> 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
> 
> 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
> 
> 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
> 
> 아이템 6. 불필요한 객체 생성을 피하라

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라
정적 팩터리 메서드는 해당 클래스의 인스턴스를 반환하는 단순한 정적 메서드이다.

#### 장점
- 이름을 가질 수 있다.
   - 생성자처럼 클래스의 이름과 동일하지 않아도 된다.
   - 시그니처가 같은 생성자가 여러 개 필요한 상황에 유리하다.
   - 시그니처란 메서드의 이름과 매개변수의 리스트이다.

- 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

- 반환 타입의 하위 객체를 반환할 수 있는 능력이 있다.

- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    - EnumSet 클래스의 경우 원소가 64개 이하면 long 변수 하나로 관리하는 RegularEnumSet을 반환하고 65개 이상이면 long 배열로 관리하는 JumboEnumSet을 반환한다.

- 정적 팩토리 메소드 작성 시점에는 반환 객체의 클래스가 존재하지 않아도 된다.

### 단점
- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.

- 정적 팩토리 메서드는 프로그래머가 찾기 어렵다. 
   - API 문서를 잘 써두고, 메소드 이름도 널리 알려진 규약을 따라 지어야 한다.
   - from, of, valueOf, getInstance ...

👉 **무작정 public 생성자를 사용하지 말고, 정적 팩터리를 고려해보자**

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
### 점층적 생성자 패턴

필수 매개변수만 받는 생성자, 필수 매개변수와 선택매개변수 1개를 받는 생성자... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.

```java
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    
    ...
```

단점 
 - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

### 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정한다.

단점
- 객체 하나를 만들기 위해서 메서드를 여러개 호출해야하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진다.
- 클래스를 불변으로 만들 수 없다.

### 빌더 패턴

필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻은 뒤, 빌더 객체가 제공하는 일종의 세터 메서드로 원하는 매개변수를 설정한다. 
매개변수가 없는 build 메서드를 호출해 객체를 얻는다.
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
```
빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

단점
- 성능에 민감한 상황에서는 빌더 생성 비용이 문제가 될 수 있다.

## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
싱글톤(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다.

**싱글턴을 만드는 세 가지 방법**
### 1. public static final 필드 방식
private 생성자는 Sujin.INSTANCE를 초기화할 때 딱 한번만 호출된다.
```java
public class Sujin {
    public static final Sujin INSTANCE = new Sujin();

    private Sujin() { }
}
```
### 2. 정적 팩터리 방식
```java
public class Sujin {
    private static final Sujin INSTANCE = new Sujin();
    private Sujin() { }
    public static Sujin getInstance() { return INSTANCE; }
}
```
장점
- 해당 클래스가 싱글턴임이 API에 명백히 드러난다.
- 정적 팩터리를 제너릭 싱글턴 팩터리로 만들 수 있다.

### 3. 열거 타입 방식
public 방식과 비슷하지만 간결하고 직렬화 문제도 막아준다.
```java
public enum Sujin {
    INSTANCE;
}
```
👉 **대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다!**

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

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

### 하나 이상의 자원에 의존하는 클래스 구현 방법

#### 1. 정적 유틸리티 클래스
```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {}
    
    public static boolean isValid(String word) {}
    public static List<String> suggestions(String type) {}
}
```


#### 2. 싱글턴
```java
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public static boolean isValid(String word) {};
    public static List<String> suggestions(String type) {};
}
```

1, 2는 유연하지 않고 테스트하기 어렵다.

#### 3. 의존 객체 주입
인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨준다.
```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
}
```
불변을 보장하고, 정적 팩터리나 빌더 모두에 똑같이 응용할 수 있다.

변형으로 생성자에 자원 팩터리를 넘겨줄 수 있다.
- 팩터리는 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체이다.
팩터리로 `Supplier<T>` 를 사용할 수 있다.

```java
Mosaic create(Supplier<? extends Tile> tileFactory) {
	this.tile = tileFactory.get();
}
```

👉 **클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면
필요한 자원을 생성자 혹은 정적 팩터리나 빌더에 넘겨주는 것이 좋다**

## 아이템 6. 불필요한 객체 생성을 피하라
```java
String s1 = new String("java...");
String s2 = "java..."; //인스턴스 재사용
```

### 정적 팩터리 메서드를 사용하자
생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.
따라서 Boolean(String) 생성자 대신 Boolean.valueOf(String) 메서드를 사용하는 것이 좋다.

### 생성 비용이 비싼 객체는 재사용하자
```java
//BEFORE
static boolean isKoreanWord(String s) {
    return s.matches("[가-힣]");
}

//AFTER
private static final Pattern KOREAN = Pattern.compile("[가-힣]");
static boolean isKoreanWord(String s) {
    return KOREAN.matcher(s).matches();
}
```
내부에서 생성되는 정규표현식용 Pattern 인스턴스는 한 번 사용하고 버려진다.
-> Pattern 인스턴스를 캐싱해두고 isKoreanWord가 호출될 때마다 이 인스턴스를 재사용한다.

### 오토박싱이 숨어들지 않도록 주의하자
오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
}
```
long 타입인 i가 Long 타입인 sum에 더해질 때마다 Long 인스턴스가 만들어진다.
박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.
