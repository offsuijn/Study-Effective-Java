## [아이템 6] 불필요한 객체 생성을 피하라

생성자 대신 정적 팩터리 메서드를 제공하면 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다. 

**생성 비용이 아주 비싼 객체 예시**

```java
static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
```

이 코드가 겉보기에는 문제가 없어 보이지만, `String.matches()`메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려진다.

따라서 이렇게 Pattern 인스턴스를 만드는게 비싸다면 매번 새로운 인스턴스를 의미없게 찍어내는 matches 메서드를 그대로 사용할 것이 아니라, Pattern 객체를 필요한 만큼만 따로 만들어주는 것이 좋다. 아래와 같이.

```java
public class RomanNumber {

    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

**같은 객체를 리턴하는 메서드 예시**

어댑터의 경우 실제 작업은 뒷단의 객체에 위임하고 자신은 인터페이스 역할만 하기 때문에 뒷단 객체 하나당 어댑터 하나만 생성한다.

따라서 아래 예시의 경우 keySet을 호출할 때마다 같은 객체가 리턴된다. 

```java
public class KeySetDemo {

    public static void main(String[] args) {
        Map<Long, String> members = new HashMap<>();
        members.put(1L, "member1");
        members.put(2L, "member2");

        Set<Long> set1 = members.keySet();
        Set<Long> set2 = members.keySet();

        System.out.println(set1 == set2); // true

        set1.remove(1L);
        System.out.println(set2.size()); // 1
        System.out.println(members.size()); // 1
    }
}
```

**오토박싱은 늘 신중하게!**

불필요한 오토박싱은 불필요한 인스턴스만 만들어낼 뿐이다. 

```java
public class AutoBoxingDemo {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        Long sum = 0l; //Long으로 박싱하는 것이 불필요, 성능 저하 
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i;
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - start);
    }
}
```

**따라서 박싱된 기본 타입보다는 프리미티브 타입을 사용하고 불필요한 오토박싱이 코드에 침투하지 않도록 한다.**

중요한 것은, **방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실이다.** 따라서 성능 개선도 좋지만 필요 없는 객체생성을 최소화 하겠다고 무리해서 재사용을 구현하려 하다가 더 큰 버그를 만나지 않도록 주의하자.
