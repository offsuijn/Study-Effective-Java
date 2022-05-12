# [아이템 13] clone 재정의는 주의해서 진행하라 - 자바에서 객체 복사는 clone()이 아닌 복사 생성자, 팩토리를 사용하자

## 자바에서 객체를 복사할 때 쓰는 방법 2가지

### 1. Object 객체의 clone()메서드 + Cloneable 인터페이스 사용

복사가 필요한 클래스에서 `clone`을 오버라이딩 하여 사용한다.

이 때 Cloneable 인터페이스를 받아서 구현해야 한다. (안그러면 `clone`메서드에서 오류남)

하지만 Cloneable과 clone 메서드를 쓰는 것에는 문제가 있다.

예외처리가 번거롭고 참조형 필드 객체(가변객체)는 단순 clone()이 아니라 딥카피로 해야 한다는 것이다.

따라서 2번의 복사 생성자 또는 복사 팩토리 사용을 권장한다. 

### 2. 복사 생성자 또는 복사 팩토리 사용

**WHY**

형변환이 필요없다. 인터페이스 타입의 인스턴스를 인수로 받을 수도 있다.

예를 들어 Collection이나 Map 타입에는 인터페이스 기반 복사 생성자, 팩토리를 제공한다. 이를 사용하면 원본 구현 타입에 얽매이지 않고 복사 타입을 정할 수 있다. 

**HashSet → TreeSet으로 복사**

```java
HashSet<Integer> set = new HashSet();
set.add(1);
set.add(2);

// 복사할 때 new TreeSet<>(set) 을 통해 HashSet인 set을 TreeSet으로 타입 변경 가능
TreeSet<Integer> copySet = new TreeSet<>(set);
```

# 복사 생성자

복사 생성자는 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자이다.

```java
// 예 1
public Yum(Yum yum) { ... };

// 예 2
public class BoundedSet<T> {
    private final LinkedList<T> data;
    private final int capacity;

    // ...

    public BoundedSet(BoundedSet<T> other) {
        data = new LinkedList<>(other.data);
        capacity = other.capacity;
    }

    // ...
}

// 예 3
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(Stack s) {
        this.elements = s.elements.clone();
        this.size = s.size;
    }
}
```

# 복사 팩터리

복사 팩토리는 복사 생성자를 모방한 정적 팩터리 메서드이다. 

(참고: 객체 생성할 때 생성자 대신 정적 팩터리 쓰기) 

```java
// 예 1
public static Yum newInstance(Yum yum) { ... };

// 예 2
public class BoundedSet<T> {
    private final LinkedList<T> data;
    private final int capacity;

    // ...

    public static <T> BoundedSet<T> newInstance(BoundedSet<T> other) {
        return new BoundedSet<T>(other.data, other.capacity);

    }
    // ...
}

// 예 3
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public static Stack newInstance(Stack s) {
        return new Stack(s.elements, s.size);
    }
}
```
