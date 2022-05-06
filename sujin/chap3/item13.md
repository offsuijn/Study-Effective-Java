## 아이템 13. clone 재정의는 주의해서 진행하라
Cloneable 인터페이스는 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.

하지만 clone() 메서드는 Cloneable 인터페이스가 아닌 Object 클래스에 선언이 되어있다.

### Cloneable
메서드가 하나도 선언되어 있지 않은 Cloneable 인터페이스는 Object의 protected 메서드인 clone() 메서드의 동작 방식을 결정힌다.

Cloneable을 구현한 클래스의 인스턴스에서 clone()을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, Cloneable을 구현하지 않은 클래스에서 이를 호출하면 CloneNotSupportedException을 던진다.

clone() 메소드는 클래스에 정의된 모든 필드가 기본 타입이거나 불변 객체를 참조하는 경우에 완벽하게 맞는다.

#### 예시: 가변 객체를 참조하지 않는 상태용 clone()
```java
@Override
public PhoneNumber clone() {
	 try {
 		return (PhoneNumber) super.clone();
 	} catch (CloneNotSupportedException e) {
		 throw new AssertionError(); // 일어날 수 X
	 }
```
편안하네요.

### 가변 객체를 참조할 때의 clone()

#### 1. clone()을 재귀적으로 호출
```java
@Override
public Stack clone() {
	try {
 		Stack result = (Stack) super.clone();
 		result.elements = elements.clone();
 		return result;
 	 } catch (CloneNotSupportedException e) {
 throw new AssertionError();
 }
```
위의 코드와 같이 clone() 메서드가 super.clone()의 결과를 그대로 반환한다면 반환된 인스턴스의 size 필드는 올바른 값을 갖지만, elements 필드는 원본 Stack 인스턴스와 같은 배열을 참조한다.
따라서 원본이나 복제본 인스턴스 중 하나를 수정하면 다른 하나도 같이 수정되어 불변식을 해치게 된다.

**배열의 clone()은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환하므로 형변환이 필요없다.**

#### 2. 연결 리스트를 재귀적으로 복사
```java
public class HashTable implements Cloneable {
	 private Entry[] buckets = ...;
 
	 private static class Entry {
		 final Object key;
		 Object value;
		 Entry next;
         
         this.key = key;
         this.value = value;
         this.next = next;
	 }
 
 // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
 Entry deepCopy() {
	 return new Entry(key, value,
	 next == null ? null : next.deepCopy());
 }
 
 @Override
 public HashTable clone() {
	 try {
		 HashTable result = (HashTable) super.clone();
		 result.buckets = new Entry[buckets.length];
		 for (int i = 0; i < buckets.length; i++) {
			 if (buckets[i] != null)
				 result.buckets[i] = buckets[i].deepCopy();
		 }
		return result;
	 } catch (CloneNotSupportedException e) {
 		throw new AssertionError();
 	 }
 }
```
재귀적인 호출로 인해 연결 리스트의 원소 수만큼 스택 프레임을 소비하기 때문에 연결 리스트가 길다면 StackOverflow를 일으킬 위험이 있다.


#### 3. 연결리스트를 반복적으로 복사
```java
Entry deepCopy() {
 	Entry result = new Entiry(key, value, next);
 	for (Entry p = result; p.next != null; p p.next)
 		p.next = new Entry(p.next.key, p.next.value, p.next.next);
 	return result;
}
```
deepCopy()를 재귀적으로 호출하는 대신 반복자를 써서 순회하는 방향으로 수정했다.


## 정리
- 상속용 클래스는 Cloneable를 구현해서는 안 된다.
- Cloneable를 구현한 Threadsafe 클래스를 작성할 때는 clone 메소드 역시 적절히 동기화 해주어야 한다.
