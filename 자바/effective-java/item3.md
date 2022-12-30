# [아이템3] private 생성자나 열거 타입으로 싱글턴임을 보증하라.

## 싱글턴이란?
싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

싱글턴을 만드는 방법은 세 가지가 있는데 다음과 같다.
* public static final 필드 방식의 싱글턴
* 정적 팩터리 방식의 싱글턴
* 열거 타입 방식의 싱글턴

각 방식의 장단점을 알아보자.

## public static final 필드 방식의 싱글턴
이 방법은 생성자는 `private`으로 감추고 유일한 인스턴스에 접근할 수 있는 수단으로 `public static` 멤버를 둔다.

```java
public class Elvis {
    
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public void sing() {
        System.out.println("I'll have a blue~ Christmas without you~");
    }
}
```
장점으로는 코드가 간결하고 싱글턴임을 API에 드러낼 수 있다는 것이다.<br>
단점은 다음과 같다.

* 싱글턴을 사용하는 클라이언트 테스트하기 어려워진다.
* 리플렉션으로 private 생성자를 호출할 수 있다.
* 역직렬화 할 때 새로운 인스턴스가 생성될 수 있다.

각 의미에 대해 알아보자.

### 클라이언트 테스트하기 어렵다는 말은 무엇일까?
```java
public class Concert {

    private boolean lightsOn;

    private boolean mainStateOpen;
    
    private Elvis elvis;

    public Concert(Elvis elvis) {
        this.elvis = elvis;
    }

    public void perform() {
        mainStateOpen = true;
        lightsOn = true;
        elvis.sing();
    }
}
```
위의 코드에서 `perform` 메서드에서 elvis.sing() 메서드를 호출하고 있다. 만약 elvis.sing()의 비용이 굉장히 크다고 가정하면, 우리는 `perform` 메서드를 테스트하고 싶을 때마다 elvis.sing() 을 호출해야 되는 문제가 발생한다. 이때 싱글턴 인스턴스를 가짜(Mock)로 대체해야 하지만, elvis는 싱글턴 객체이기 때문에 그렇게 하지 못한다.

```java
    @Test
    void perform() {
        Concert concert = new Concert(Elvis.INSTANCE);
        concert.perform();

        assertTrue(concert.isLightsOn());
        assertTrue(concert.isMainStateOpen());
    }
```

책에서도 다음과 같이 언급되어 있다.
```text
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다. 
```

대신 인터페이스로 타입을 정의하면 테스트를 진행할 수 있다.
```java
public interface IElvis {

    void leaveTheBuilding();

    void sing();
}

public class Elvis implements IElvis {
    
    public static final Elvis INSTANCE = new Elvis();
    
    private Elvis() {
    }

    @Override
    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    @Override
    public void sing() {
        System.out.println("I'll have a blue~ Christmas without you~");
    }
}
```

이후 가짜 Elvis 클래스를 만들어 테스트 할 수 있다.
```java
public class MockElvis implements IElvis {

    @Override
    public void leaveTheBuilding() {

    }

    @Override
    public void sing() {
        System.out.println("You ain't nothin' but a hound dog");
    }
}
```
```java
    @Test
    void performWithInterface() {
        Concert concert = new Concert(new MockElvis());
        concert.perform();

        assertTrue(concert.isLightsOn());
        assertTrue(concert.isMainStateOpen());
    }
```





### 리플렉션으로 private 생성자 호출?
책에서 다음과 같은 문장이 나온다.
```
권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
```

아래 코드를 먼저 보자.
```java
public static void main(String[] args) {
    try {
        // getConstructor() 의 경우 public한 생성자에만 접근이 가능
        Constructor<Elvis> declaredConstructor = Elvis.class.getDeclaredConstructor();
        declaredConstructor.setAccessible(true);        // true를 주지 않으면 private이기 때문에 생성자 호출 불가
        Elvis elvis1 = declaredConstructor.newInstance();
        Elvis elvis2 = declaredConstructor.newInstance();
        System.out.println(elvis1 == elvis2);           // false    // error!
        System.out.println(elvis1 == Elvis.INSTANCE);   // false
    } catch (NoSuchMethodException | InvocationTargetException | InstantiationException | IllegalAccessException e) {
        e.printStackTrace();
    }
}
```
먼저 `Elvis` 클래스에 대한 생성자 정보를 `getDeclaredConstructor` 메서드로 얻는다. 이후 `setAccessible` 메서드를 통해 private 생성자를 접근가능하도록 한다. 이후 `Elvis` 인스턴스를 만드는 과정을 보여주고 있다.

