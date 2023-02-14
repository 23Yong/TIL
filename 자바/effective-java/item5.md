# [아이템 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 의존 객체 주입
* 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

위 같은 클래스의 경우 의존 객체 주입을 사용해야한다.

> *의존 객체 주입* <br>
> 인스턴스를 생성할 때 필요한 자원을 생성자에 넘겨주는 방식


```java
public class SpellChecker {

    private final Dictionary dictionary;

    public SpellChecker(Dictionary dictionary) {
        this.dictionary = dictionary;
    }

    public SpellChecker(Supplier<Dictionary> dictionarySupplier) {
        this.dictionary = dictionarySupplier.get();
    }

    public boolean isValid(String word) {
        // SpellChecker 만의 코드
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        // SpellChecker 만의 코드
        return dictionary.closeWordsTo(typo);
    }
}
```

위의 `SpellChecker` 클래스는 의존 객체 주입 방식을 사용하고 있다.
생성자의 파라미터로 받는 `Dictionary` 타입이 인터페이스인 경우 모든 코드를 재사용가능하다.
또한 테스트 코드를 작성할 때도 가짜 Dictionary (Mock Dictionary)를 생성한 뒤 테스트를 진행할 수 있다.

이 방식의 변형으로 생성자에 자원 팩터리를 넘겨줄 수도 있다.
위의 `Supplier<T>` 타입을 파라미터로 받는 생성자에서 알 수 있듯이 get() 메서드를 통해 의존 객체 주입을 구현한 모습이다.

## 팩터리 메서드 패턴
이펙티브 자바 책에 다음과 같은 문장이 쓰여져 있다.
> `팩터리 메서드 패턴`을 구현한 것이다.

팩터리 메서드 패턴은 구체적인 클래스의 인스턴스를 만들지는 서브 클래스가 정하는 패턴이다.

[guru - design pattern](https://refactoring.guru/design-patterns/factory-method)

위의 사이트에 굉장히 잘 설명이 되어 있다.

![image](https://user-images.githubusercontent.com/66981851/218752073-0ff3fd56-e96f-46c9-b284-38aa0502b792.png)

위의 그림과 같이 `Logistics` 라는 인터페이스가 존재하고 구체적인 클래스들이 어떤 인스턴스를 만들지 결정하고 있다.<br>
이 패턴을 사용하면 만약 새로운 `Logistics` 타입의 구체 클래스가 생겨도 클라이언트는 코드를 변경할 필요가 없다.
그저 `createTransport()` 메서드를 호출하면 될 뿐이다.

## 한정적 와일드 카드 타입과 제너릭
이펙티브 자바에서 다음과 같은 코드가 주어진다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

여기서 `<? extends Tile>`이 한정적 와일드 카드 타입과 제너릭이다.

자바에서 제너릭을 사용할 때 일반적으로 아래와 같이 사용한다.
```java
class Transport<T> {

    private T some;
    // ...
}
```
이는 T타입의 some 변수를 멤버로 갖는 Transport 클래스이다.
그런데 만약 T 타입을 어떤 타입으로 제한하고 싶을 때 사용하는 것이 한정적 와일드 카드 타입이다.
```java
class Transport<T extends WithWheels> {

    private T some;
}
```
이렇게 되면 T타입은 WithWheels 클래스와 그 하위타입만 허용된다.

## 스프링 IoC
책에서 스프링에 대한 언급이 잠깐 나온다.
스프링에도 의존 객체 주입을 구현한 것이 있는데 이를 스프링 IoC 컨테이너 (BeanFactory, ApplicationContext)라고 한다.

여기서 IoC란 Inversion of Control의 약자로 문자 그대로 뒤집힌 제어권을 의미한다.

여기서 제어권이란 본인이 인스턴스를 생성하거나,
본인의 메서드를 호출,
필요로 하는 의존성을 만들거나 설정하는 등 이런 일련의 행위를 의미한다.
즉, 프로그램의 코드가 자신에게 있지 않고 외부에 있다는 것을 의미한다.

예를 들어 Servlet을 생각해보자.
우리가 어떤 url로 요청을 보내면 서블릿 컨테이너가 이에 맞는 서블릿을 호출해 적절한 메서드(doGet, doPost) 메서드를 호출해준다.<br>
즉 우리는 이런 메서드를 직접 호출하지 않는다. 서블릿 컨테이너가 호출하는 것이다.
이를 제어의 역전이라고 한다.

이런 스프링 IoC 컨테이너를 사용하면 다음과 같은 장점들이 있다.
* 인스턴스를 직접 만들 필요가 없다. -> 스프링이 필요한 빈(Bean)들을 생성하고 주입해준다.
    * 우리는 스프링이 만들어준 객체를 우리가 직접 만든 객체와 구분짓기 위해 빈(Bean)이라 부른다.
* 싱글톤 scope를 쉽게 사용할 수 있다. (스프링 컨테이너 내부에서)
    * 스프링에서 싱글톤 scope 뿐만 아니라 prototype, request, session.. 등 다양한 스코프 타입을 제공해준다.
* 객체(Bean) 생성과 관련한 라이프사이클 메서드를 제공해준다. (init, destroy)

