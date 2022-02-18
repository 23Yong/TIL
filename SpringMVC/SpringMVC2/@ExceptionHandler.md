# @ExceptionHandler

@ExceptionHandler는 @Controller, @RestController가 적용된 빈에서 발생하는 예외를 잡아 처리해주는 기능이다.

@ExceptionHandler로 등록한 메서드는 ExceptionHandlerExceptionResolver에 의해 동작한다. 스프링은 ExceptionHandlerExceptionResolver를 기본으로 제공하고 기본으로 제공하는 ExceptionResolver 중 우선순위가 가장 높다.

스프링이 기본으로 제공하는 ExceptionResolver들의 구현체들은 다음과 같다.
| HandlerExceptionResolver | Description |
|------------------------- | ----------- |
| SimpleMappingExceptionResolver | 예외 클래스들과 error 뷰를 매핑합니다. 브라우저에서 에러 페이지 뷰를 렌더링할 때 유용하다.|
| DefaultHandlerExceptionResolver | 스프링의 내부 예외를 처리한다. 예를 들어 파라미터 바인딩 시점에 타입이 맞지 않으면 TypeMismatchException이 발생하는데 이는 결과적으로 WAS, 서블릿 컨테이너까지 예외가 던져지고 500오류가 발생한다. 그런데 HTTP에서는 이런 경우는 보통 400오류를 사용하도록 되어 있다. 이때 DefaultHandlerExceptionResolver가 400 오류로 변경해준다.|
| ResponseStatusExceptionResolver | @ResponseStatus 애노테이션이 젹용되어있는 메서드들을 resolve하고 애노테이션에 있는 값으로 HTTP 코드를 변경해준다. |
| ExceptionHandlerExceptionResolver | @Controller 혹은 @ControllerAdvice 안에 @ExceptionHandler가 적용되어 있는 메서드를 해당 예외가 발생되었을 떄 호출하여 예외를 처리한다. |

코드를 살펴보자.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {

	Class<? extends Throwable>[] value() default {};
}
```

value를 통해 어떤 예외를 처리할 지 지정할 수 있다. 이때 value를 지정하지 않으면 모든 예외를 잡기 때문에 주의하자.

### @ExceptionHandler 사용

사용법을 코드로 살펴보자. IllegalArgumentException을 처리하는 코드이다.
```java
@Controller
public class ExHandleController {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<String> illegalExHandle(IllegalArgumentException e) {
        // ...
    }
}
```

@ExceptionHandler 애노테이션을 선언하고 처리하고 싶은 예외를 지정해주면 된다. 해당 컨트롤러에서 해당 예외가 발생하면 위의 메서드가 호출되며 예외를 처리해준다.

또한 다양한 예외를 한 번에 처리할 수도 있다.
```java
@ExceptionHandler({AException.class, BException.class})
public String ex(Exception e) {
    // ...
}
```
@ExceptionHandler의 값에 예외를 생략할 수도 있는데, 생략시 메서드의 파라미터 예외가 지정된다.

### @ExceptionHandler의 실행 흐름
@ExceptionHandler의 실행흐름은 다음과 같다.
- 컨트롤러르 호출했더니 AException이 발생해 해당 예외가 컨트롤러 밖으로 던져진다.
- 예외가 발생했기 때문에 ExceptionResolver가 동작한다. 이중 우선순위가 가장 높은 ExceptionHandlerExceptionResolver가 동작한다.
- ExceptionHandlerExceptionResolver는 AException을 처리할 수 있는 @ExceptionHandler가 있는지 탐색한다.
- 해당 핸들러가 있으면 호출해 예외를 적절히 처리한다. 

## @ControllerAdvice
@ExceptionHandler는 등록된 Controller에서만 동작을 하고 중복된 예외에 대해서 각 컨트롤러에 @ExceptionHandler를 일일이 두는 것은 번거롭다. 이를 해결하기 위해 나온 것이 바로 @ControllerAdvice이다.

@ControllerAdvice는 @Controller가 붙어있는 모든 클래스에 대해 예외를 잡을 수 있도록 도와준다. 그러면 @ControllerAdvice에 예외를 처리하는 @ExceptionHandler를 두면 모든 컨트롤러에서 발생하는 예외를 처리할 수 있다.

@RestControllerAdvice 라는 것도 있는데 이는 @ControllerAdvice에 @ResponseBody를 더한 것과 같다. @Controller, @RestController를 생각하면 될 것 같다.

## 참고 자료
[김영한- MVC2](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)

[@ExceptionHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)