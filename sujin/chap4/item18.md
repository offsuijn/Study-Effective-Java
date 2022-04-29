## 아이템 18. 상속보다는 컴포지션을 사용하라
상속은 코드를 재사용하는 강력한 수단이지만 다음과 같은 문제가 있다.

### 상속은 캡슐화를 깨뜨린다.
- 상속은 하위 클래스가 상위 클래스에 대한 내부 구현 정보를 알게 한다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
#### 예시: 상속을 잘못 사용한 경우
```java
public class InstrumentedHashSet<E> extends HashSet<E> {

    //추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    //접근자 메소드
    public int getAddCount() {
        return addCount;
    }
}
```
- HashSet을 상속하고, 처음 생성된 이후 원소 몇 개가 더해졌는지 알 수 있는 기능을 추가했다.
- addAll()을 호출할 시 오동작한다.
   - HashSet의 addAll() 메서드가 add() 메서드를 사용하여 구현되기 때문이다.

### 컴포지션 사용
기존의 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

-> **컴포지션(Composition)**

새 클래스의 인스턴스 메서드들은 private 필드로 참조하는 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.

-> **전달(forwarding)**

#### 컴포지션의 장점
- 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않게 된다.
- 새로운 클래스는 기존의 클래스의 내부 구현에 대해 알지 못하게 되어 캡슐화 원칙도 잘 지켜지게 된다.

#### 예시: 컴포지션과 전달 방식 사용
```java
// 래퍼 클래스
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}

// 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    @Override
    public void clear() { s.clear(); }

    @Override
    public boolean contains(Object o) { return s.contains(o); }

    @Override
    public boolean isEmpty() { return s.isEmpty(); }

    @Override
    public int size() { return s.size(); }

    @Override
    public Iterator<E> iterator() { return s.iterator(); }

    @Override
    public boolean add(E e) { return s.add(e); }

    @Override
    public boolean remove(Object o) { return s.remove(o); }

    @Override
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }

    @Override
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }

    @Override
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }

    // 나머지 코드는 생략
}
```
- Set 인터페이스를 구현했고, Set의 인스턴스를 인수로 받는 생성자를 하나 제공한다.
- **임의의 Set**에 계측 기능을 덧씌워 새로운 Set으로 만들어준다.
   - 상속 방식은 구체 클래스 각각을 따로 확장해야 한다.
   - 하지만 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 기능을 확장할 수 있다.
   
- 다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 **래퍼 클래스**라고 한다.
- 다른 Set에 계측 기능을 덧씌운다는 뜻에서 **데코레이터 패턴**이라고 한다.
- 컴포지션과 전달의 조합은 넓은 의미로 위임(delegation)이라고 부른다.

### 정리
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 사용한다.
- 상속의 취약점을 피하려면 컴포지션과 전달을 사용하자.
- 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 좋다.
