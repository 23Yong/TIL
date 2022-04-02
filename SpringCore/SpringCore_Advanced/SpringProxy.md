# 스프링이 지원하는 프록시

[이전](./JDK%EB%8F%99%EC%A0%81%ED%94%84%EB%A1%9D%EC%8B%9C_vs_CGLIB.md)에 JDK 동적 프록시와 CGLIB의 차이를 알아보았다. 만약 두 기술을 함께 이용할 때 JDK가 제공하는 `InvocationHandler`와 CGLIB가 제공하는 `MethodInterceptor`를 중복으로 만들어 관리해야 할까? 

## ProxyFactory

스프링은 동적 프록시를 통합해 관리할 수 있게 해주도록 `ProxyFactory`라는 기능을 제공해준다. 우리는 이제 편리하게 이 프록시 팩토리 하나로 프록시를 생성할 수 있다. 프록시 팩토리는 인터페이스가 있다면 JDK 동적 프록시를 이용하고 구체 클래스만 있다면 CGLIB를 사용한다. 물론 CGLIB만 사용하도록 설정할 수 있다.

![](/img/SpringProxy.png)

다음은 CGLIB만 사용하도록 설정하는 옵션이다.
```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```
ProxyFactory에 `setProxyTargetClass`옵션을 true로 설정해도 된다.
```java
ProxyFactory proxyFactory = new ProxyFactory(target);
proxyFactory.setProxyTargetClass(true); // 강제로 CGLIB를 사용하도록
```

참고로 스프링 부트는 AOP를 만들 때 기본적으로 proxyTargetClass=true로 설정해 사용한다. 
SpringBoot에서는 CGLIB가 안정화되었다고 판단했기 때문이라고 한다.

## Advice, Pointcut, Advisor

### Adivce
스프링은 JDK 동적 프록시의 `InvocationHandler`와 CGLIB의 `MethodInterceptor`를 신경쓰지 않고 개발할 수 있도록 도와주는 개념인 `Advice`를 도입하였다. `InvocationHandler`나 `MethodInterceptor`가 둘을 개념적으로 추상화한 `Advice` 를 호출하게 된다. 프록시 팩토리를 사용하면 전용 `InvocationHandler`와 `MethodInterceptor`를 사용하게 된다.

Adivce를 만드는 방법은 대표적으로 `org.aopalliance.intercept`의 `MethodInterceptor` 인터페이스를 구현하면 된다.

```java
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {

	@Nullable
	Object invoke(@Nonnull MethodInvocation invocation) throws Throwable;
}
```

- MethodInvocation : 내부에는 메서드 호출방법과 현재 프록시 객체 인스턴스, args, 메서드 정보등이 포함되어 있다. 기존의 인자들이 모두 이 안에 들어갔다고 생각하면 된다.

이제 실제 Advice를 만들어보자.
```java
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        System.out.println("TimeProxy 종료, resultTime = " + resultTime);
        return result;
    }
}
```

- invocation.proceed() : 이 메서드를 호출하게 되면 target 클래스를 호출하고 그 결과를 받는다.

이제 테스트코드를 작성해보자. 먼제 인터페이스가 있는 인스턴스에 대해 프록시를 생성한 것을 확인해보자.
```java
public class ProxyFactoryTest {

    @Test
    void interfaceProxy() {
        Animal target = new Dog();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());

        Animal proxy = (Animal) proxyFactory.getProxy();
        proxy.sound();

        System.out.println(target.getClass());
        System.out.println(proxy.getClass());
    }
}
```
```
TimeProxy 실행
왈왈
TimeProxy 종료, resultTime = 0
class hello.proxy.advice.code.Dog
class com.sun.proxy.$Proxy9
```

코드를 보면 프록시 팩토리의 생성자에 프록시의 호출 대상(target)을 함께 넘겨준다. 프록시 팩토리는 이 인스턴스 정보 기반으로 프록시를 만들어낸다. 만약 이 인스턴스에 인터페이스가 있으면 JDK 동적 프록시 방식으로, 인터페이스가 없다면 CGLIB를 통해 프록시를 생성한다. 위 코드에서는 Animal 이라는 인터페이스를 Dog이 구현하고 있기 떄문에 출력결과에서 JDK 동적 프록시 방식으로 프록시가 생성된 것을 확인할 수 있다. 

