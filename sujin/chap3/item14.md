## 아이템 14. Comparable을 구현할지 고려하라
Comparable의 compareTo() 메소드는 단순 동치성 비교뿐만 아니라 순서까지 비교할 수 있으며 제네릭하다.

### compareTo() 메소드의 일반규약
- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다(따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다).
- Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0이다.
- Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) > 0이다.
- (x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.

마지막 규약은 필수는 아니지만 꼭 지키는 것이 좋다.
### compareTo() 메소드 작성 요령
- compareTo() 메서드는 각 필드의 순서를 비교한다.
- 객체 참조 필드를 비교하려면 compareTo() 메서드를 재귀적으로 호출한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.
#### 예시: 객체 참조 필드가 하나뿐인 비교자

```java
public class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    // 나머지 코드 생략
}
```

#### 예시: 기본 타입 필드가 여럿일 때의 비교자

```java
public int compareTo(Number n) {
        int result = Integer.compare(first, n.first); // 가장 중요한 필드
        if (result == 0) {
            result = Integer.compare(second, n.second); // 두 번째로 중요한 필드

            if (result == 0) {
                result = Integer.compare(third, n.third); // 세 번째로 중요한 필드
            }
        }

        return result;
    }
```

- 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해나가는 것이 좋다.
- 비교 결과가 0이 아니라면 즉시 순서가 결정되면서 그 결과를 반환한다.


## 정리
- 순서를 고려해야 하는 값 클래스를 작성할 경우 꼭 Comparable 인터페이스를 구현해야 한다.
- 그 인스턴스들은 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러져야 한다.
- compareTo()는 필드의 값을 비교할 때 ‘<‘와 ‘>’ 연산자는 쓰지 말도록 한다.
   - 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용한다.
