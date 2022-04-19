
## 아이템 6. 불필요한 객체 생성을 피하라
```java
String s1 = new String("java...");
String s2 = "java..."; //인스턴스 재사용
```

### 정적 팩터리 메서드를 사용하자
생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.
따라서 Boolean(String) 생성자 대신 Boolean.valueOf(String) 메서드를 사용하는 것이 좋다.

### 생성 비용이 비싼 객체는 재사용하자
```java
//BEFORE
static boolean isKoreanWord(String s) {
    return s.matches("[가-힣]");
}

//AFTER
private static final Pattern KOREAN = Pattern.compile("[가-힣]");
static boolean isKoreanWord(String s) {
    return KOREAN.matcher(s).matches();
}
```
내부에서 생성되는 정규표현식용 Pattern 인스턴스는 한 번 사용하고 버려진다.
-> Pattern 인스턴스를 캐싱해두고 isKoreanWord가 호출될 때마다 이 인스턴스를 재사용한다.

### 오토박싱이 숨어들지 않도록 주의하자
오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다.

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
}
```
long 타입인 i가 Long 타입인 sum에 더해질 때마다 Long 인스턴스가 만들어진다.
박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.
