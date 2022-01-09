# 스프링 MVC 구조

![](/img/SpringMVC구조.png)
스프링 MVC의 전체적인 구조는 다음과 같다.

[이전](/SpringMVC/MVC프레임워크의이해/여러컨트롤러적용.md)에 했던 FrontController의 역할을 DispatcherServlet이 수행하고 있는 것을 확인할 수 있다.

전체적인 동작순서는 front controller와 같이 클라이언트로 부터 요청이 들어오면 
- 핸들러 매핑을 통해 요청 URL에 대한 핸들러(컨트롤러)를 조회하고
- 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
- 이후 어댑터를 실행해 실제 핸들러를 호출해 반환 정보를 ModelAndView로 변환해 반환받는다.
- 이후 viewResolver를 통해 View를 반환받고 렌더링하여 최종결과를 출력한다.


## Dispatcher Servlet
dispatcher servlet의 클래스 다이어그램을 살펴보면 다음과 같다.
![](/img/DispatcherServlet.png)
dispatcher servlet 역시 HttpServlet을 상속받아서 사용하는 것을 확인할 수 있다.

요청 흐름은 서블릿이 호출되면 ```HttpServlet```이 제공하는 ```service()``` 메서드가 호출된다.
스프링 MVC는 ```DispatcherServlet```의 부모 클래스인 ```FrameworkServlet```에서 ```service()```메서드를 오버라이딩 해두었다. 결국 ```FrameworkServlet```의 ```service()``` 메서드를 시작으로 여러 메서드가 호출되고, ```DispatcherServlet```의 ```doDispatch()```메서드가 호출된다.


### DispatcherServlet - doDispatch
doDispatch 메서드의 핵심 코드들은 다음과 같다.
```java
mappedHandler = getHandler(processedRequest);

if (mappedHandler == null) {
    noHandlerFound(processedRequest, response);
	return;
}
```
핸들러정보를 request를 통해 찾아내고 매핑할 핸들러가 없으면 찾지 못했다는 메서드를 호출한다.

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```
이후 핸들러를 처리할 수 있는 어댑터를 조회한다.

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
```
찾아낸 핸들러 어댑터를 호출해 ModelAndView를 반환받는다.
이후 ```processDispatchResult```메서드와 ```render```메서드를 통해 뷰를 렌더링한다.

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {
    // 뷰 렌더링 호출
    render(mv, request, response);
}
```
```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    View view;
	String viewName = mv.getViewName();
	if (viewName != null) {
		// We need to resolve the view name.
		view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
		if (view == null) {
			throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
			"' in servlet with name '" + getServletName() + "'");
		}
	}
    /** 생략 **/

    view.render(mv.getModelInternal(), request, response);
}
```
위 두 함수에서 뷰를 렌더링하는 부분만 가져온 것이다.
processDispatchResult에서 render함수를 호출하고, 뷰 리졸버를 통해 뷰를 찾아 View를 반환한다. 이후 view를 렌더링한다.


## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)