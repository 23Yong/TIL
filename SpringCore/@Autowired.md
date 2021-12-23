# @Autowired

@Autowired 어노테이션은 필요한 의존 객체의 '타입'에 해당하는 빈을 IoC 컨테이너에서 찾아서 주입해준다. 

@Autowired에는 required라는 옵션이 있는데, 이는 관계를 주입받을 의존객체가 필수적이지 않은 경우에 관계를 주입받지 못하더라도 빈을 생성할 수 있도록 도와준다. @Autowired(required=false)를 통해 설정이 가능하며 기본 값은 true로 되어있다.

## 사용 위치

@Autowired를 사용할 수 있는 곳은 다음과 같다.
- 생성자
- setter
- 필드

### 생성자 (스프링 4.3부터 생략)
```java
    private BookRepository bookRepository;

    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
```
생성자를 통해 bookRepository에 BookRepository 타입의 빈을 주입해주고 있다. 
### setter

```java
    private BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
```
setter를 통해 bookRepository에 BookRepository 타입의 빈을 주입해주고 있다.

### 필드
```java
    @Autowired
    BookRepository bookRepository;
```
필드에 바로 @Autowired 어노테이션을 붙임으로서 빈을 주입해주고 있다.

## 같은 타입의 빈이 여러 개
위는 해당 타입의 빈이 한 개인 경우 문제없이 사용할 수 있다. 하지만 해당 타입의 빈이 여러 개일 경우는 에러가 발생한다.

이는 빈을 탐색할 때 타입을 통해 빈을 탐색하기 때문인데, 해결방법은 다음과 같다.
- @Primary 사용
    - @Primary를 사용하면 어느 빈을 사용할지 우선적으로 명시해줘 같은 타입의 빈이 여러 개라도 @Primary가 붙어있는 빈을 가져온다.
- 해당 타입의 빈을 모두 수용
    - ```java
        @Autowired
        List<BookRepository> beans;
        ```
        위와 같이 List를 사용하여 해당 타입의 빈을 모두 가져와서 선택하는 방법이 있다.
- @Qualifier 사용
    - 빈의 아이디를 지정하지 않으면 자동적으로 빈으로 등록한 클래스의 smallCase로 빈의 아이디가 설정되는데, @Qualifier("빈의 아이디")를 사용해 어느 빈을 사용할지 명시적으로 나타낼 수 있다.
    ```java
    @Autowired @Qualifier("myBookRepository")
    BookRepository bookRepository;
    ```

## 어떻게 동작하는 것일까
BeanPostProcessor라는 빈 라이프사이클 인터페이스의 구현체를 이용하여 동작한다. 
여기서 BeanPostProcessor는 빈의 initialization이라는 라이프 사이클이 있는데, 그 라이프 사이클 이전 혹은 이후에 할 수 있는 또 다른 라이프 사이클 콜백 인터페이스가 있다. 그게 BeanPostProcessor이다. 

initialization은 @PostConstruct와 같은 어노테이션(빈의 만들어진 다음에 해야 할 일을 정의)을 붙여서 정의할 수 있고, InitializingBean 인터페이스를 구현해서 이 인터페이스가 제공하는 메서드를 override해서 정의 할 수 있다.

그래서 @Autowired는 BeanPostProcessor의 구현 클래스 중에서(제공하는 콜백 중에서) AutowireAnnotationBeanProcessor가 동작을 해서 Autowired라는 어노테이션을 처리한다. 이에 따라서 @Autowired가 붙은 필드에 관계를 주입해주는 것이다.(빈이 생성되기 전)

그러면 BeanPostProcessor의 구현체가 어떻게 동작할까.
빈 팩토리가 BeanPostProcessor의 타입을 찾아서 빈들에게 이 BeanPostProcessor를 적용해준다. (BeanPostProcessor의 로직을 빈에 적용하는 것)


## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)
