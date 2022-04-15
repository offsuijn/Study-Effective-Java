# 아이템 7 다 쓴 객체 참조를 해제하라

자바에서도 메모리 관리를 중요시 해야 한다. 

## 메모리 직접 관리

```java
// Can you spot the "memory leak"?
public class Stack {

    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size]; // 주목!!
    }

    /**
     * 원소를 위한 공간을 적어도 하ㅏ이상 확보한다.
		 * 
     */
    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 코드에서 무엇 문제일까?

이 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 왜? 이 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다. 

▶️ 해당 참조를 다 썼을 때 null 처리로 해결할 수 있다. 

위의 예시에서의 스택 클래스에서 각 원소의 참조가 더 이상 필요 없어지는 시점은 스택에서 꺼내질 때다. 

다음과 같이 코드를 수정할 수 있다. 

```java
public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object value = this.elements[--size];
        this.elements[size] = null;// 다 쓴 참조 해제
        return value;
    }
```

이렇게 수정하여 객체를 정리해줄 수 있다. 

하지만 매번 객체를 null처리해주는 것은 바람직하지 않다. 

**왜?**

**객체 참조를 null 처리하는 일은 예외적인 경우여야 하기 때문이다.** 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 (scope) 밖으로 밀어내는 것이다. 

그럼 null처리는 언제 해야 할까?

메모리를 직접 관리할 때! 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리해줘야 한다. 

## 캐시

캐시를 사용할 때도 메모리 누수 문제를 조심해야 한다. 객체의 레퍼런스를 캐시에 넣어 놓고 캐시를 비우는 것을 잊기 쉽다. **캐시의 키**에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 `WeakHashMap`을 쓸 수 있다.

weak 해시맵 참조: [https://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/](https://blog.breakingthat.com/2018/08/26/java-collection-map-weakhashmap/)

또는 특정 시간이 지나면 캐시 값이 의미가 없어지는 경우에 백그라운드 쓰레드를 사용하거나 (아마도 `ScheduledThreadPoolExecutor`), 새로운 엔트리를 추가할 때 부가적인 작업으로 기존 캐시를 비우는 일을 할 것이다. (`LinkedHashMap` 클래스는 `removeEldestEntry`라는 메서드를 제공한다.)

## 콜백

세 번째로는 리스너 혹은 콜백으로 메모리 누수가 발생한다. 클라이언트가 콜백을 등록할 수 있는 API를 만들고 콜백을 뺄 수 있는 방법을 제공하지 않는다면, 계속해서 콜백이 쌓일것이다. 이때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다. 

<aside>
💡 메모리 누수는 발견하기 쉽지 않기 때문에 수년간 시스템에 머물러 있을 수도 있다. 코드 인스택션이나 `heap profiler` 같은 디버깅 툴을 사용해서 찾아야 한다. 따라서 이런 문제를 예방하는 방법을 학습하여 미연에 방지하는 것이 좋다.

</aside>