이처럼 Reflection API를 통해 private 생성자를 호출할 수 있다.
이를 막아야 한다면 `Elvis` 클래스의 생성자에 다음과 같은 코드를 추가할 수 있다.
```java
    private static boolean created;

    private Elvis() {
        if (created) {
            throw new UnsupportedOperationException("can't be created by constructor");
        }

        created = true;
    }
```

static 변수인 `created` 플래그 변수를 사용해 한 번 생성되었다면 더 이상 생성하지 못하도록 막는 방법도 고려할 수 있다.

### 역직렬화 할 때 새로운 인스턴스가 생성된다고?
```
싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.
```

`Elvis` 클래스가 `Serializable` 인터페이스를 구현하고 있을 때 직렬화 시킬 수 있다. 하지만 `Elvis` 인스턴스를 직렬화 시키고 다시 역직렬화 시켰을 때 다른 인스턴스가 생성된다.
```java
    public static void main(String[] args) {
        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
            out.writeObject(Elvis.INSTANCE);
        } catch (IOException e) {
            e.printStackTrace();
        }

        try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
            Elvis elvis = (Elvis) in.readObject();
            System.out.println(elvis == Elvis.INSTANCE);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```
```
false
```
이는 역직렬화 할 때 새로운 `Elvis` 인스턴스가 생성되기 때문이다. 이를 해결하기 위해 `readResolve` 메서드를 정의해야 하는데, 역직렬화를 수행할 때 `readResolve` 메서드가 호출되면서 새로운 인스턴스가 아닌 기존에 있는 INSTANCE를 반환하도록 할 수 있다.

```java
    public Object readResolve() {
        return INSTANCE;
    }
```
`readResolve` 메서드를 오버라이딩 한 것은 아니지만 마치 오버라이딩 한 것 처럼 동작한다.

## 정적 팩터리 방식의 싱글턴
어떤 클래스를 싱글턴으로 만드는 두 번째 방법은 정적 팩터리 방식의 싱글턴을 이용하는 것이다.

```java
public class Elvis implements Singer {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return  INSTANCE;
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public void sing() {
        System.out.println("I'll have a blue~ Christmas without you~");
    }
}
```
`Elvis.getInstance()`는 항상 똑같은 인스턴스를 반환하기 때문에 새로운 인스턴스를 만들지 않는다. 대신 위에서 public static final 방식과 마찬가지로 Reflection API와 직렬화의 단점은 그대로 가지고 있다. 대신 이 방식의 장점은 다음과 같다.

* API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
* 정적 팩터리를 제네릭 싱글턴 팩터리로 변경할 수 있다.
* 정적 팩터리의 메서드 참조를 공급자(Supplier)로 사용할 수 있다.

### API를 바꾸지 않는다?
private static final 은 그대로 둔 채, getInstance 메서드만 수정하면 싱글턴이 아니게 변경할 수 있다. 
```java
    public static Elvis getInstance() {
        return new Elvis();
    }
```

### 정적 팩터리를 제네릭 싱글턴 팩터리로 변경
위 `Elvis` 클래스를 제네릭으로 싱글턴 팩터리로 변경한 코드를 보자.

