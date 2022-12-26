# [아이템2] 생성자에 매개변수가 많다면 빌더를 고려하라.

`[아이템1]` 에서 살펴보았듯이 생성자와 정적 팩터리에는 문제점이 있다. 바로 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다.<br>

다음과 같은 클래스가 있다고 해보자.
```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    // ...
}
```

## 점층적 생성자 패턴 | 생성자 체이닝

이렇게 초기화 시켜주어야 하는 필드가 많을 때 점층적 생성자 패턴이나 생성자 체이닝을 고려할 수 있다.
```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, 
                            int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, 
                            int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, 
                            int fat, int sodium, int carbohydrate) {
        this.servingsize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

`NuturitionFacts` 클래스의 인스턴스를 생성하려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.

하지만 여기서 매개변수가 더 늘어나게 되면 코드를 작성하거나 읽기 어려워지는 단점이 존재한다.

## 자바빈즈 패턴

다른 방법으로는 자바빈즈 패턴을 사용할 수 있다. 매개변수가 없는 생성자를 만든 후에 `setter`를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

```java
public class NutritionFacts {

    private final int servingSize = -1;
    private final int servings = -1;
    private final int calories = 0;
    private final int fat = 0;
    private final int sodium = 0;
    private final int carbohydrate = 0;

    public NuturitionFacts() {}
    
    // setter methods..
    // ...

    public static NuturitionFacts of(int servingSize, int servings, 
            int calories, int fat, int sodium, int carbohydrate) {
        NutritionFacts nf = new NutritionFacts();
        nf.setServingSize(servingSize);
        nf.setServings(servings);
        nf.setCalories(calories);
        nf.setFat(fat);
        nf.setSodium(sodium);
        nf.setCarbohydrate(carbohydrate);

        return nf;
    }
}
```
자바빈즈 패턴을 사용했을 때 완전한 객체를 만드려면 메서드를 여러번 호출해야 한다는 문제가 있다.
또한 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다. (필수 요소들을 초기화하지 못한 상태로 객체가 생성될 가능성이 있기 떄문이다.)<br>
이런 문제 때문에 클래스를 불변으로 만들 수 없고 스레드 안정성을 얻으려면 프로그래머가 추가적인 작업을 해주어야 한다.

## 빌더 패턴
빌더 패턴은 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴이다. 클라이언트는 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 이후 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 `build` 메서드를 호출해 필요한 객체를 얻는다.

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 필드
        private final int servingSize;
        private final int servings;

        // 선택 필드 - 기본값으로 초기화
        private int calories     = 0;
        private int fat          = 0;
        private int sodium       = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) { calories = val; return this; }
        public Builder fat(int val) { fat = val; return this; }
        public Builder sodium(int val) { sodium = val; return this; }
        public Builder carbohydrate(int val) { carbohydrate = val; return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
`NutritionFacts` 클래스의 내부 클래스인 `Builder`의 메서드들을 살펴보면 `setter`와 비슷하지만 `Builder` 타입 그 자체를 반환하고 있는 것을 볼 수 있다.
이후 플루언트 API 혹은 메서드 체이닝을 이용해서 객체를 생성한다.

```java
NutritionFacts nutritionFacts = new Builder(1, 1)
            .calories(342)
            .fat(10)
            .sodium(35)
            .carbohydrate(27)
            .build();
```

이 클라이언트 코드는 쓰기 쉬울 뿐만 아니라 간결하고, 자바빈즈 패턴보다 안전한다.

## 계층형 빌더
빌더 패턴은 계층적으로 설계된 클래스와 사용하기 좋다. 각 계층의 클래스에 관련 빌더를 `멤버`로 정의한다. 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    // 재귀적인 타입제한을 사용 -> 타입이 자기 자신의 generic 타입이기 때문
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```

`Pizza` 추상 클래스는 추상 빌더를 내부 static 클래스로 두고 있다. `Pizza.Builder` 클래스는 `self()` 메서드를 `abstract` 메서드로 두고, 이를 호출하는 클래스 자기 자신을 반환하고 있다. 

만약 아래 코드와 같이 `this` 를 반환하면 `Pizza` 타입을 반환하기 때문에 하위 클래스에서는 불필요한 타입 캐스팅이 필요해진다. 그렇기 때문에 위와 같이 `return self()`를 통해 하위 클래스의 `this`를 반환하게 한다.

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public Builder<T> addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return this;    // 바뀐 부분!!
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
}
```

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> { // <> 안에 있는 Builder는 NyPizza의 Builder
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

```java
    NyPizza pizza = (NyPizza) new NyPizza.Builder(SMALL)    // .Builder 는 결국 Pizza의 Builder를 반환 -> 불필요한 타입 캐스팅 발생!!
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();
```

그래서 `self` 메서드를 사용해서 `Builder`의 하위 클래스 타입의 `Builder`를 반환해서 사용할 수 있도록 한다.
이렇게 함으로써 얻을 수 있는 또 다른 이점은 하위 클래스의 메서드를 사용할 수 있다는 것이다.
`Calzone` 클래스는 자신만의 `sauceInside` 메서드를 두고 있는데, 위와 같이 하위 클래스 타입의 `Builder`를 반환했기 때문에 이를 사용할 수 있게 된다.

```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)",
                toppings, sauceInside ? "안" : "바깥");
    }
}
```
```java
    Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
```

## 정리
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 나을 수도 있다. 빌더는 점층적 생성자보다 코드의 가독성이 높고, 자바빈즈 패턴보다 안전하다.

## IllegalArgumentException
책 19p에 `IllegalArgumentException` 대한 언급이 나온다. 이 예외에 대해 간단히 알아보도록 한다.<br>
`IllegalArgumentException`은 `RuntimeException`의 하위 클래스로 적합하지 않거나 적절하지 못한 인자를 메서드로 넘겨주었을 때 발생하는 예외이다. 

### CheckedException과 UnCheckedException
`UnCheckedException`은 `RuntimeException`을 상속받은 예외들을 말하고 `CheckedException`은 `Exception` 클래스 중 `RuntimeException`을 제외한 예외를 말한다. `CheckedException`은 메서드 선언부에 `throws` 키워드를 이용해 명시적으로 예외가 발생한다고 말하거나 `try-catch` 블럭을 이용해 예외를 처리해야 한다. 그렇지 않으면 컴파일 에러가 발생하게 된다. 반면 `UnCheckedException`은 메서드 선언부에 명시적으로 알리지 않아도 되고, 반드시 `try-catch` 블럭을 통해 처리하지 않아도 된다. <br>
그런데 간혹 `UnCheckedException` 도 메서드 선언부에 선언하는 경우도 있는데, 이는 선언한 메서드가 `UnCheckedException`을 발생시킬 수 있다는 것을 명시적으로 알려주기 위해서라고 생각한다. 이로써 메서드를 호출하는 쪽에서는 이 예외를 가지고 무엇을 할 지 결정할 수 있다.

[오라클 docs, Unchecked Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)
