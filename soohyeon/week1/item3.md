# 아이템 3 Private 생성자나 열거타입으로 싱글턴임으로 보증하라

> 오직 한 인스턴스만 만드는 클래스를 싱글톤이라 부른다. 보통 함수 같은 stateless 객체 또는 본질적으로 유일한 시스템 컴포넌트를 싱글톤으로 만든다. 
싱글톤은 이를 사용하는 클라이언트를 테스트하기가 어렵다. 
싱글턴으로 만드는 두가지 방식이 있는데, 두 방법 모두 생성자를 private으로 만들고 public static 멤버를 사용해서 유일한 인스턴스를 제공한다.
> 

## Public Static Final 필드

첫 번째 방법은 final 필드로 제공한다. 

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

}
```

public이나 protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다. 리플랙션을 이용해서 private 생성자를 호출하는 방법을 제외하게 된다면 말이다. 이런 API의 사용이 static 팩토리 메소드를 사용하는 방법에 비해 더 명확하고 간단하다. 

## Static Factory Method

두번째 방법인 정적 팩토리 메서드를 활용하는 것이다. 

```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

}
```

Elvis.getInstance()는 항상 같은 객체의 참조를 반환하므로 유일성이 보장된다. 

1. API를 변경하지 않고도 싱글톤으로 사용할지 말지 변경할 수 있다. 
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (왜 좋은지...?)
3. 정적 팩터리 메서드 참조를 공급자로 사용할 수 있다.

## 직렬화

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화 해보자. 두 방법 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다. 그 문제를 해결하려면 모든 인스턴스 필드에 일시적 (transient)를 추가하고 (즉, 직렬화 하지 않겠다는 뜻) 메소드를 다음과 같이 구현하면 된다. 

<aside>
💡 **직렬화?**
데이터 문맥에서 데이터 구조나 오브젝트 상태를 동일하거나 다른 컴퓨터 환경에 저장하고 나중에 재구성할 수 있는 포맷으로 변환하는 과정이다. 
이걸 왜 하는거야..? 어떨 때 필요한거지???
객체를 데이터로: 직렬화
데이터를 객체로: 역직렬화 → 중복되는 객체가 있을 수도 있다.

</aside>
