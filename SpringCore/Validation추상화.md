# Validation 추상화
애플리케이션에서 사용하는 객체들을 검증할 때 사용하는 인터페이스, Validator를 제공한다. 주로 스프링 웹MVC에서 사용하지만, 웹 계층 전용은 아니다.
BeanValidation이 제공하는 여러 validation용 어노테이션을 사용해서 객체의 데이터를 검증할 수 있다.

## BeanValidation
사용자 입력의 유효성을 검증하는 것은 대부분의 애플리케이션에서 일반적인 요구사항이다. 여기서 Java Bean Validation은 이러한 종류의 논리를 처리하기 위한 사실상의 표준이 되었다.
- JSR 380
    - JSR 380은 JakartaEE 및 JavaSE의 일부인 bean validation을 위한 Java API의 spec이다. 
    - JSR은 어노테이션을 통해(ex. @Min, @Max, @NotNull, @Size, @Email, @NotEmpty...) 빈의 속성이 특정 조건을 충족하는지 확인한다.

## Validator
스프링이 제공하는 인터페이스(org.springframework.validation.Validator)에는 두 가지 메서드를 구현해야한다.
- supports
    - 인자로 넘어온 검증해야하는 인스턴스의 클래스타입의 변수가 Validator가 검증할 수 있는(지원하는) 클래스인지 확인하는 메서드
- validate
    - 실질적으로 검증작업이 일어나는 곳

코드를 보며 살펴보자.
Evnet라는 클래스가 있고, 타이틀이 비어있는 값이 아니라고 가정하자.
```java
public class Event {

    Integer id;

    String title;

    public Integer getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```
이 이벤트를 검증하는 EventValidator 클래스를 만들면
```java
public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Event.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "not empty", "Empty title is now allowed");
    }
}
```
- ```supports``` : Event 타입의 클래스를 검증한다.
- ```validate``` : ValidationUtils를 사용해서 rejectIfEmptyOrWhitespace 메서드를 이용한다. 만약 event의 title이 비어있다면 errorCode 로 not empty, errorCode를 찾지 못했을 때 defaultMessage로 'Empty title is not allowed' 메시지를 적어준다.

이제 이를 테스트해보자.
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Event event = new Event();
        EventValidator eventValidator = new EventValidator();
        event.setId(1);
        // 어떤 이벤트를 검사할 것인지, 오브젝트의 이름
        Errors errors = new BeanPropertyBindingResult(event, "event");

        // errors에 검증 에러를 담아준다.
        eventValidator.validate(event, errors);

        System.out.println(errors.hasErrors());
        errors.getAllErrors().forEach(e -> {
            System.out.println("======== error code =========");
            // 에러코드 출력
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```
Event 객체를 생성하고 EventValidator를 통해 검증을하는 코드이다. Event의 title에 값을 주지 않았기 때문에 에러가 발생할텐데, 출력결과를 살펴보면
```
true
======== error code =========
not empty.event.title
not empty.title
not empty.java.lang.String
not empty
Empty title is now allowed
```
지금 우리가 만든 에러코드인 'not empty' 외에 3가지를 추가해준 것을 확인할 수 있다.

하지만 이 방법은 오래된 방법이다.
스프링부트 2.0.5 이상 버전을 사용하면 LocalValidatorFactoryBean을 빈으로 자동으로 등록을 해준다. 따라서 우리는 이 빈을 이용해서 validation annotation을 사용해 검증할 수 있다.

## Validation annotation 사용
Event 클래스를 다시 작성해보면
```java
public class Event {

    Integer id;

    @NotEmpty
    String title;

    @Min(0)
    Integer limit;

    public Integer getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public Integer getLimit() {
        return limit;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public void setLimit(Integer limit) {
        this.limit = limit;
    }
}
```
title 에 @NotEmtpy, limit에 @Min 어노테이션을 달아 유효성 검증을 적용한다.

그리고 이를 테스트 하기위한 코드를 작성해보면
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(validator.getClass());

        Event event = new Event();
        event.setLimit(-1);
        // 어떤 이벤트를 검사할 것인지, 오브젝트의 이름
        Errors errors = new BeanPropertyBindingResult(event, "event");

        // errors에 검증 에러를 담아준다.
        validator.validate(event, errors);

        System.out.println(errors.hasErrors());
        errors.getAllErrors().forEach(e -> {
            System.out.println("======== error code =========");
            // 에러코드 출력
            Arrays.stream(e.getCodes()).forEach(System.out::println);
            System.out.println(e.getDefaultMessage());
        });
    }
}
```
validator를 @Autowired를 통해 주입받고 에러를 발생시키도록 limit에 -1을 설정하면 다음과 같은 출력이 나온다.
```
class org.springframework.validation.beanvalidation.LocalValidatorFactoryBean
true
======== error code =========
Min.event.limit
Min.limit
Min.java.lang.Integer
Min
0 이상이어야 합니다
======== error code =========
NotEmpty.event.title
NotEmpty.title
NotEmpty.java.lang.String
NotEmpty
비어 있을 수 없습니다
```

springboot 2.3.0 부터 starter-validation을 따로 설정해주어야 정상적으로 동작하니 참고해두자.


## 참고자료
[스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core/dashboard)

[밸덩 BeanValidation](https://www.baeldung.com/javax-validation)