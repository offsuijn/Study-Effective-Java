# 아이템 33. 타입 안전 이종 컨테이너를 고려하라

`Set<T>`과 같이 일반적인 제너릭 형태에서는 한 컨테이너가 다룰 수 있는 매개변수의 수가 제한된다.
만약 이보다 유연한 수단이 필요할 때 타입 안전 이종 컨테이너 패턴을 사용할 수 있다.

## 타입 안전 이종 컨테이너 패턴
컨테이너 대신 키를 매개변수화하고, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 것을 타입 안전 이종 컨테이너 패턴이라고 한다.

#### 예제: 타입 안전 이종 컨테이너 패턴 - API
```java
// 각 타입별로 즐겨찾는 인스턴스를 저장하고 검색할 수 있는 클래스
public class Favorites{
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type)
}
```
각 타입의 Class 객체를 매개변수화하여 키 역할로 사용한다.
컴파일 타임 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴(`Class<T>`)을 타입 토큰이라 부른다.

#### 예제: 타입 안전 이종 컨테이너 패턴 - 클라이언트
```java
public static void main(Stringg[] args) {
	Favorites f = new Favorites();

	f.putFavorite(String.class, "JAVA");
    f.putFavorite(Integer.class, 123456);

	String favoriteString = f.getFavorite(String.class);
    Integer favoriteInteger = f.getFavorite(Integer.class);

	System.out.printf("%s", favoriteString); // JAVA 출력
    System.out.printf("%d", favoriteInteger); // 123456 출력
}
```
메서드들이 타입 토큰을 주고받으며 올바른 값을 반환한다.
맵과 달리 여러 가지 타입의 원소를 담을 수 있다.
따라서 Favorites는 타입 안전 이종 컨테이너라 부를 수 있다.

## 제약 2가지

### 1. Class 객체를 제너릭이 아닌 Raw Type으로 넘기면 타입 안정성이 쉽게 깨진다.
```java
f.putFavorite((Class) Integer.class, "Integer의 인스턴스가 아닙니다."); // 컴파일 에러가 뜨지 않음
int favoriteInteger = f.getFavorite(Integer.class) // ClassCastException 발생
```
putFavorite를 사용할 때는 정상적으로 동작하지만 getFavorite를 호출하면 ClassCastException이 발생한다.

타입 불변성을 어기는 일이 없도록 보장하려면, `putFavorite`메소드에서 동적 형변환 `type.cast()`를 해주면 된다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```


### 2. 실체화 불가 타입에는 사용할 수 없다.
`String`이나 `String[]`은 사용할 수 있어도 실체화 불가 타입인 `List<String>`은 저장할 수 없다. 
- `List<String>`용 Class 객체를 얻을 수 없기 때문에 코드가 컴파일되지 않는다.
- `List<String>.class`와 `List<Integer>.class`는 `List.class`라는 Class 객체를 반환한다. -> 대혼란 야기
