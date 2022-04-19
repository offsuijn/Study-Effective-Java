## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
### 점층적 생성자 패턴

필수 매개변수만 받는 생성자, 필수 매개변수와 선택매개변수 1개를 받는 생성자... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.

```java
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    
    ...
```

단점 
 - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

### 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정한다.

단점
- 객체 하나를 만들기 위해서 메서드를 여러개 호출해야하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진다.
- 클래스를 불변으로 만들 수 없다.

### 빌더 패턴

필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻은 뒤, 빌더 객체가 제공하는 일종의 세터 메서드로 원하는 매개변수를 설정한다. 
매개변수가 없는 build 메서드를 호출해 객체를 얻는다.
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
```
빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

단점
- 성능에 민감한 상황에서는 빌더 생성 비용이 문제가 될 수 있다.
