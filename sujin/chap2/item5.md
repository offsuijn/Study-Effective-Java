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
