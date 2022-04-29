## [아이템 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

하나 이상의 자원에 의존하는 클래스는 정적 유틸리티(모든 멤버가 정적인 클래스)나
싱글톤으로 작성해서는 안된다.

- 정적 유틸리티

```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private static final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
        // Noninstantiable
    }

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

interface Lexicon {}

class KoreanDicationry implements Lexicon {}
```

- 싱글톤

```java
// 부적절한 싱글톤 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {

    private final Lexicon dictionary = new KoreanDicationry();

    private SpellChecker() {
    }

    public static final SpellChecker INSTANCE = new SpellChecker() {
    };

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}
```

위 두 예시의 문제는 
의존하는 자원( dictionary) 의 종류를 바꾸거나 확장할 때 문제가 생긴다.

**사용하는 자원에 따라 동작이 달라지는 클래스에는 위와 같은 방법이 적합하지 않다.**

대신, **인스턴스를 생성할 때 생성자(혹은 정적 펙터리 메서드나 빌더)에 필요한 자원을 넘기는 방식을 사용**해야 한다. (의존성 주입)

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestion(String typo) {
        throw new UnsupportedOperationException();
    }
}
```

**Supplier<T> 를 활용한 의존성 주입**

생성자에 자원 팩토리를 넘겨주는 방식으로 이 패턴을 확장할 수 있다.

팩토리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말하는데, 주로 공급자(Supplier<T>)를 가리킨다. 

**이 때 Supplier<T>를 입력받는 메서드는 일반적으로 `한정적 와일드카드 타입`을 사용해서 팩터리의 타입 매개변수를 한정해야 한다.**

참고_ **와일드카드**

- `<? extends T>` 와일드 카드의 상한 제한(upper bound) : T와 그 자손들을 구현한 객체만 매개변수로 가능
- `<? super T>` 와일드 카드의 하한 제한(lower bound) : T와 그의 조상들을 구현한 객체만 매개변수로 가능
- `<?>` 제한 없음

```java
Mosaic create(Supplier<? extends Tile> tileFactory{...}
```

위의 코드 예시는 클라이언트가 제공한 펙터리가 생성한 Tile들로 구성된 모자이크를 만드는 메서드이다. 이 때 Tile의 종류는 Tile의 하위 클래스로도 가능하다.

**이러한 의존 객체 주입은 클래스의 유연성, 재사용성, 테스트 용이성을 개선한다.**
