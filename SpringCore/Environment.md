# Environment
ApplicationContext가 가지고 있는 기능 중 하나이다.
어떤 환경이라고 생각하면 되는데, 우리가 테스트 환경에서는 어떤 빈들이 필요할 수 있고, 실제 제품 환경에서는 다른 빈들이 필요할 수 있다는 것이다.

## 프로파일
프로파일은 빈들의 묶음인데, 위 각각의 환경들에서 어떤 빈들을 써야하는 경우나 특정환경에서만 어떤 빈들을 등록해야 하는 경우 프로파일이라는 기능을 사용할 수 있다.

이 기능은 Environment라는 인터페이스를 통해 사용할 수 있다.

우리가 따로 설정하지 않은 빈들은 기본 프로파일이라고 생각하면 된다. (우리가 이 빈들이 무슨 프로파일에 들어가는 빈이라고 설정해주지 않았기 때문에)

### 사용방법
BookRepository 인터페이스와 이를 구현하는 TestBookRepository 클래스가 있다고 생각하자.
```java
public interface BookRepository {

}
```

```java
public class TestBookRepository implements BookRepository {

}
```

그리고 프로파일의 이름을 'test'라고 설정하고 test프로파일에서만 사용할 빈 실정파일을 만든다.
```java
@Configuration
@Profile("test")
public class TestConfiguration {

    @Bean
    public BookRepository bookRepository {
        return new TestBookRepository();
    }
}
```
이 빈 설정파일은 test 프로파일일 때만 사용되는 빈 설정파일이 된다. 우리는 test프로파일로 이 애플리케이션을 실행하기 전까지는 이 빈설정파일이 적용이 되지 않는다.

### 설정방법
이제 이 프로파일을 적용한 빈을 사용해보자
```java
@Autowired
ApplicationContext context;

@Autowired
BookRepository bookRepository;

@Override
public void run(ApplicationArguments args) throws Exception {
        Environment env = context.getEnvironment();
        System.out.println("profiles: " + Arrays.toString(env.getActiveProfiles()));
        System.out.println("defalult: " + Arrays.toString(env.getDefaultProfiles()));
    }
```
이를 실행시켜보면 에러가 발생한다. 왜냐하면 프로파일을 설정해주지 않았기 때문이다.

프로파일을 적용하기 위해서는 IntelliJ의 Active profiles에 프로파일 이름을 주면된다.
혹은 VM options에 -Dspring.profiles.active="test,A,B..." 이런식으로 프로파일의 이름들을 주면된다.

이제 실행시켜보자.
![실행결과](../img/profile_test.png)

:dart:
```java
@Repository
@Profile("test")
public class TestBookRepository implements BookRepository { }
```
이렇게 컴포넌트 스캔으로 등록되는 빈에도 프로파일을 지정할 수 있다.

## 프로퍼티
Environment가 제공하는 다른 기능이다. 이 기능은 애플리케이션에 등록되어있는 여러 key-value쌍으로 제공되는 프로퍼티에 접근할 수 있는 기능이다. 계층형으로 접근을한다. 여기서 말하는 계층형이란 우선순위를 말한다.

프로퍼티에는 우선순위가 있는데 다음과 같다.
- StandardServletEnvironment의 우선순위
    - ServletConfig 매개변수
    - ServletContext 매개변수
    - JNDI (java:comp/env/)
    - JVM 시스템 프로퍼티 (-Dkey="value")
    - JVM 시스템 환경변수(운영체제 환경 변수)

### 접근 방법
그러면 프로퍼티에 어떻게 접근할까
먼저 VM option에 다음과 같이 설정해두자.
- -Dapp.name=Spring5
그 후 다음 메서드를 호출하면 Spring5가 출력이된다.
```java
System.out.println(environment.getProperty("app.name"));
```

이런 방법을 사용하지 않고 체계적으로 프로퍼터를 관리하고 싶을 때는 ```.properties```파일을 사용할 수 있다.

resources 디렉토리에 app.properties를 만들수 있다.
app.properties에 다음과 같이 ```app.about=spring```
이라고 작성한 후 Configuration이 들어있는 파일에서 ```@PropertySource``` 어노테이션으로 ```@PropertySource("classpath:/app.properties")```를 붙인다.

```java
@SpringBootApplication
@PropertySource("classpath:/app.properties")
public class Demospring51Application {
    public static void main(String[] args) {
        SpringApplication.run(Demospring51Application.class, args);
    }
}
```

이렇게 되면 프로퍼티소스를 통해서 Environment에 들어가면 우리는 Environment에서 꺼내 쓸 수 있다.
```java
@Override
    public void run(ApplicationArguments args) throws Exception {
        Environment env = context.getEnvironment();
        System.out.println(env.getProperty("app.name"));
        System.out.println(env.getProperty("app.about"));
    }
```

## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)
