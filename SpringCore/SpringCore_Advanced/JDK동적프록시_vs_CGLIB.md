# JDK 동적 프록시와 CGLIB

JDK 동적 프록시와 CGLIB 등장 전, 우리는 프록시를 적용 대상마다 만들어 주어야 했다. 하지만 자바에서 제공하는 JDK 동적 프록시나 CGLIB를 사용하면 프록시 객체를 동적으로 만들 수 있다. 즉, 프록시 객체를 적용 대상마다 만들 필요가 없다는 것이다. 단순히 프록시를 적용할 코드를 작성해두고 프록시 객체를 찍어내면 된다.

JDK 동적 프록시와 CGLIB 모두 런타임 위빙 방식이며 프록시 패턴을 바탕으로 동작한다.

## JDK 동적 프록시
JDK에서 제공하는 동적 프록시는 `Interface`를 기반으로 Proxy를 생성해준다. 따라서 인터페이스가 필수이다.
자바에서는 Relection을 활용한 Proxy클래스를 제공해준다. `java.lang.reflect.Proxy` 클래스의 `new ProxyInstance()` 메서드를 사용해 프록시 객체를 만들어 낸다.

이를 이해하기 위해 예제 코드를 통해 살펴보자.
먼저 인터페이스가 필수이기 때문에 인터페이스를 만들어준다.

```java
public interface Animal {
    
    String sound();
}
```
이 인터페이스를 구현하는 타겟 클래스 두 개를 만들어준다.
```java
public class Cat implements Animal {

    @Override
    public String sound() {
        System.out.println("고양이는 냐옹");
        return "냐옹";
    }
}
```
```java
public class Dog implements Animal {

    @Override
    public String sound() {
        System.out.println("강아지는 왈왈");
        return "왈왈";
    }
}
```
그리고 JDK 동적 프록시에 적용할 로직은 `InvocationHandler` 인터페이스를 구현하면 된다.
```java
package java.lang.reflect;
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```
- proxy : 프록시 자신
- method : 호출한 메서드
- args : 메서드를 호출할 때 전달한 인수

이제 구현 코드를 보자.
```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("Time Proxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("resultTime = {}", resultTime);
        return result;
    }
}
```

이제 테스트 코드를 작성해보자.
```java
@Slf4j
public class JdkDynamicProxy {

    @Test
    void create_dynamic_proxy() {
        // 타겟 클래스 생성
        Animal dogTarget = new Dog();
        Animal catTarget = new Cat();

        // 동적 프록시 생성
        Animal dogProxy = (Animal) Proxy.newProxyInstance(Animal.class.getClassLoader(), new Class[] {Animal.class}, new TimeInvocationHandler(dogTarget));
        Animal catProxy = (Animal) Proxy.newProxyInstance(Animal.class.getClassLoader(), new Class[] {Animal.class}, new TimeInvocationHandler(catTarget));

        // 타겟 인스턴스 메서드를 invoke()
        dogProxy.sound();
        catProxy.sound();

        log.info("dogTarget Class = {}", dogTarget.getClass());
        log.info("dogProxy Class = {}", dogProxy.getClass());
    }
}
```
```
Time Proxy 실행
강아지는 왈왈
resultTime = 0
Time Proxy 실행
고양이는 냐옹
resultTime = 0
dogTarget Class = class hello.proxy.jdk.code.Dog
dogProxy Class = class com.sun.proxy.$Proxy9
```
출력 결과를 보면 프록시가 잘 수행되는 것을 확인할 수 있다.
`dogProxy Class = class com.sun.proxy.$Proxy9` 부분이 동적으로 생성된 프록시 클래스 정보이다. 이 프록시가 TimeInvocationHandler 로직을 수행하게 되는 것이다.

### JDK 동적 프록시의 한계
JDK 동적 프록시는 인터페이스가 필수적이다. 인터페이스 없이 클래스만 있는 경우는 어떻게 해야할까? 이것은 JDK 동적 프록시를 통해서는 어렵고 `CGLIB`라는 바이트코드를 조작하는 특별한 라이브러리를 사용해야 한다.

## CGLIB
