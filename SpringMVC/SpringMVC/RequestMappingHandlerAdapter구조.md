# RequestMappingHandlerAdapter 구조
SpringMVC의 전체적인 구조는 다음과 같았다.
![](/img/V5.png)

이전에 했던 메시지컨버터는 어느 단계에서 적용되는 것일까. 

이를 위해 ```@RequestMapping```애노테이션을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter를 알아보자.

## RequestMappingHandlerAdapter 동작 방식
애노테이션 방식의 핸들러(컨트롤러)는 매우 다양한 파라미터를 사용할 수 있었다. 스프링은 어떻게 파라미터들을 유연하게 처리할 수 있을까.

![](/img/RequestMappingHandlerAdapter.png)

## ArgumentResolver

이는 ArgumnetResolver덕분인데, 애노테이션 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 이 ArgumentResolver를 호출해서 핸들러(컨트롤러)가 필요한 파라미터의 값들을 생성시킨다. 

이 ArgumentResolver의 코드를 살펴보자. 원래는 HandlerMethodArgumentResolver이다.

```java
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter parameter);

    @Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```

주석을 살펴보면
- ```supportsParameter```: 주어진 메서드의 파라미터가 이 resolver에 의해 처리될 수 있는 값인지 확인한다. 
- ```resolveArgument```: 메서드의 파라미터들을 주어진 요청의 인자 값으로 resolving한다. ModelAndView 컨테이너는 요청에 대한 모델의 접근을 제공한다. ```WebDataBinderFactory```가 데이터 바인딩과 type 변환 목적으로 필요할 때 WebDataBinder 인스턴스를 만드는 방법을 제공한다.

이렇게 적혀있다. 사실상 supportsParameter를 통해 주어진 파라미터를 지원하는지 확인 후, 처리할 수 있다면 객체를 생성해 반환하는 것으로 생각할 수 있다. 그리고 이렇게 생성된 객체가 컨트롤러가 호출될 때 넘어가는 것이다.

## ReturnValueHandler

핸들러(컨트롤러)가 요청을 처리해 어떠한 값을 반환을 할텐데, 단순히 ```@Controller``` 애노테이션만 붙였다면 ModelAndView를, ```@ResponseBody``` 애노테이션을 붙였다면 HTTP 메시지 바디에 들어갈 메시지를 직접 반환을 할 것이다. 이는 ReturnValueHandler 덕분인데, 원래는 HandlerMethodReturnValueHandler이다.

```java
public interface HandlerMethodReturnValueHandler {
	boolean supportsReturnType(MethodParameter returnType);

	void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception;

}
```
주석을 살펴보면
- ```supportsReturnType```: 메서드의 반환 타입이 이 핸들러에 의해 처리될 수 있는지 확인한다.
- ```handleReturnValue```: 모델에 attribute를 더하고 뷰를 설정하거나 ```ModelAndViewContainer.setRequestHandled```의 flag를 true로 설정하여 응답이 직접 처리되었음을 나타내는 주어진 반환 값을 처리한다.


## HTTP 메시지 컨버터
다시 처음으로 돌아가서 그렇다면 메시지 컨버터는 어디에 적용되는 것일까.

HTTP 메시지 컨버터를 사용하는 @RequestBody는 파라미터의 반환 값에 사용되고 @ResponseBody는 컨트롤러의 반환 값에 사용된다.

- 요청의 경우는 @RequestBody를 사용하는 ArgumentResolver가 있고, HttpEntity를 처리하는 ArgumentResolver가 있다. 이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다.

- 응답의 경우도 마찬가지이다. HTTP 메시지 컨버터를 호출해서 응답 결과를 만드는 것이다.

- 스프링 MVC는 @RequestBody, @ResponseBody가 있으면 `RequestResponseBodyMethodProcessor`(ArgumentResolver)가, HttpEntity가 있으면 `HttpEntityMethodProcessor`(ArgumentResolver)를 사용한다.


## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)