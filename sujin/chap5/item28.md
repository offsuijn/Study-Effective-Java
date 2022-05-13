## 아이템 28. 배열보다는 리스트를 사용하라

## 배열과 제너릭 타입의 중요한 차이
### 1. 공변
배열은 **공변**이다.
- Sub가 Super의 하위 타입이라면 배열 `Sub[]`는 배열 `Super[]`의 하위 타입이 된다.

반면 제네릭은 **불공변**이다.
- `List<Object>`는 `List<String>`의 하위 타입도 아니고 상위 타입도 되지 않는다.

문제는 배열에 있다!

아래의 코드처럼 배열에서는 오류를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일 시점에 오류를 바로 확인할 수 있다.

#### 예제: 배열 - 런타임에 실패한다.
```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException 발생
```

#### 예제: 제너릭 - 컴파일되지 않는다.
```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이므로 컴파일부터 막힌다.
ol.add("타입이 달라 넣을 수 없다.");
```
    

### 2. 실체화
배열은 **실체화**된다.
- 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- 그래서 위의 예제에서 ArrayStoreException가 발생한 것이다.

제너릭은 실체화되지 않는다.
- 타입 정보가 런타임에는 소거된다.
- 원소 타입을 컴파일타임에만 검사하며 런타임 시점에는 알 수조차 없다. 
- 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있도록 해준다.

#### 이러한 차이로 인해 배열과 제네릭은 잘 어우러지지 못한다.
- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
   - `new List<E>[], new List<String>[], new E[]` // 오류 발생~!
   - 타입 안전성이 보장되지 않기 때문에 제너릭 배열 생성을 막는 것이다.
   
### 배열과 제너릭을 섞어 쓰다가 오류를 만난다면
**배열을 리스트로 대체하자!**

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 `E[]` 대신에 컬렉션인 `List<E>`를 사용하면 해결된다.

#### 예제: 형변환 오류가 날 수 있다.
```java
public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }
    
    // 이 메서드를 호출할 때마다 반환된 Object를 형변환해야 한다.
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
#### 예제: 배열 대신 리스트를 사용한다.
```java
class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

런타임에 형변환 오류를 발생시키지 않는다.

