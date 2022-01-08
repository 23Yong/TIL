# View 분리

![](/img/V1.png)
위 그림에서 모든 컨트롤러에서 뷰로 이동하는 것은 코드에 중복이 존재한다. 
```java
String viewPath = "WEB-INF/views/something.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
해당 코드들이 각 컨트롤러에 중복되는 문제가 발생한다. (JSP를 사용하는 경우)

![](/img/V2.png)
이 중복되는 부분을 분리하기 위해 View를 처리하는 객체를 두어 Controller는 View를 반환하고 FrontController에서 이를 받아 View를 처리하는 함수를 호출하고 forward하는 로직을 수행한다.


## View 분리 도입
먼저 View를 처리하는 클래스를 만든다.
```java
public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);

        dispatcher.forward(request, response);
    }
}
```
그리고 컨트롤러들의 인터페이스를 만든다. 기존 [컨트롤러](./front-controller_pattern.md)와는 다르게 MyView를 반환하도록 한다.
```java
public interface MyController {
    MyView proces(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
마지막으로 프런트 컨트롤러를 만든다. 
```java
@WebServlet(name = "frontControllerServlet", urlPatterns = "/front-controller/*")
public class FrontControllerServlet extends HttpServlet {

    private Map<String, MyController> controllerMap = new HashMap<>();

    public FrontControllerServlet() {
        // controllerMap에 url, 각 controller매핑
    }

    @Override
    public void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        String requestURI = request.getRequestURI();

        MyController controller = controllerMap.get(requestURI);
        if(controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```
MyController의 반환타입이 MyView이기 떄문에, 프런트 컨트롤러는 MyView를 반환받고, 이를 이용해 render() 메서드를 호출해 forward로직을 수행해 JSP가 실행된다.

## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)