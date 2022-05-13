# [아이템 14] Comparable을 구현할지 고려하라

## compareTo 와 equals 결과가 같도록 하자

**일관되지 않은 사례**

`BigDecimal` ”1.0” 과 “1.00” 은 equals 비교에서 서로 다른 값이라고 결과가 나오지만 compareTo로 값을 비교했을 때는 같다고 나온다.

### Comparable 은 타입을 인수로 받는 제네릭 인터페이스 : Comparable<Class>

따라서 CompareTo 메서드의 인수 타입은 컴파일 타임에 정해진다.

인수의 타입을 확인하거나 형변환할 필요가 없다.

## 비교자 Comparator

Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자를 대신 사용한다.

## compareTo 메서드에서 <와 >연산자 쓰지말기

### 대신  `compare` 메서드 사용하기

```java
return Integer.compare(this.value,o.value);
```

### 대신 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드 사용하기

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
comparingInt((PhoneNumber pn) -> pn.areaCode)
.thenComparingInt(pn->pn.prefix)
.thenComparingInt(pn->pn.lineNum);

public int compareTo(PhoneNumber pn){
	return COMPARATOR.compare(this,pn);
}
```
