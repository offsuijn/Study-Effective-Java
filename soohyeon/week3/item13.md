# 아이템 13 Clone 재정의는 주의해서 진행하라

**Cloneable이란?**

복제해도 되는 클래스임을 명시하는 용도의 mixin interface이다. 

믹스인이란 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.

하지만 clone 메서드가 Object에 선언되어 있고 protected 상태이기 때문에 문제가 발생한다. 

→ Cloneable을 구현하는 것 만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. 

이런 문제점에도 불구하고 Cloneable은 잘 쓰이기 때문에 알아두는 것이 좋다. 

**그럼 왜 사용되는 걸까?**

1. **Object의 protected 메서드인 clone의 동작 방식을 결정한다.** 
하단의 코드는 Cloneable 인터페이스이다. 
    
    ```java
    /**
     * A class implements the <code>Cloneable</code> interface to
     * indicate to the {@link java.lang.Object#clone()} method that it
     * is legal for that method to make a
     * field-for-field copy of instances of that class.
     * <p>
     * Invoking Object's clone method on an instance that does not implement the
     * <code>Cloneable</code> interface results in the exception
     * <code>CloneNotSupportedException</code> being thrown.
     * <p>
     * By convention, classes that implement this interface should override
     * {@code Object.clone} (which is protected) with a public method.
     * See {@link java.lang.Object#clone()} for details on overriding this
     * method.
     * <p>
     * Note that this interface does <i>not</i> contain the {@code clone} method.
     * Therefore, it is not possible to clone an object merely by virtue of the
     * fact that it implements this interface.  Even if the clone method is invoked
     * reflectively, there is no guarantee that it will succeed.
     *
     * @author  unascribed
     * @see     java.lang.CloneNotSupportedException
     * @see     java.lang.Object#clone()
     * @since   1.0
     */
    public interface Cloneable {
    }
    ```
    
    인터페이스에는 아무것도 구현되어있지 않지만, 실제로는 object의 clone 메서드의 동작 방식을 결정한다. 
    
    1. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환한다.
    2. Cloneable을 구현하지 않은 클래스의 인스턴스에서 clone을 호출하면 CloneNotSupportedException을 던진다. 
    
    1번을 보면 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지지라 기대한다. 하지만 clone 메서드가 Object에 선언되어 있고 protected 상태이기 때문에 제대로 복사가 안되는건가??  Cloneable을 구현하는 것 만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. 
    
    하지만 이를 만족하기 위해서는 위험하고 모순적인 메커니즘(생성자를 호출하지 않고도 객체를 생성할 수 있는 것)이 동반되어야 한다.
    
    ### Clone 메서드의 일반 규약
    
    1. x.clone() != x 는 참이다. 원본 객체와 복사 객체는 서로 **다른** 객체이다.
    2. x.clone().getClass() == x.getClass() 는 참이다. (객체가 참조하는 클래스가 같음) 하지만 반드시 만족해야 하는 것은 아니다. 왜?
    [https://woochan-autobiography.tistory.com/222](https://woochan-autobiography.tistory.com/222)
    3. x.clone().equals(x) 는 참이지만 필수는 아니다. super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면, 이 식은 참이다. 
    4. x.clone().getClass() == x.getClass() 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
    
    생성자 연쇄에서 강제성이 빠진 매커니즘과 비슷하다. clone을 재정의한 클래스가 final이면 걱정해야할 하위 클래스 (super.clone()으로 잘못된 클래스의 객체를 만드는)가 없으므로 관례를 무시해도 안전하다. 만약, clone 메소드가 super.clone 이 아닌 생성자를 호출해 얻은 인스턴스를 반환 하더라도 컴파일시에 문제가 되지않지만 해당 클래스의 하위 클래스에서 super.clone을 호출한다면 하위 클래스 타입 객체를 반환하지 않고 상위 클래스 타입 객체를 반환하여 문제가 생길 수 있다.
    
    ### clone에 완벽한 코드
    
    모든 필드가 기본 타입이거나 **불변 객체**를 참조하는 경우
    
    ```java
    class PhoneNumber implements Cloneable {
      @Override
      public PhoneNumber clone() {
        try {
          return (PhoneNumber) super.clone(); //phonenimber의 clone 메서드는 phonenumber를 반환시킴
        } //클라이언트가 형변환하지 않아도 되게끔 super.clone에서 얻은 객체를 반환하기 전에 phonenumber로 형변환시켜줌
    		catch(ClassNotSupportedException e) {
          //아무처리를 하지 않거나, RuntimeException으로 감싸는 것이 사용하기 편하다.
    			//왜? PhoneNumber가 cloneable을 구현하고 있기 때문에 이 catch문이 실행되지 않을 것을 알기 때문
        }
      }
    }
    ```
    
    쓸데없는 복사를 지양하기 위해서 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다. 
    
    ### clone에 재앙인 코드
    
    위의 구현에서 클래스가 가변 객체를 참조하는 순간 재앙이 시작됨
    
    아이템 7에서 언급한 stack 클래스를 사용해서 확인해보자.
    
    ```java
    private Object[] elements;
    		private int size = 0;
    		private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    		public StackExample() {
    				this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    		}
    
    		public void push(Object e) {
    				ensureCapacity();
    				elements[size++] = e;
    
    		}
    
    		public Object pop() {
    				if (size == 0) {
    						throw new EmptyStackException();
    				}
    				Object result = elements[--size];
    				elements[size] = null; // 다 쓴 참조 해제 return result;
    				return result;
    		}
    
    		// 원소를 위한 공간을 적어도 하나 이상 확보한다.
    		private void ensureCapacity() {
    				if (elements.length == size) {
    						elements = Arrays.copyOf(elements, 2 * size + 1);
    				}
    		}
    
    ```
    
    이 코드에서 clone메서드가 단순히 super.clone의 결과를 그대로 반환하게 되면 elements 필드는 원본 stack 인스턴스와 똑같은 배열을 참조할 것이다. 이는 원본이나 복제본 중 하나를 수정하면 불변식이 해쳐진다는 뜻이다. 때문에 프로그램이 이상하게 동작하거나 nullpointerException을 던진다는 것이다. 
    
    Stack 클래스의 유일한 생성자를 호출하면 이런 상황은 절대 일어나지 않는다.
    
    왜?
    
    clone 메서드는 사실상 생성자와 같은 효과를 내기 때문이다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 
    
    ### 방법1 - 배열의 clone을 재귀적으로 호출하기
    
    elements 배열의 clone을 재귀적으로 호출해 스택 내부 정보를 복사해 주는 것이 가장 쉬운 방법이다.
    
    ```java
    		@Override
    		public StackExample clone() {
    				try {
    						StackExample result = (StackExample) super.clone();
    						result.elements = elements.clone(); // elements 배열의 clone을 재귀적으로 호출
    						return result;
    				} catch (CloneNotSupportedException e) {
    						throw new AssertionError();
    				}
    		}
    ```
    
    - elements.clone()는 object 타입으로 형변환이 이제 필요가 없는데, 그 이유는 배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환하기 때문이다.
    
    → **배열을 복제할 때는 배열의 clone 메서드를 사용하는 것을 권장한다. (clone 기능을 제대로 사용하는 유일한 배열,,,)**
    
    - elements 필드가 final 이었다면 위 방식은 작동하지 않는다. 
    왜?
    애초에 final 필드에는 새로운 값을 할당할 수 없기 때문이다. 
    → ‘가변 객체를 참조하는 필드는 final로 선언하라’라는 일반 용법과 충돌함 (이럴거면 왜만든거???)
    
    ### 방법2
    
    해시테이블용  clone 메서드로 보자. 
    
    ```java
    public class HashTable implements Cloneable  {
      private Entry[] buckets = ...;
      private static class Entry {
        final Object key;
        Object value;
        Entry next;
    
        Entry(Object key, Object value, Entry next) {
          this.key = key;
          this.value = value;
          this.next = next;
        }
      }
    }
    ```
    
    방법1과 마찬가지로 단순히 버킷 배열의 clone을 재귀적으로 호출해보자
    
    ```java
    @Override public HashTable clone() {
    	try {
    		HashTable result = (HashTable) super.clone();
    		result.buckets = buckets.clone();
    		return result;
    	} catch (CloneNotSupportedException e)
    			throw new AssertionnError();
    	}
    }
    ```
    
    복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 이를 해결하려면 각 버킷을 구성하는 연결리스트를 복사해야 한다. 
    
    ```java
    public class HashTable implements Cloneable {
    		private Entry[] buckets = ...;
            
            private static class Entry {
            	final Object key;
                Object value;
                Entry next;
                
                Entry(Object key, Object value, Entry next) {
                	this.key = key;
                    this.value = value;
                    this.next = next;
                }
                
                // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
                Entry deepCopy() {
                	return new Entry(key, value,
                    	next == null ? null : next.deepCopy());
                }
            }
            
            @Override public HashTable clone() {
            	try {
                	HashTable result = (HashTable) super.clone();
                    result.buckets = new Entry[buckets.length];
                    for (int i = 0; i < buckets.length; i++)
                    	if (buckets[i] != null)
                        	result.buckets[i] = buckets[i].deepCopy();
                    return result;
                } catch (CloneNotSupportedException e) {
                	throw new AssertionError();
                }
            }
            ... // 나머지 코드 생략
    }
    ```
    
    - HashTable.Entry가 **깊은 복사**를 지원하도록 보강됨, HashTable의 clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은복사를 수행함, deepCopy처럼 연결리스트를 재귀적으로 복사함
    - 여기서 리스트가 길면 스택 오버플로를 일으킬 수 있어서 **반복자**를 써서 순회하는게 좋음
    
    ```java
    Entry deepCopy() {
    	Entry result = new Entry(key, value, next);
        for (Entry p = result; p.next != null; p = p.next)
        	p.next = new Entry(p.next.key, p.next.value, p.next.next);
        return result;
    }
    ```
    
    ****깊은 복사****
    
    **깊은 복사**는 실제 값을 **새로운 메모리 공간**에 복사하는 것을 의미한다.
    
    즉, 위처럼 **깊은 복사**를 하게 되면 원본과 복사본에 서로 영향을 미치지 못한다.
    
    이에 비해 **얕은 복사**의 경우 **주소 값**을 복사하기 때문에, **참고하고 있는 실제값**은 같다. 따라서 원본을 변경해도 복사본 역시 변경되는 상황이 발생한다. 
    
    위처럼 아예 HashTable의 경우 **깊은 복사**로 원본에 영향을 받지 않는다. 
    
    ### **복잡한 가변 객체**
    
    **복잡한 가변 객체**를 복제하는 방법은 super.clone을 호출하여 얻은 **객체의 모든 필드**를 **초기 상태**로 **설정**한 다음, 원본 객체의 상태를 다시 생성하는 **고수준 메서드**들을 호출한다. (해시테이블의 경우 put 메서드를 통해 호출해서 처리하는 등)
    
    - 단점
        - 간단하고 괜찮지만 저수준에서 바로 처리할 때보다는 느리다
        - 전체 Cloneable 아키텍처와 어울리지 않는다
    
    상속해서 쓰기 위한 클래스 설계 방식 두 가지 중 어느 쪽이든, 상속용 클래스는 Cloneable을 구현해서는 안된다. 
    
    다른 방법으로는, clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다. 다음과 같이 clone을 퇴화시켜놓으면 된다.
    
    ### **하위 클래스에서 Cloneable을 지원하지 못하게 하는 clone 메서드**
    
    ```java
    @Override protected final Object clone() throws CloneNotSupportedException {    throw new CloneNotSupportedException();}
    ```
    
    Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone메서드 역시 적절히 동기화해줘야 한다. Object의 clone 메서드는 동기화를 신경쓰지 않는다. 
    
    → super.clone 호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화해줘야 한다.
    
    <aside>
    💡 Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.
    
    </aside>
    
    이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경해야 한다. 이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다. 
    
    → 객체의 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고, 복사본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다. 
    
    하지만 다행히도 이처럼 복잡한 경우는 드물다고 한다. (...) 
    
    **결론**
    
    - Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다.
    - 그렇지 않은 상황에서는 복사 생성자와 복사 팩토리를 사용해서 더 나은 객체 복사 방식을 제공할 수 있다.
    
    ### **복사 생성자**
    
    복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
    
    ```
    public Yum(Yum yum) { ... };
    ```
    
    ### **복사 팩터리**
    
    복사 팩터리는 복사 생성자를 모방한 정적 팩터리이다.
    
    ```
    public static Yum newInstance(Yum yum) { ... };
    ```
    
    Cloneable/clone 방식보다 낫다. 
    
    복사 생성자와 복사 팩터리는 해당 클래스가 구현한 ‘인터페이스’ 타입의 인스턴스를 인수로 받을 수 있다. 
    
    또한 이를 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다. 
    
    ### 핵심 정리
    
    <aside>
    💡 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안되며, 새로운 클래스도 이를 구현해서는 안된다. 복제 기능은 배열 빼고는 생성자와 팩터리를 이용하는게 베스트
    
    </aside>
