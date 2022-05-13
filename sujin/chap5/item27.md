## 아이템 27. 비검사 경고를 제거하라
제네릭을 사용하기 시작하면 수많은 컴파일러 경고들을 마주치게 된다.
#### 비검사 경고
- `warning : [unchecked]`
- casting을 할 때 검사를 하지 않았다고 뜨는 경고이다.
### 할 수 있는 한 모든 비검사 경고를 제거하라.
대부분의 비검사 경고는 쉽게 제거할 수 있다.
#### 예시: 컴파일러 오류가 난다.
`Set<Car> cars = new HashSet();`

```java
Venery.java:4: warning: [unchecked] unchecked conversion
                Set<Car> cars = new HashSet();
                                ^
  required: Set<Car>
  found:    HashSet
```

-> `Set<Car> cars = new HashSet<>();`

컴파일러가 알려준 대로 수정하면 경고가 사라진다.

### 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있으면,  @SuppressWarnings(“unchecked”)를 달아 경고를 숨기자.
- @SuppressWarnings은 지역변수 선언부터 클래스 전체까지 선언할 수 있지만, 가능한 가장 좁은 범위에 적용하자.
   - 절대로 클래스 전체에 적용해서는 안 된다.
- 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션은 지역변수 선언 쪽으로 옮기자.

#### 예시: 지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.
```java
public <T> T[] toArray(T[] a) {
   if (a.length < size) {
      // 수정 전 : 형변환 오류가 발생한다.
      return (T[]) Arrays.copyOf(elements, size, a.getClass());
      
      // 수정 후
      // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
      @SuppressWarnings("unchecked") T[] result = 
         (T[]) Arrays.copyOf(elements, size, a.getClass());
      return result; 
   } 
   System.arraycopy(elements, 0, a, 0, size);
   if (a.length > size)
      a[size] = null;
   return a; 
}
```
애너테이션은 선언에만 달 수 있기 때문에 return 문에는 @SuppressWarning을 다는 게 불가능하다.

**@SuppressWarnings(“unchecked”) 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.**