```java
public class MetaElvis<T> {
    private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();

    private MetaElvis() {}

    public static <T> MetaElvis<T> getInstance() {
        return (MetaElvis<T>) INSTANCE;
    }

    public void say(T t) {
        System.out.println(t);
    }
}
```
```java
    public static void main(String[] args) {
        MetaElvis<String> elvis1 = MetaElvis.getInstance();
        MetaElvis<Integer> elvis2 = MetaElvis.getInstance();

        System.out.println(elvis1);
        System.out.println(elvis2);
        System.out.println(elvis1.equals(elvis2));  // true
        // == 비교는 타입이 아예 다르기 때문에 불가
    }
```
이처럼 `Elvis` 클래스를 \<T>를 통해 제네릭 싱글턴 팩터리로 변경하였다. 이처럼 인스턴스는 동일하지만 원하는 타입으로 변경할 수 있다는 장점을 볼 수 있다. 여기서 `getInstance`가 하는 역할은 `MetaElvis` 클래스의 싱글턴 인스턴스를 원하는 타입으로 변경하는 역할을 수행하고 있다. 

### 메서드 참조를 공급자로 이용한다.
`getInstance` 메서드를 Java8에서 등장한 Functional Interface인 Supplier로 사용할 수 있다. Functional Interface는 람다 표현식과 메서드 참조에 대한 '타겟 타입'을 제공한다.

Supplier의 코드를 보면 다음과 같다.
```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```
굉장히 간단한 모습인데 메서드의 파라미터로 아무것도 받지 않고 T 타입의 인스턴스를 반환하는 get메서드만을 가지고 있다.
이 FunctionalInterface의 메서드 형태에 만족하기만 한다면 우리는 인터페이스를 구현하지 않더라도 이를 사용할 수 있게 된다. 아래 코드를 보자.

```java
public class Concert {

    public void start(Supplier<Singer> singerSupplier) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }

    public static void main(String[] args) {
        Concert concert = new Concert();
        concert.start(Elvis::getInstance);
    }
}
```
`Elivs::getInstance`가 바로 Supplier 인터페이스의 형태를 만족하고 있어 `start` 메서드의 파라미터로 사용할 수 있는 것이다. 원래의 형태는 `() -> Elvis.getInstance()` 인데 이것 자체가 Supplier를 구현하고 있는 익명 인스턴스라고 생각할 수 있고 위는 이를 메서드 참조로 바꾼 것이다. Java8 이전의 형태는 아래와 같다.
```java
concert.start(new Supplier<Singer>() {
    @Override
    public Singer get() {
        return Elvis.getInstance();
    }
});
```

메서드 참조를 좀 더 자세히 공부해보자. 메서드 참조는 메서드 하나만 호출하는 람다 표현식을 줄여쓰는 방법이다.
메서드 참조의 종류에는 네 가지가 있다.
* 스태틱 메서드 레퍼런스
* 인스턴스 메서드 레퍼런스
* 임의 객체의 인스턴스 메서드 레퍼런스
* 생성자 레퍼런스

| Kind                                                        | Syntax                               | Examples                                                          |
|-------------------------------------------------------------|--------------------------------------|-------------------------------------------------------------------|
| static method reference                                     | ContainingClass::staticMethodName    | Person::compareByAge <br> MethodReferencesExamples::appendStrings |
| instance method reference                                   | containingObject::instanceMethodName | myComparisonProvider::compareByName <br> myApp::appendStrings2    | 
| instance method of an arbitrary object of a particluar type | ContainingType::methodName           | String::compareToIgnoreCase <br> String::concat                   |
| Reference to a constructor                                  | ClassName::new                       | HashSet::new                                                      |

해당 문서를 보면 예제 코드가 존재한다.
[Oracle - method references](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)

## 열거 타입 방식의 싱글턴
싱글턴을 만드는 세 번째 방법은 열거 타입을 이용하는 것이다.
```java
public enum Elvis implements IElvis {
    INSTANCE;

    @Override
    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께~");
    }

    @Override
    public void sing() {

    }
}
```

이 방식을 사용하면 직렬화, 리플렉션 등의 문제를 간결하게 해결할 수 있다. 대부분의 상황에서는 원소가 하나뿐인 열거타입이 싱글턴을 만드는 가장 좋은 방법이다.

## 참고 자료
[인프런 - 이펙티브 자바 완벽 공략](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1/dashboard)