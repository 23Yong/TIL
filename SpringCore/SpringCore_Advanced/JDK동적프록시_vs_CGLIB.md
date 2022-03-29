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
CBLIB는 바이트코드를 조작해 동적으로 클래스를 생성하는 기술을 제공해주는 라이브러리로 인터페이스가 아닌 `구체 클래스`를 대상으로 동작이 가능하다. 상속방식을 이용해 Proxy화 할 메서드를 오버라이딩 하는 방식이기 때문에 private 혹은 final 키워드가 들어간 경우 Proxy에서 해당 메서드에 대한 Aspect를 적용할 수 없다.

코드를 통해 이해해보자. 먼저 JDK 동적 프록시와는 달리 인터페이스가 굳이 필요없기 때문에 클래스만 만들어준다.
```java
public class Dog {

    public String sound() {
        System.out.println("강아지는 왈왈");
        return "왈왈";
    }
}

```

JDK 동적 프록시에 `InvocationHandler`가 있듯이 CGLIB에는 `MethodInterceptor`가 있다. 
```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        long startTime = System.currentTimeMillis();
        log.info("Time Proxy 시작");

        Object result = method.invoke(target, objects);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("resultTime = {}", resultTime);
        return result;
    }
}
```
- o : CGLIB가 적용된 객체
- method : 호출된 메서드
- objects : 메서드를 호출하면서 전달된 인수들
- methodProxy : 메서드 호출에 사용

이제 테스트 코드를 작성해보자.
```java
@Slf4j
public class Cglib {

    @Test
    void cglib() {
        Dog dog = new Dog();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Dog.class);
        enhancer.setCallback(new TimeMethodInterceptor(dog));
        Dog proxy = (Dog) enhancer.create();

        proxy.sound();

        log.info("targetClass = {}", dog.getClass());
        log.info("proxyClass = {}", proxy.getClass());
    }
}
```
```
Time Proxy 시작
강아지는 왈왈
resultTime = 10
targetClass = class hello.proxy.cg.code.Dog
proxyClass = class hello.proxy.cg.code.Dog$$EnhancerByCGLIB$$3a8a3b6e
```
출력결과를 통해 프록시가 잘 적용되는 것을 확인할 수 있다.

CGLIB를 통해 프록시를 생성할 때는 두 가지 작업이 필요하다.
1. `net.sf.cglib.proxy.Enhancer` 클래스를 사용해 프록시 객체 생성
2. `net.sf.cglib.proxy.Callback` 클래스를 사용해 프록시 객체 조작

- CGLIB는 Enhancer를 사용해 프록시를 생성하기 떄문에 `new Enhancer()`를 통해 Enhancer 인스턴스를 생성해준다.
- CGLIB는 구체 클래스를 상속받아 프록시를 생성하기 때문에 상위클래스를 `setSuperClass(Dog.class)`를 통해 지정해준다.
- 프록시 객체를 조작하기 위해 `setCallback(new TimeMethodInterceptor)`를 통해 프록시에 적용할 로직을 할당해준다.
- 이후 `create()`를 통해 프록시 객체를 생성한다. 앞에서 `setSuperClass()`를 통해 설정한 클래스를 상위 클래스로 상속 받아 만들어진다.

출력결과에서도 알 수 있듯이 CGLIB를 통해 프록시가 만들어진 것을 확인할 수 있다.

### CGLIB의 제약
클래스 기반 프록시는 상속을 이용하기 때문에 몇 가지 제약이 존재한다.
- CGLIB는 부모 클래스로부터 자식 클래스를 동적으로 생성하기 때문에 부모 클래스의 기본 생성자가 필요하다.
- 클래스에 `final`이 붙으면 상속이 불가능하기 때문에 CGLIB에서는 예외가 발생한다.
- 마찬가지로 메서드에 `final`이 붙으면 해당 메서드를 오버라이딩 할 수 없기 때문에 프록시 로직이 동작하지 않는다.

## 참고 자료
[인프런 - 스프링 핵심원리_고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)