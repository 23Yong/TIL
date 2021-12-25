# ApplicationEventPublisher
이벤트 프로그래밍을 할 때 필요한 인터페이스를 제공해주며 옵저버 패턴의 구현체이다.

## 이벤트정의
스프링 4.2 이전에는 ApplicationEvent를 상속받아 구현했었다. 하지만 지금은 그렇지 않다.

이 이벤트는 빈으로 등록되지 않는다는 것도 기억해두자.
```java
public class MyEvent {
    private int data

    private Object source;

    public MyEvent(Object source, int data) {
        this.source = source;
        this.data = data;
    }

    public Object getSource() {
        return source;
    }

    public int getData() {
        return data;
    }
}
```
이제 이 이벤트를 publish 해야하는데, 이 기능을 applicationContext가 가지고 있다.

```java
@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationEventPublisher publishEvent;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        publishEvent.publishEvent(new MyEvent(this, 100));
    }
}
```

이렇게 publish한 이벤트를 처리를 하는 클래스르 하나 만들어야 한다. 이 이벤트 핸들러는 빈으로 등록이되어야 한다.
```java
@Component
public class MyEventHandler {

    @EventListener
    public void handle(MyEvent event) {
        System.out.println("이벤트 받았다. 이벤트는 : " + event.getData());
    }
}
```

이제 코드를 실행을 하면 SpringBootApplication이 구동이 되고, AppRunner가 실행이 되면서 이벤트를 발생시킨다. 이때 발생시킨 이벤트를 등록되어있는 빈들 중 MyEventHandler가 처리를 해주면서 "이벤트 받았다. 이벤트는 100" 이라고 출력을 해주는 것이다.

### 다른 이벤트 처리 클래스
이번엔 다른 이벤트 처리 클래스를 만들어보자. 
```java
@Component
public class AnotherHandler {
    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("Another " + myEvent.getData());
    }
}
```
동일하게 MyEvent 이벤트를 처리하도록 하고 어플리케이션을 실행하면 순차적으로 실행이 된다.(여기서 순차적이란 뭐가 먼저 실행될지는 모르지만 한 이벤트 처리가 끝나고 다음 이벤트 처리가 수행된다는 뜻이다. 동시에 다른 스레드에서 처리가 된다는 것이 아니다.)

### 이벤트 처리기에 순서 정하기
만약 이벤트에 순서가 중요한 경우 순서를 정하고 싶다면 ```@Order``` 어노테이션을 사용하면 된다. 

```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class AnotherHandler {
    @EventListener
    public void handle(MyEvent myEvent) {
        System.out.println("Another " + myEvent.getData());
    }
}
```
이렇게 설정하고 실행하면 Another이 먼저 출력되는 것을 확인할 수 있다.

### 비동기적 실행
이벤트 핸들링을 비동기적으로 실행하고 싶다면 ```@Async``` 어노테이션을 사용하면 된다. ```@Async```를 같이 쓰면 당연히 순서는 보장이 되지 않는다.(다른 스레드에서 실행되고, 스레드가 실행되는 순서는 스레드 스케줄링에 따라 달라지기 때문에..)

```java
@Component
public class AnotherHandler {
    @EventListener
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("Another " + myEvent.getData());
    }
}
```
이렇게 이벤트 핸들러 함수에 ```@Async``` 어노테이션을 달아두고 Configuration이 있는 클래스에 ```@EnableAsync``` 어노테이션을 달아준다.
```java
@SpringBootApplication
@EnableAsync
public class Demospring51Application {
    public static void main(String[] args) {
        SpringApplication.run(Demospring51Application.class, args);
    }
}
```

이후 실행시켜 스레드를 출력해보면 각기 다른 스레드에서 돌아가는 것을 확인할 수 있다.


## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)