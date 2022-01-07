# Model 추가
컨트롤러의 입장에서 보면 사용하지도 않는 HttpServletRequest, HttpServletResponse를 인자로 받고 있다. 요청 파라미터를 자바의 Map 형태로 넘기도록 하면 지금 구조에서 각 컨트롤러가 서블릿을 몰라도 동작할 수 있다.
그리고 request 객체 대신 Map객체를 만들어 반환하도록 한다. 

## ModelView
서블릿에 종속적인 컨트롤러가 아닌 서블릿의 종속성을 제거한 컨트롤러를 만들기 위해 Model을 직접만들고, View이름 까지 전달하는 객체를 만들어보자.

ModelView 클래스를 만든다. lombok을 사용해 getter, setter를 만들었다.
```java
@Getter @Setter
public class ModelView {

    private String viewName;

    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }
}
```
컨트롤러에서 뷰의 이름과 뷰를 렌더링할 때 필요한 model 객체에 key, value형태로 넣어주도록 한다.

```java
public interface MyController {
    public ModelView process(Map<String, String> paramMap);
}
```
이 컨트롤러는 서블릿 기술을 전혀 사용하지 않아 코드가 단순해지고 테스트 코드 작성이 용이해진다.

HttpServletRequest가 제공하는 파라미터는 프런트 컨트롤러가 paramMap에 담아 호출해주면 된다. 응답 결과로 Model 데이터를 포함하는 ModelView객체를 반환한다.

이를 구현하는 컨트롤러 하나를 보자.
```java
public class BookSaveController implements MyController {

    @Override
    public void process(Map<String, String> paramMap) {
        String name = paramMap.get("name");
        String description = paramMap.get("description");

        Book book = new Book(name, description);

        ModelView mv = new ModleView("save-result");
        mv.getModel().put("book", book);
        return mv;
    }
}
```
파라미터 정보가 Map에 담겨있기 때문에 이를 필요한 요청 파라미터에서 조회해 사용하고, 모델은 단순한 map이기 떄문에 model에 뷰에서 필요한 Book이란 객체를 담아 반환하도록 한다.

이제 front-controller를 보자.
```java
@WebServlet(name = "frontControllerServlet", urlPatterns = "/front-controller/*")
public class FrontControllerServlet extends HttpServlet {
    private Map<String, MyController> controllerMap = new HashMap<>();

    public FrontControllerServlet() {
        // controllerMap에 url과 각 컨트롤러 매핑
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        MyController controller = controllerMap.get(requestURI);
        if(controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);
    }

    private MyView viewResolver(String viewName) {
        MyView view = new MyView("/WEB-INF/views/" + viewName + ".jsp");
        return view;
    }
    
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```
createParamMap() 메서드를 통해 URL을 통해 전달된 파라미터들을 뽑아 Map으로 변환하도록 하고 이를 컨트롤러에 전달하며 process메서드를 호출한다. 이를 통해 반환된 ModelView 객체를 얻는다.

이 ModelView를 통해 논리 뷰를 얻고 이를 viewResolver를 통해 실제 물리 주소로 변환한다. 뷰 객체를 통해 화면을 렌더링하며 JSP로 forward한다.

MyView의 render
```java
public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
}

private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
    model.forEach((key, value) -> request.setAttribute(key, value));
}
```
JSP는 request.getAttribute로 데이터를 조회하기 떄문에 setAttribute로 모델의 데이터를 담아둔다.

## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)