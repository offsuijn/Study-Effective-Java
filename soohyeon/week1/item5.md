# 아이템 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> 많은 클래스는 여러 자원에 의존한다. 이 책에서는 SpellChecker와 Dictionary를 예로 들고 있다. 즉 SpellCheck가 Dictionary 를 사용하고, 이를 의존하는 리소스 또는 의존성이라고 부른다. 이때 SpellChecker를 다음과 같이 구현하는 경우가 있다.
> 

## 부적절한 구현

### Static 유틸 클래스로 구현

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

### 싱글턴으로 구현

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

두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 훌륭하지 않다.  어떤 클래스가 사용하는 리소스에 따라 행동을 달리 해야 하는 경우에는 스태틱 유틸리티 클래스와 싱글톤을 사용하는 것은 부적절하다. 

그런 요구 사항을 만족할 수 있는 간단한 패턴으로 생성자를 사용해서 새 인스턴스를 생성할 때 사용할 리소스를 넘겨주는 방법이 있다.

## 적절한 구현

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }
    
    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }

}

class Lexicon {}
```

위와 같은 의존성 주입은 생성자, 스태틱 팩토리 그리고 빌더에도 적용할 수 있다. 

이 패턴의 변종으로 리소스의 팩토리를 생성자에 전달하는 방법도 있다. 

의존성 주입이 유연함과 테스트 용이함을 크게 향상 시켜주지만, 의존성이 많은 큰 프로젝트인 경웅에는 코드가 장황해 질 수 있다. 그점은 대거, 쥬스, 스프링 같은 프레임웍을 사용해서 해결할 수 있다.

흐음... 어떻게...?

<aside>
💡 **핵심정리**
클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 말자. 그런 경우에는 리소스를 생성자나 팩토리로 전달하는 의존성 주입을 사용하여 유연함, 재사용성, 테스트 용이성을 향상 시키자.

</aside>
