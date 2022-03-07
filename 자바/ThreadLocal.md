# ThreadLocal

쓰레드로컬은 해당 쓰레드만 접근 가능한 저장소를 말한다. 쓰레드로컬은 우리에게 각 쓰레드 별로 데이터를 저장하도록 해준다. 이는 동시성 문제를 해결해주는데 도움이 된다.

## Map에 데이터 저장
다음과 같이 사용자의 이름을 저장하는 Context가 있다고 하자.
```java
public class Context {

    private String username;

    public Context(String username) {
        this.username = username;
    }
}
```

여기서 사용자마다 하나의 쓰레드를 가지고 있다고 했을 떄 `Runnable` 인터페이스를 구현하고 있는 `SharedMapWithUserContext` 클래스를 생성해보자.
```java
public class SharedMapWithUserContext implements Runnable {
 
    public static Map<Integer, Context> userContextPerUserId
        = new ConcurrentHashMap<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContextPerUserId.put(userId, new Context(userName));
    }

    // standard constructor
}
```
각 쓰레드별로 사용자의 이름을 얻을 수 있게 되었다. 하지만 맵을 사용하지 않고 일반 변수 필드를 사용했다면 userA가 값을 저장한 후 userB가 값을 저장했다면 userA가 작성했던 값이 사라지기 때문에 문제가 발생한다. 이제 이를 ThreadLocal을 사용해 변경해보자.

## ThreadLocal을 사용해 데이터 저장
각 쓰레드는 `ThreadLocal` 인스턴스를 가지고 있다. 코드로 살펴보자.
```java
public class ThreadLocalWithUserContext implements Runnable {
 
    private static ThreadLocal<Context> userContext 
      = new ThreadLocal<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContext.set(new Context(userName));
        System.out.println("thread context for given userId: " 
          + userId + " is: " + userContext.get());
    }
    
    // standard constructor
}
```
- set() 메서드를 이용해 ThreadLocal에 값을 저장할 수 있다.
- get() 메서드를 이용해 ThreadLocal에 있는 값을 조회할 수 있다.

## ThreadLocal과 ThreadPool
ThreadLocal은 각 쓰레드마다 값을 저장할 수 있는 쉬운 방법을 제공한다. 하지만 WAS(톰캣)처럼 ThreadPool과 함께 사용할 때는 주의해야 한다.

다음과 같은 시나리오를 생각해보자.
1. 애플리케이션이 쓰레드 풀에서 thread-A를 가지고 왔다.
2. thread-A에 ThreadLocal을 이용해 값을 저장했다.
3. thread-A의 작업이 끝나고 애플리케이션은 thread-A를 풀에 반환한다.
4. 후에 애플리케이션이 다른 요청을 처리하기 위해 쓰레드 풀에서 쓰레드를 빌려온다.
5. thread-A를 빌려오게 되었고, thread-A의 ThreadLocal의 값을 지워주지 않았기 때문에 값이 재사용되는 문제가 발생한다.

이를 해결하기 위해서는 반드시 작업이 끝난 후 ThreadLocal의 값을 `remove()`메서드를 통해 지워주어야 한다.

### ThreadPoolExecutor
`ThreadPoolExecutor`를 통해 사용자 커스텀 훅인 beforeExecute()와 afterExecute()를 사용할 수 있다.
쓰레드 풀은 `beforeExecute()`메서드를 쓰레드 풀에서 빌려간 쓰레드가 실행되기 전 호출해준다. `afterExecute()`메서드는 해당 쓰레드의 애플리케이션의 작업이 끝난 후 실행된다.


## 참고 자료

[밸덩 - ThreadLocal](https://www.baeldung.com/java-threadlocal)

[스프링 핵심 원리 고급편 - 김영한](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)