이후 프록시 팩토리에 `addAdvice(new TimeAdivce())`메서드를 통해 어드바이스를 추가해준다. 어드바이스는 위에서 설명 했듯이 JDK 동적 프록시의 `InvoationHandler`와 CGLIB의 `MethodInterceptor` 같은 것이라고 생각하면 된다. 이렇듯 **프록시가 제공하는 부가 기능**을 어드바이스라 한다.

마지막으로 `getProxy()`메서드를 통해 프록시 객체를 생성하고 이를 받는다.

이번에는 구체 클래스만 있는 프록시 생성을 확인해보자.

```java
public class ProxyFactoryTest {

    @Test
    void interfaceProxy() {
        Cat target = new Cat();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());

        Cat proxy = (Cat) proxyFactory.getProxy();
        proxy.sound();

        System.out.println(target.getClass());
        System.out.println(proxy.getClass());
    }
}
```
```
TimeProxy 실행
냐옹
TimeProxy 종료, resultTime = 29
class hello.proxy.advice.code.Cat
class hello.proxy.advice.code.Cat$$EnhancerBySpringCGLIB$$168c1be5
```

위의 ~~$$EnhancerBySpringCGLIB\$$~~에 의해 CGLIB를 통해 프록시가 생성된 것을 확인할 수 있다.

### Pointcut
포인트컷은 어디에 부가기능을 적용할지, 어디에 부가기능을 적용하지 않을지 판단하는 필터링 로직이다.

### Advisor
단순하게 하나의 어드바이스와 하나의 포인트컷을 가지고 있는 것이다. 코드로 살펴보자.

```java
@Test
void advisorTest() {
    Animal target = new Dog();
    ProxyFactory proxyFactory = new ProxyFactory(target);

    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new TimeAdvice());

    proxyFactory.addAdvisor(advisor);
    Animal proxy = (Animal) proxyFactory.getProxy();

    proxy.sound();
}
```
```
TimeProxy 실행
왈왈
TimeProxy 종료, resultTime = 0
```
Advisor 인터페이스의 가장 일반적인 구현체인 `DefaultPointcutAdvisor`를 사용했다. 생성자에는 하나의 포인트컷과 어드바이스를 넣어주면 된다. 

`Pointcut.TRUE`는 항상 true를 반환하는 포인트컷이다. 어디든 부가기능을 적용하겠다는 소리이다. 

이번에는 하나의 프록시에 여러 어드바이저를 적용해보자.
먼저 어드바이스 2개를 만든다.
```java
static class Advice1 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("advice1 호출");
        return invocation.proceed();
    }
}

static class Advice2 implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("advice2 호출");
        return invocation.proceed();
    }
}
```
이후 테스트코드를 작성해보자.
```java
@Test
void multiAdvisorTest() {
    DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
    DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());

    // 프록시 생성
    Animal target = new Dog();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    proxyFactory.addAdvisor(advisor2);
    proxyFactory.addAdvisor(advisor1);
    Animal proxy = (Animal) proxyFactory.getProxy();

    // 실행
    proxy.sound();
}
```
```
advice2 호출
advice1 호출
왈왈
```
출력결과를 보면 등록하는 순서대로 어드바이저가 호출되는 것을 확인할 수 있다. 스프링 AOP를 적용할 때 지금처럼 프록시는 하나만 만들고, 하나의 프록시에 여러 어드바이저를 적용하는 것을 기억하자. 다시 말하면, 하나의 target에 여러 AOP가 동시에 적용되어도, 스프링의 AOP는 하나의 프록시만 만든다.

일단은 프록시에 맞추어 용어가 설명되어 있다. 이후 AOP에 맞추어 다시 정리해야겠다..

## 참고 자료

[인프런 - 스프링 핵심원리_고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)