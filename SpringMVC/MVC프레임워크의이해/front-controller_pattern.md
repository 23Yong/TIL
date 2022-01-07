# front-controller pattern
기존 클라이언트-서버 구조는 각 클라이언트가 요청을 보내면 서버는 그에 맞는 컨트롤러로 요청을 처리하는 구조였다.
![](/img/프런트컨트롤러_전.png)

하지만 이는 중복되는 코드가 발생하는 문제가 있고, 서버 입장에서 현재 웹 애플리케이션 실행에 있어서 작업을 일괄처리하기가 어렵다는 문제가 있다.

이를 해결하기 위한 것이 프런트 컨트롤러이다.
![](/img/프런트컨트롤러.png)

각 클라이언트로부터 오는 모든 요청을 한 곳에서 받고, 이 요청을 처리할 수 있는 적절한 컨트롤러에서 요청을 처리한다.

## front-controller 패턴 특징
- 프런트 컨트롤러 서블릿 하나로 클라이언트로 요청을 받는다.
- 프런트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다.
- 입구가 하나, 공통 처리 가능
- 프런트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.

## front-controller 도입
```java
public interface MyController {
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
컨트롤러 인터페이스를 만들고 각 컨트롤러들은 해당 인터페이스를 구현하도록 한다. 프런트 컨트롤러는 이 인터페이스를 호출해 구현에 관계없이 각 컨트롤러들을 호출할 수 있다.

이제 프런트 컨트롤러를 만들어보자.
```java
@WebServlet(name = "frontControllerServlet", urlPatterns = "/front-controller")
public class FrontControllerServlet extends HttpServlet {

    private Map<String, MyController> controllerMap = new HashMap<>();

    public FrontControllerServlet() {
        // 각 컨트롤러에 맞는 url 매핑
    }

    @Override
    public void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        MyController controller = controllerMap.get(requestURI);
        if(controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(request, response);
    }
}
```
/front-controller로 오는 모든 url들을 이 서블릿이 받고, service() 메서드에서 ```requestURI```를 조회해 실제 호출할 컨트롤러를 찾아 실행하는 구조로 되어있다.

## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)