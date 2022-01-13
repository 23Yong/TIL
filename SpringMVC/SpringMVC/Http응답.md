# HTTP Response

서버에서 응답 데이터를 만드는 방법은 크게 3가지이다.
1. 정적 리소스
    - 스프링 부트는 class path의 다음 디렉토리에 정적 리소스를 제공한다.
        - /static, /public, /resources, /META-INF/resources
2. 뷰 템플릿 사용
    - 뷰 템플릿을 거쳐 HTML이 생성되고, 뷰가 응답을 만들어 전달한다.
    - 스프링 부트는 기본 뷰 템플릿 경로를 제공한다.
        - src/main/resource/templates
3. HTTP 메시지 사용
    - @ResponseBody나 HttpEntity를 사용하면, 뷰 템프릿을 사용하는 것이 아니라 HTTP 메시지 바디에 내용을 적어 응답 데이터를 전달할 수 있다.


## HTTP Response - 메시지 바디에 직접 입력

HTTP API를 이용하는 경우 정적 리소스를 전달하는 것이 아니라 사용자가 필요한 데이터를 전달하는 것이기 때문에 HTTP 메시지 바디에 JSON과 같은 데이터를 넣어 전달한다.

코드로 살펴보자.
```java
@GetMapping("/response")
public void responseBody(HttpServletResponse response) {
    response.getWriter().write("ok");
}
```
ServletResponse를 통해 메시지 바디에 직접 작성하는 방식이다.

```java
@GetMapping("/response")
public ResponseEntity<String> responseBody() {
    return new ResponseEntity<>("ok", HttpStatus.OK);
}
```

```java
@GetMapping("/response")
public ResponseEntity<Member> responseBody() {
    Member member = new Member();
    member.setId(1);

    return new ResponseEntity<>(member, HttpStatus.OK);
}
```
HttpEntity를 통해 메시지 바디에 넣을 내용과 응답코드를 작성하는 방식이다. ResponseEntity는 HttpEntity를 상속받은 것인데, HTTP 메시지의 헤더, 바디 정보를 가지고 있다. ResponseEntity는 여기에 HTTP 응답코드를 설정할 수 있다.

```java
@ResponseBody
@GetMapping("/response")
public String responseBody() {
    return "ok";
}
```

```java
@ResponseBody
@GetMapping("/response")
public Member responseBody() {
    Member member = new Member();
    member.setId(1);

    return member;
}
```

@ResponseBody 애노테이션을 통해 뷰 리졸버가 아닌 HTTP 메시지 컨버터가 동작하게 해 반환 값을 메시지 바디에 넣는 방식이다.

:dart: @RestController
클래스 레벨에서 @Controller 애노테이션 대신 @RestController를 사용하면 해당 컨트롤러에 모두 @ResponseBody가 적용된다.

## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)