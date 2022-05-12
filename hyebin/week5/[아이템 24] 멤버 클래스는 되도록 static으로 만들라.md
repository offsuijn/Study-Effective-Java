# [아이템 24] 멤버 클래스는 되도록 static으로 만들라

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들기**

[static(정적) 더 알아보기](https://www.notion.so/Static-94791feb1cbf46e0a6b56a86ad6c4a31) 

## 중첩 클래스

다른 클래스 안에 정의된 클래스

**자신을 정의한 클래스 안에서만 쓰여야 한다**

중첩 클래스 종류 : [ 정적 멤버 클래스 , (비정적) 멤버 클래스, 익명 클래스, 지역 클래스] 

## 정적 멤버 클래스 : 외부에서 상위 클래스 없이 접근 가능

- 바깥 클래스의 일반 멤버들에 접근 불가
- 바깥 클래스와 함께 쓰일 때만 유용한 `public 도우미` 클래스로 사용
- 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있음
- 외부에 공개하고 싶지 않고 바깥 클래스를 직접 사용하지 않으면 **private 정적 멤버 클래스**

**ex_ Calculator 클래스의 public 정적 멤버 클래스 Operation → Calculator.Operation.PLUS 로 활용** 

## 비정적 멤버 클래스 : 외부에서 상위 인스턴스 없이 접근 불가

- 정적 멤버 클래스에서 `staitc`이 안붙음
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결됨
- 비정적 멤버 클래스에서 **정규화된 `this`**를 사용해 바깥 인스턴스의 메서드 호출 OR 바깥 인스턴스 참조 가능
    
    정규화된 this 란? **클래스명.this** 형태로 바깥 클래스의 이름을 명시해서 참조 
    
- 바깥 인스턴스와 독립적으로 생성할 수 없음
- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화 될 때 확립 , 변경 불가
    
    → 보통은 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지지만 직접 *바깥클래스.new Member Class(args)*를 호출할 수도 있음
    
- 어댑터를 정의할 때 주로 사용
    
    어떤 클래스의 인스턴스를 감싸 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 활용
    
    ex_Map의 keySet, entrySet, values 
    

## 내부 클래스를 웬만하면 `static` 클래스로 만들어야 하는 이유

⇒ 비정적 멤버 클래스는 인스턴스가 만들어지는 순간 자동으로 바깥 인스턴스로의 숨은 참조를 갖게 되기 때문에 

- 참조 저장 시간, 공간 낭비
- 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거 못함

## 익명 클래스

- 이름이 없음
- 바깥 클래스의 멤버가 아님
- 멤버와 달리 쓰이는 시점에 선언과 동시에 인스턴스를 만듦
- 정적 문맥에서도 `static final` 선언된 상수 변수 이외의 정적 멤버는 가질 수 없음
- 정적 팩터리 메서드를 구현할 때도 사용

```java
List<Integer> list = Arrays.asList(10, 5, 6, 7, 1, 3, 4);
Collections.sort(list, new Comparator<Integer>() {
@Override
public int compare(Integer o1, Integer o2) {
return Integer.compare(o1, o2);
}
});
System.out.println(list);
```

Java8부터 람다가 나오고 아래와 같이 사용한다

```java
Collections.sort(list, (o1, o2) -> Integer.compare(o1, o2));
Collections.sort(list, Comparator.comparingInt(o -> o));

```

## 지역클래스

- 지역 변수를 선언할 수 있는 곳이라면 어디든 선언 가능, 유효범위도 지역 변수와 같음
- 멤버 클래스처럼 이름이 있고 반복해서 사용 가능
- 정적 멤버는 가질 수 없음

```java
public class TestThread {
public static void main(String[] args) {
Thread goodNightThread = new Thread(() -> {
try {
for (int i = 0; i <10; i++){
Thread.sleep(1000);
System.out.println("양 " + i + "마리...");
}
} catch (InterruptedException e) {
e.printStackTrace();
}
});
goodNightThread.start();
}
}

```

## 정리

- 중첩 클래스에는 네 가지(정적 멤버 클래스, 비정적 멤버 클래스, 익명 클래스, 지역 클래스)가 있으며 각각의 쓰임이 다르다.
- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자.
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않다면 지역 클래스로 만들자.
