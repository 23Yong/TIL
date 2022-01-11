# enum
생활속에서 몇 가지로 한정된 값만을 가지는 데이터를 흔히 찾아볼 수 있다. 이때 유용하게 사용할 수 있는 것이 enum이다. 

개발을 진행하는데 있어서도 일반적인 문자열을 사용하는 것보다 안전하다. 또한 C, C++과 달리 Enum이 클래스이기 때문에 많은 장점을 갖는다고 생각한다.

## 열거타입의 선언
열거 타입을 선언하기 위해선 열거타입의 이름을 정하고 열거타입의 이름으로 소스파일(.java)를 생성해야 한다. 

```java
public enum Direction {
    EAST, WEST, SOUTH, NORTH
}
```

다음과 같이 열거 타입을 선언하고 열거 상수를 선언하는데, 관례적으로 열거 상수는 모두 대문자로 작성한다.

## 열거 타입 변수
열거 타입을 선언하고 사용해보자. 열거 타입도 결국 하나의 데이터 타입이기 때문에 변수를 사용해야 하고 다음과 같이 사용할 수 있다.
```java
Direction direction;
```

이 변수에 열거 상수를 저장할 수 있는데, 열거 상수는 단독으로 사용할 수 없기 때문에 열거타입.열거상수 형태로 사용한다.
```java
Direction direction = Direction.EAST;
```

이를 이제 사용해보자.

```java
public class EnumDirectionExample {

    Direction direction;

    public EnumDirectionExample(Direction direction) {
        this.direction = direction;
    }

    public void testDirection() {
        switch (direction) {
            case EAST:
                System.out.println("동쪽 입니다.");
                break;
            case WEST:
                System.out.println("서쪽 입니다.");
                break;
            case NORTH:
                System.out.println("북쪽 입니다.");
                break;
            case SOUTH:
                System.out.println("남쪽 입니다.");
                break;
        }
    }

    public static void main(String[] args) {
        EnumDirectionExample ede = new EnumDirectionExample(Direction.EAST);
        ede.testDirection();

        ede = new EnumDirectionExample(Direction.WEST);
        ede.testDirection();

        ede = new EnumDirectionExample(Direction.SOUTH);
        ede.testDirection();

        ede = new EnumDirectionExample(Direction.NORTH);
        ede.testDirection();
    }
}
```
```
동쪽 입니다.
서쪽 입니다.
남쪽 입니다.
북쪽 입니다.
```

열거타입도 참조타입이기 때문에 열거 타입 변수에 null을 저장할 수 있다.
다음과 같이 열거 타입 변수를 선언했다고 생각해보자.
```java
Direction direction = Direction.EAST;
```
열거 타입 변수 direction은 스택 영역에 생성되고, 열거 타입 Direction의 EAST, WEST, SOUTH, NORTH은 객체이기 때문에 결국 힙 영역에 생성된다.
이를 direction이 참조하게 되는 것이다.

실제로 테스트를 해보면
```java
public class EnumDirectionExample {

    Direction direction;

    public EnumDirectionExample(Direction direction) {
        this.direction = direction;
    }

    public static void main(String[] args) {
        EnumDirectionExample ede = new EnumDirectionExample(Direction.EAST);
        System.out.println(ede.direction == Direction.EAST);
    }
}
```
```
true
```
같은 Direction 객체를 참조하기 때문에 true가 나오는 것을 확인할 수 있다.

## 열거 객체의 메서드
열거 객체는 열거 상수의 문자열을 내부 데이터로 가지고 있는데, 열거 객체가 가지고 있는 메서드들을 살펴보자.

- String ```name()```
    - 열거 객체의 문자열을 리턴한다.
- int ```ordinal()```
    - 열거 객체의 순번을 리턴 (0부터 시작)
- int ```compareTo()```
    - 열거 객체를 비교해서 순번 차이를 리턴
- 열거타입 ```valueOf(String name)```
    - 주어진 문자열의 열거 객체를 리턴
    - valueOf 메서드에 없는 값을 넣으면 IllegalArgumentException이 발생한다.
- 열거 배열 ```values()```
    - 모든 열거 객체들을 배열로 리턴

예시로 살펴보자.
```java
public class EnumDirectionExample {

    Direction direction;

    public EnumDirectionExample(Direction direction) {
        this.direction = direction;
    }

    public void enumMethodTest() {
        System.out.println("열거 객체의 문자열은 " + direction.name());
        System.out.println("열거 객체의 순번은 " + direction.ordinal());
        System.out.println("열거 객체의 순번 차이는 " + direction.compareTo(direction));
        System.out.println("열거 객체의 타입은 " + direction.valueOf("WEST"));
        System.out.println("모든 열거 객체는 " + Arrays.toString(direction.values()));
    }

    public static void main(String[] args) {
        EnumDirectionExample ede = new EnumDirectionExample(Direction.WEST);
        ede.enumMethodTest();
    }
}
```
```
열거 객체의 문자열은 WEST
열거 객체의 순번은 1
열거 객체의 순번 차이는 0
열거 객체의 타입은 WEST
모든 열거 객체는 [EAST, WEST, SOUTH, NORTH]
```

::dart:: 위의 모든 메서드들은 java.lang.Enum에 선언된 메서드이다. 이는 모든 열거 타입이 컴파일 시 Enum 클래스를 상속받게 되어있기 때문이다.

## 값으로 열거 객체 검색
이번에는 Week의 열거형을 정의해보자. 열거상수와 함께 값을 제공한 형태이다. 실제로 열거형 내부에 멤버 변수를 정의할 수 있다. 
```java
public enum Week {
    MONDAY("Monday"),
    TUESDAY("Tuesday"),
    WEDNESDAY("Wednesday"),
    THURSDAY("Thursday"),
    FRIDAY("Friday"),
    SATURDAY("Saturday"),
    SUNDAY("Sunday");

    private final String value;
    Week(String value) {
        this.value = value;
    }
}
```
다음과 같이 두 가지의 멤버 데이터를 둘 수도 있다.
```java
public enum Week {
    MONDAY("Monday", 1),
    TUESDAY("Tuesday", 2),
    WEDNESDAY("Wednesday", 3),
    THURSDAY("Thursday", 4),
    FRIDAY("Friday", 5),
    SATURDAY("Saturday", 6),
    SUNDAY("Sunday", 7);
    
    private final String value;
    private final int code;

    Week(String value, int code) {
        this.value = value;
        this.code = code;
    }
}
```

## 참고 자료
[밸덩 enum](https://www.baeldung.com/java-search-enum-values)

[이것이 자바다](http://www.yes24.com/Product/Goods/15651484)