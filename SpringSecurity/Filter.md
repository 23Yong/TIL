# Filter

스프링 시큐리티를 시작하기 전에, 필터에 대해 먼저 알아야 한다. <br>
스프링 시큐리티의 서블릿 지원은 `Servlet Filter`에 기본을 두고 있기 떄문에 Filter의 역할에 대해 먼저 아는 것이 중요하다.<br> 
우선은 Spring Security의 `DelegatingFilterProxy`에 가기 전 과정을 학습해보자.

어떤 애플리케이션의 공통 관심사항을 처리할 때, 특히 웹 개발에 있어서 Filter나 Interceptor를 사용하는 것은 좋은 방법이라고 생각한다.<br> 특히 Filter는 메서드의 파라미터로 ServletRequest나 ServletResponse를 가지고 있어서 사용자의 요청으로 들어온 Http 메서드 정보나, URI 정보 등을 얻을 수 있기 때문이다. ~~(잡담?)~~

## Filter란?
먼저 Filter에 대한 정의를 얘기하자면, Filter는 J2EE 표준 스펙으로 Servlet API 2.3 부터 등장하였고 <br>
DispatcherServlet에 요청이 전달되기 전, 후에 부가작업을 처리하는 객체이다.

## Spring Security 레퍼런스에서는?

다음은 <a href="https://docs.spring.io/spring-security/reference/servlet/architecture.html">Spring Security 레퍼런스</a>에 있는 사진이다.

![](/SpringSecurity/img/FilterChainImg.png)

클라이언트 사이드에서 애플리케이션에 요청을 보내게 되면 `Filter`와 `Servlet`을 포함하고 있는 컨테이너가 요청으로 들어온 URI에 맞는 `HttpServletRequest`를 처리하게 된다.<br>
대부분의 서블릿은 보통 하나의 `HttpServletRequest`, `HttpServletResponse`를 처리하게 되지만 여러 개의 필터는 다음과 같이 사용된다.
- 다음 필터 또는 서블릿이 호출되지 않도록 한다. 이 경우 일반적으로 필터는 `HttpServletResponse`를 작성한다.
- 다음 필터 또는 서블릿이 사용하는 `HttpServletRequest` 혹은 `HttpServletResponse` 수정

~~(공식 문서에 있는 내용인데, 뭔가 말이 어렵다..)~~

## 서블릿 필터

서블릿 필터가 제공하는 메서드는 `init`, `doFilter`, `destroy` 메서드를 제공해준다.
```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```
각 메서드에 대해 설명해보자면
1. `init`
    - `FilterConfig`를 파라미터로 받고 있는데, 이는 필터에 대한 정보를 가지고 있다. 
    - Filter가 생성될때 한 번만 호출되는 함수이다.
2. `destroy`
    - Filter가 소멸될 때 한 번만 호출되는 함수이다.
3. `doFilter`
    - 개발자는 이곳에 사용자의 요청을 처리하고 싶은 로직을 담으면 된다. 
    - 요청이 들어올 때마다 이 함수가 실행이 된다.
    - FilterChain 파라미터는 Filter가 여러 개 모여 생성된 체인이라고 생각하면된다.
    - 주의할 점은 FilterChain의 doFilter 메서드를 호출해야 다음 필터로 넘어간다는 점이다.

## Filter Chain
```java
public interface FilterChain {

    public void doFilter(ServletRequest request, ServletResponse response)
            throws IOException, ServletException;
}
```

FilterChain은 다음과 같이 `doFilter` 메서드를 가지고 있는 인터페이스이다. <br> 진짜 호출되는 것은 `ApplicationFilterChain`이고 테스트를 위해 FilteraChain을 Mocking하면 `MockFilterChain`이 동작하게 된다.<br>
`ApplicationFilterChain`의 `doFilter` 메서드는 다음과 같이 구현되어 있다.
```java
public void doFilter(ServletRequest request, ServletResponse response)
        throws IOException, ServletException {

        if( Globals.IS_SECURITY_ENABLED ) {
            final ServletRequest req = request;
            final ServletResponse res = response;
            try {
                java.security.AccessController.doPrivileged(
                        (java.security.PrivilegedExceptionAction<Void>) () -> {
                            internalDoFilter(req,res);
                            return null;
                        }
                );
            } catch( PrivilegedActionException pe) {
                Exception e = pe.getException();
                if (e instanceof ServletException) {
                    throw (ServletException) e;
                } else if (e instanceof IOException) {
                    throw (IOException) e;
                } else if (e instanceof RuntimeException) {
                    throw (RuntimeException) e;
                } else {
                    throw new ServletException(e.getMessage(), e);
                }
            }
        } else {
            internalDoFilter(request,response);
        }
    }
```
Global.IS_SECURITY_ENABLED는 시큐리티의 정책여부를 확인하는데, 기본 값이 null 이기 때문에 internalDoFilter 메서드를 타고 들어간다.<br>

> 궁금해서 debug 포인트를 찍고 어떤 `Filter` 구현체가 동작되나 보았더니 `OncePerRequestFilter`가 호출되는 것을 확인할 수 있었다. (궁금하신 분들은 디버그 포인트를 찍어보시길..)

![](/SpringSecurity/img/FilterChain.png)

위의 사진을 보면 pos와 n을 확인할 수 있는데 n은 총 등록된 Filter의 개수이다. <br>
이를 배열로 관리를 하는데 pos는 여기서 이 배열의 인덱스로 생각하면 된다. <br>

해당 filter 배열을 살펴보면, 
![](/SpringSecurity/img/filters.png)

다음과 같은 필터들이 있는 것을 확인할 수 있다. 이중 스프링 시큐리티에서 사용하는 `DelegatingFilterProxy`에 집중해야 한다. <br>
이 `DelegatingFilterProxy`가 스프링 시큐리티에서 사용하는 필터들을 기존 FilterChain에 끼워넣게 되는 다리 역할을 해준다.

## 정리
스프링 시큐리티를 사용하기 전, Filter에 대해 학습해보았다. 먼저 Filter는 Servlet (Spring MVC에서는 DispatcherServlet)에 요청이 들어가기 전, 후로 공통되는 로직을 처리해주는 객체이다.

이 Filter 메서드는 3가지 메서드를 가지고 있는데 우리가 집중해야 할 부분은 `doFilter`메서드이다. <br>
`doFilter` 메서드는 요청이 들어올 때마다 호출되기 때문에 개발자는 이곳에 로직을 작성하면 된다.

`doFilter`의 메서드 파라미터 `FilterChain`은 다음 필터를 호출해 줄 수 있는 인터페이스인데, 실제 요청이 들어오면 `ApplicationFilterChain`이 동작하게 되고 안에서 다시 `doFilter` 메서드를 호출시켜 다음 필터를 체인처럼 연결하는 것이다.<br>

## 참고 자료
[스프링 시큐리티 레퍼런스](https://docs.spring.io/spring-security/reference/servlet/architecture.html)