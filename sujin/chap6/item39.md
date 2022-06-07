# Item 39 : 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴
- 변수나 함수의 이름을 일관된 방식으로 작성하는 패턴
- 전통적으로 도구나 프레임워크에서 특별히 다뤄야할 프로그램 요소를 구분하기 위해 사용됨

예를 들면 JUnit3 에서는 테스트 메서드 이름을 test로 시작하게끔 했다. 
이러한 명명 패턴 방식은 효과적이지만 단점도 크다.

## 명명 패턴의 단점
### 첫 번째, 오타가 나면 안 된다.
test로 시작해야하는데 실수로 이름을 tsetSafetyOverride로 지으면 테스트 메서드로 인식하지 못하고 테스트를 수행하지 않는다.
### 두 번째, 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
JUnit3의 명명 패턴인 'test'를 메서드가 아닌 클래스의 이름으로 지음으로써 해당 클래스의 모든 테스트 메서드가 수행되길 바랄 수 있지만 JUnit은 클래스 이름에는 관심이 없다. 따라서 개발자가 의도한 테스트는 전혀 수행되지 않는다.
### 세 번째, 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
특정 예외를 던져야 성공하는 테스트가 있을 때, 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있지만 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.
***
애너테이션을 사용하면 위의 단점을 모두 해결할 수 있다.

## 마커 애너테이션
- 아무 매개변수 없이 단순히 대상에 마킹하는 용도
```java
/**
* @Test
* 테스트 메서드임을 선언하는 애너테이션이다.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```

## 메타 애너테이션
- 애너테이션 선언에 다는 애너테이션
- 위 예제에서는 @Retention과 @Target이 메타 애너테이션에 해당함


- @Retention(RetentionPolicy.RUNTIME) - 보존 정책
   - @Test가 런타임에도 유지되어야 한다는 표시

- @Target(ElementType.METHOD) - 적용 대상
   - @Test가 반드시 메서드에 선언되어야 한다는 표시


## 마커 애너테이션 처리기
- 마커 애너테이션은 적절한 애너테이션 처리기가 필요함
- Test라는 이름에 오타가 있거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내어주는 역할을 함

```java
// 마커 애너테이션 처리기
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) {
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n",
                      passed, tests - passed);
  }
}
```
리플렉션을 이용하여 마커 애너테이션을 찾고, 예외 발생 시 InvocationTargetException으로 Wrapping 되어 해당 예외에 담긴 실패 정보를 추출해서 출력한다.

## 정리
>애너테이션으로 처리할 수 있다면 명명 패턴을 사용할 이유는 없다.
>
>자바 프로그래머라면 자바가 제공하는 애너테이션 타입들은 사용하자!
