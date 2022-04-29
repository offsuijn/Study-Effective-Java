## [아이템 7] 다 쓴 객체 참조를 해제하라

객체 참조를 제때에 미리 해제하지 않으면 **메모리 누수**가 나기 쉽다. 

이러한 문제는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있으니 미리 예방법을 익혀두는 것이 좋다.

1. **자기 메모리를 직접 관리하는 클래스에서 쓰지 않는 객체는 `null`처리하기** 

```java
public class Stack{
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack(){
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	public void push(Object e){
		ensureCapacity();
		elements[size++] = e;
	}
	public Object pop(){
		if(size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}

	private void ensureCapacity(){
		if(elements.length == size)
			elements = Arrays.copyOf(elements,2*size+1);
	}
}

```

위 코드의 Stack 클래스는 ensureCapacity 메서드를 통해 elements 메모리의 크기를 자신이 직접 관리한다. 

이 코드에서 메모리 누수가 일어날 수 있는 지점은 pop 메서드인데, elements 배열의 index를 size 변수로 포인터 위치만 앞으로 옮겨갈 뿐, 실질적으로 pop되어서 더 이상 쓰이지 않는 Object 객체는 여전히 elements 배열에 살아 남아있다.

elements 배열을 **활성 영역**이라고 하는데, 더 이상 쓰지 않는 객체가 활성 영역에 남아있는 경우 가비지 컬렉터가 해당 객체 뿐만 아니라 해당 객체가 참조하는 객체들 모두를 회수해가지 못하는 문제가 생긴다. 이는 잠재적으로 성능 저하의 문제를 초래한다.

따라서 위와 같은 경우 pop을 해서 더 이상 참조할 일이 없는 객체는 `null`처리를 해주어야 한다. 아래 코드와 같이 말이다.

```java
public Object pop(){
	if(size == 0) throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```

이 코드는 메모리 누수 문제 말고도 더 이상 쓰이지 않는 객체를 참조하려고 하면 `NullpointException`이 나서 오류 수정에 더욱 용이하다는 장점이 있다. 

1. **WeakHashMap 사용하기** 

일반적인 HashMap의 경우 일단 Map 안에 Key 와 Value 가 put되면 사용여부와 관계 없이 해당 내용은 삭제되지 않는다. WeakHashMap은 Key 객체가 null이 되면 강제 Garbage Collection 이후에 해당 key와 value의 값을 map에서 제거한다. 

```java
public class WeakHashMapTest {
    public static void main(String[] args) {
        WeakHashMap<Integer, String> map = new WeakHashMap<>();
        Integer key1 = 1000;
        Integer key2 = 2000;
        map.put(key1, "test a");
        map.put(key2, "test b");
        key1 = null;
        System.gc();  //강제 Garbage Collection
        map.entrySet().stream().forEach(el -> System.out.println(el));
    }
}
```
