# HTTP 요청 파라미터
클라이언트에서 서버로 요청을 보낼 때 주로 다음 3가지 방법을 이용한다.
1. GET - 쿼리 파라미터
    - /url?name=park&age=25
2. POST - HTML form
    - Content-Type : application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달한다.
3. HTML message body - 메시지 바디에 직접 데이터를 담아서 요청

## 요청 파라미터 조회
GET 방식의 쿼리 파라미터 조회든, POST 방식의 HTML form 전송 방식이든 둘다 형식이 같기 때문에 같은 방식으로 데이터를 조회할 수 있는데, 이를 요청 파라미터(request parameter) 조회라고 한다.

### @RequestParam
스프링이 제공하는 ```@RequestParam```을 사용하면 요청 파라미터를 쉽게 조회할 수 있다. 다음을 살펴보자.
```java
@Slf4j
@Controller
public class RequestParamController {

    @ResponseBody
    @RequestMapping("/request-param")
    public String requestParam(
                        @RequestParam("username") String username,
                        @RequestParam("age") int age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
}
```
```
INFO 67716 --- [nio-8080-exec-2] h.s.b.request.RequestParamController     : username=hello, age=20
```
- HTTP 파라미터 이름과 변수의 이름이 같으면 ```@RequestParam```의 (name="xxx") 생략이 가능하다.


### 파라미터 필수 여부
쿼리 파라미터의 필수여부를 지정할 수 있다. 바로 코드로 살펴보자.
```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
                @RequestParam(required = true) String username,
                @RequestParma(required = false) int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

- @RequestParam.required
    - 파라미터의 필수 여부를 나타내며, 기본 값은 true이다.
- /request-param-required?username=
    - 파라미터의 이름만 있고 값이 없는 경우 빈 문자열로 넘어가니 주의하자.
- /request-param-required?username=park
    - 값이 없는 경우 null이 넘어가게 되는데, 이때 기본값(primitive)에 null을 넣는 것은 불가능하니 주의하자.

### 파라미터의 기본 값
파라미터의 값이 없을 때 defaultValue를 통해 기본 값을 지정할 수 있다. 코드로 살펴보자.

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
                @RequestParam(defaultValue = "guest") String username,
                @RequestParam(defaultValue = "-1") int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

- defaultValue는 빈 문자의 경우에도 기본 값을 적용하니 기억해두자.
    - /request-param-default?username=

### 파라미터를 Map으로 조회
파라미터 값들을 Map, MultiValueMap으로 조회할 수 있다. 

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
}
```
- 파라미터를 Map, MultiValueMap으로 조회할 수 있다.
    - ```@RequestParam Map```
    - ```@RequestParam MultiValueMap```
    - 파라미터 값이 1개가 확실하다면 Map을 사용하고 그렇지 않다면, MultiValueMap을 사용하자.

### @ModelAttribute
위와 같은 방식으로 요청 파라미터를 얻었으면, 이를 이용해 필요한 객체를 만들고 값을 넣어주어야 한다. 보통 setter를 이용하지만 스프링에서는 이를 지원해주는 @ModelAttribute를 제공한다.

우선 요청 파라미터를 바인딩 받을 객체를 생성한다. lombok을 이용해 getter, setter를 만든다.
```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```
이제 ```@ModelAttribute```를 사용해보자.
```java
@ResponseBody
@RequestMapping("/model-attribute")
public String modelAttribute(@ModelAttribute HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```
해당 URL로 접근시 HelloData 객체가 생성이되고, 요청 파라미터 값들도 모두 들어가 있다.

이는 스프링 MVC가 ```@ModelAttribute```가 있으면 다음과 같이 실행하기 때문이다.
- HelloData 객체를 생성
- 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해 파라미터의 값을 바인딩한다.
### ::dart:: 프로퍼티
객체가 getUsername(), setUsername()을 가지고 있으면, 이 객체는 username이라는 프로퍼티를 가지고 있다. username의 값을 변경하면 setUsername()이 호출되고 값을 조회하면 getUsername()이 호출된다.


## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)