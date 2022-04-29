## [아이템 8] finalizer와 cleaner 사용을 피하라

자바 언어 명세는 finalizer와 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 못한다. 따라서 **프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.** 

finalizer와 cleaner는 가비지 컬렉터의 효율을 떨어뜨리기 때문에 심각한 성능 문제도 동반한다.

**AutoCloseable** 

`try-with-resources` 형태는 아래 코드와 같이 try()의 괄호 안에서 try 문이 끝날 때 종료(close)할 객체를 선언해주는 형태이다. 이 패턴의 장점은 의도한 코드대로 진행이 다 되고 난 후 객체를 해제하고 싶을 때 복잡한 예외처리 없이 코드를 짤 수 있다는 것이다. 

다만 `try-with-resources` 형태에서 자동으로 `close()`메서드가 동작하기 위해서는 해당 인스턴스가 `Autocloseable`을 구현하고 있어야 한다. 

```java
public class AutoCloseableExample {
    public static void main(String[] args) {
        try (CustomResource customResource = new CustomResource()){
            customResource.doSomething();
        } catch (Exception e){
            e.printStackTrace();
        }
    }
}

public class CustomResource implements AutoCloseable {

    public void doSomething(){
        System.out.println("Do something ...");
    }

    @Override
    public void close() throws Exception {
        System.out.println("CustomResource Closed!");
    }
}
```

**굳이 cleaner와 finalizer을 사용하는 경우?**

1. AutoCloseable 에서 close 메서드가 동작하지 않을 상황을 대비한 안전망
    
     그러나 값어치가 있는지 잘 따져봐야 한다고 하는 걸로 보아 딱히 추천하지는 않는 것 같다.
    
2. 네이티브 객체 회수 할 때 
    
    네이티브 객체는 자바 객체가 아니니 가비지 컬렉터가 그 존재를 모르기 때문에 cleaner와 finalizer가 회수해줘야 한다고 한다. 그러나 이 경우에도 위험 부담은 여전히 존재하기 때문에 **네이티브 객체가 심각한 자원을 가지고 있어서 반드시 회수해줘야 하는 경우**에만 사용하기
    
