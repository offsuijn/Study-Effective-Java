# [item 20] 추상클래스 보다는 인터페이스를 우선하라

## 추상 클래스 vs 인터페이스

**공통점**

선언 내용은 존재하지만 구현 내용은 없다 (추상 메서드를 갖는다)

인스턴스로 생성할 수 없다.

**목적**

인터페이스 : 함수의 껍데기만 존재해서 구현을 강제. 구현 객체가 같은 동작을 하도록 보장 Has A

추상클래스: 추상 클래스를 상속받아 기능을 이용하고 추가 Is - A

**다중상속**

인터페이스: 가능 / 추상클래스 : 불가능

**자바는 단일 상속만 지원하므로 기존의 다른 클래스를 상속 받고 있는 상황이라면 추상 클래스를 상속 받는 새로운 타입을 정의하기 까다롭다.**

## 인터페이스는 기존 클래스에 구현하기 쉽다 :MixIn 정의

기존에 A라는 기능을 하는 클래스가 있으면 인터페이스의 기능을 추가하고 싶을 때 손쉽게 `implement`s 만해서 추가할 수 있다.

하지만 추상클래스는 기존의 A 기능과 추상 클래스 B 기능을 덧붙이기가 쉽지 않다. 기존의 A 기능을 유지하기 힘들다.

```java
public class A implements Comparable{

  @Override    //믹스인
  public int compareTo(Object o) {
    return 0;
  }

  public void a(){} //주된 타입

}
```

```java
//추상 클래스 C를 상속받아 믹스인을 구현할 수 없다
public class A extends B{

  public void a(){} //주된 타입
  
}

abstract class C{
  abstract int compareTo(Object o); //믹스인
}
```

## 인터페이스는 계층구조가 없는 타입 프레임워크를 만들 수 있다

```java
public abstract class Singer {
   abstract AudioClip sing(Song song);
}

public abstract class Songwriter {
    abstract Song compose(int chartPosition);
}

// 두 개를 extend할 수 없다
public abstract class SingerSongwriter {
    // Singer
    abstract AudioClip sing(Song song);
    // Songwriter
    abstract Song compose(int chartPosition);
    
    abstract AudioClip strum();
    abstract void actSensitive();
}

//이런 구조는 논리적으로 맞지 않다.
public abstract class Singer extends Songwriter{
   abstract AudioClip sing(Song song);
}

public abstract class SingerSongwriter extends Singer{
    // Singer
    abstract AudioClip sing(Song song);
    // Songwriter
    abstract Song compose(int chartPosition);
    
    abstract AudioClip strum();
    abstract void actSensitive();
}
```

인터페이스로 작성하면 이런 고민이 없다

```java
public interface Singer  {
    AudioClip sing(Song s);
}

public interface Songwriter  {
    Song compose(int chartPosition);
}

//둘을 상속 받는 새로운 인터페이스도 작성 가능함
public interface SingerSongwriter extends Singer, Songwriter  {
    AudioClip strum();
    void actSensitive();
}
```

## 인터페이스의 default 메서드

인터페이스 메서드 중 구현 방법이 명확한 것이 있다면, default 메서드로 제공 가능함

- 단, default 메서드를 제공할 때 상속하는 사람들을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화해야함.
- equals나 hashCode 같은 Object 메서드를 디폴트 메서드로 제공해서는 안됨.
- 인터페이스는 인스턴스 필드를 가질 수 없고, public이 아닌 정적 맴버도 가질 수 없음(자바 9부터는 private static 메서드도 구현가능)

## 템플릿 메서드 패턴

### 템플릿 메서드를 쓰는 경우 ⇒ 인터페이스를 우선적으로 구현하되 인터페이스의 메서드 중에서 디폴트 메서드나 기반 메서드가 안되는 메서드가 존재하는 경우 (ex. Object 메서드)

인터페이스와 추상 골격 구현(skeletal implementation)클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취할 수 있음. 관례상 인터페이스이름이 Interface라면 그 골격 구현 클래스는 AbstractInterface로 짓는다.

**즉, 추상 클래스 + 인터페이스 조합**

인터페이스로는 타입을 쉽게 정의하고, 필요하다면 디폴트 메서드 몇 개도 함께 제공함. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다. 

→ 인터페이스를 구현하는데 필요한 대부분의 일이 완료된다.

List인터페이스를 구현하면서 AbstractList는 추상 클래스로 작성되어 있다.

![Untitled](%5Bitem%2020%5D%20%E1%84%8E%E1%85%AE%E1%84%89%E1%85%A1%E1%86%BC%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%20%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%AE%E1%84%89%E1%85%A5%20da857b1c77464fc0bea978b5a42dcf96/Untitled.png)

AbstractList에서는 List 인터페이스를 대부분 구현하고 구현하지 않은 메서드는 추상 메서드로 남겨놨다. 

![Untitled](%5Bitem%2020%5D%20%E1%84%8E%E1%85%AE%E1%84%89%E1%85%A1%E1%86%BC%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%20%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%AE%E1%84%89%E1%85%A5%20da857b1c77464fc0bea978b5a42dcf96/Untitled%201.png)

![Untitled](%5Bitem%2020%5D%20%E1%84%8E%E1%85%AE%E1%84%89%E1%85%A1%E1%86%BC%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A2%E1%84%89%E1%85%B3%20%E1%84%87%E1%85%A9%E1%84%83%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%90%E1%85%A5%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%E1%84%85%E1%85%B3%E1%86%AF%20%E1%84%8B%E1%85%AE%E1%84%89%E1%85%A5%20da857b1c77464fc0bea978b5a42dcf96/Untitled%202.png)

## 골격 구현 클래스 작성

1. 인터페이스를 확인해 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다. 

```java
public interface AInterface {

  public boolean equals();

  public int getSize();

  public boolean isEmpty();
  
}
```

1. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공한다. 단 equals와 hashCode 같은 Object의 메서드는 디폴트 메서드로 제공하면 안된다.

```java
public interface AInterface {

  public boolean equals();

  public int getSize(); //기반 메서드

  public default boolean isEmpty(){  //기반 메서드로 구현한 default 메서드
    return getSize() > 0;
  }
}
```

1. 만약 인터페이스의 메서드가 모두 기반메서드와 디폴트 메서드가 된다면 골격 구현 클래스를 만들 필요가 없다.
2. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아있다면 이 인터페이스를 구현하는 골격 클래스를 만들고 남은 메서드들을 작성해서 넣는다.

```java
public abstract class AbstractAInterface implements AInterface{

  @Override
  public boolean equals(Object obj) {
    
    ...
    
    return ..;
  }
}
```

## 결론

일반적으로 다중 구현용 타입에는 인터페이스가 가장 적합하다. 

복잡한 인터페이스라면 골격 구현을 함께 제공하는 방법을 고려하자. 

골격 구현은 '가능한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. 

인터페이스에 걸려있는 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하다.
