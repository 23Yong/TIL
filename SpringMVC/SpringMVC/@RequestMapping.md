# @RequestMapping
디스패처서블릿이 특별한 빈들에 요청처리와 적절한 응답을 렌더링을 위임한다. 여기서 특별한 빈은 스프링이 관리하고 있는 객체 인스턴스를 말한다. 디스패처서블릿에 의해 찾아지는 빈들 중 HandlerMapping타입의 빈이 있는데, 이 구현체 중 하나가 RequestMappingHandlerMapping이다.

핸들러 매핑의 전체 흐름은 RequestMappingHandlerMapping 빈이 생성되고 초기화하면서 initHandlerMethods를 호출한다. 빈 팩토리에 등록되어 있는 빈들 중 @Controller 또는 @RequestMapping을 달고 있는 빈을 가지고 온다. 이후 핸들러가 될 수 있는 모든 메서드를 추출하고 추출된 registry에 등록하는 것이다.

## HTTP 메서드에 따른 @RequestMapping
@RequestMapping 애노테이션을 사용하여 URL요청을 컨트롤러 메서드에 매핑할 수 있다. URL, HTTP 메서드, 요청 매개변수, 헤더 및 미디어 유형별로 일치시킬 수 있는 다양한 속성을 가지고 있다. 

HTTP 메서드에 따라 다양한 애노테이션을 지원한다.
- @GetMapping
- @PostMapping
- @PutMapping
- @PatchMapping
- @DeleteMapping

원래는 @RequestMapping(value = "/mapping", method = RequestMethod.GET) 이런식으로 사용해야 했지만, 이것을 메서드별로 편리하게 사용할 수 있는 것이다.

@GetMapping을 열어보명 다음과 같이 되어있다.
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = RequestMethod.GET)
public @interface GetMapping {
    // ...
}
```

다음과 같이 사용할 수 있다.
```java

@RestController
@RequestMapping("/mapping/users")
class UserController {

    @GetMapping("/{userId}")
    public User getUser(@PathVariable("userId") Long id) {
        // ...
    }

    @PostMapping
    public void addUsr() {
        // ...
    }
}
```

@RequestMapping애노테이션이 클래스단위, 메서드단위로 적용된 것을 확인할 수 있다.

클래스레벨에서 "/mapping/users"를 사용했다면, 메서드레벨에서 쓴 URL과 합쳐져서 동작을 한다.
- 클래스레벨 : @RequestMapping("/spring/users")
    - 메서드레벨 : @RequestMapping("/save") -> "mapping/users/save"

## @PathVariable 사용
HTTP API는 리소스 경로에 식별자를 넣는 것을 선호하는데, ```@RequestMapping```은 URL경로를 템플릿화 할수 있다. ```@PathVariable```을 사용하여 매칭 되는 부분을 편리하게 조회할 수 있다.(@PathVariable의 이름과 파라미터 이름이 같으면 생략 가능)

다중으로 @PathVariable을 사용할 수도 있다.
```java
@GetMapping("mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
    // ...
}
```


## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)

[스프링 MVC 레퍼런스](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-requestmapping)