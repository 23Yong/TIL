# HTTP Message Converter

뷰 템플릿을 이용해 HTML을 응답하는 것이 아닌, HTTP API처럼 JSON데이터를 HTTP 메시지 바디에 직접 넣으려고 하는 경우 HTTP 메시지 컨버터를 사용할 수 있다.

## 스프링 MVC에서의 HTTP 메시지 컨버터
스프링 MVC는 다음의 경우 HTTP 메시지 컨버터를 적용한다.
- HTTP 요청: ```@RequestBody```, ```HttpEntity(RequestEntity)```
- HTTP 응답: ```@ResponseBody```, ```HttpEntity(ResponseEntity)```

HttpMessageConverter 인터페이스를 살펴보자.
```java
public interface HttpMessageConverter<T> {
    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

    List<MediaType> getSupportedMediaTypes();

    default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
		return (canRead(clazz, null) || canWrite(clazz, null) ?
				getSupportedMediaTypes() : Collections.emptyList());
	}

    T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
            throws IOException, HttpMessageNotReadableException;

    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;
}
```

- ```canRead```, ```canWrite```: 파라미터로 주어진 클래스, 미디어타입을 지원하는지 판단한다. 읽을 수 있다면 true, 그렇지 않다면 false를 반환한다.
- ```read```, ```write```: 메시지를 읽고 쓰는 기능이다.
- ```getSupportedMediaTypes```: 메시지 컨버터가 지원하는 미디어타입 리스트를 반환한다. 


## 주요 메시지 컨버터
- ByteArrayHttpMessageConverter: byte[] 타입 데이터를 처리한다.
    - 클래스 타입: byte[], 미디어 타입: \*/*
    
- StringHttpMessageConverter: String 문자로 데이터를 처리한다.
    - 클래스 타입: String, 미디어 타입: \*/*
    - ```java
        @RequestMapping
        void hello(@RequestBody String data) {}
        ```
- MappingJackson2HttpMessageConverter: application/json
    - 클래스 타입: 객체 또는 HashMap, 미디어타입: application/json 관련
    - ```java
        @RequestMapping
        void hello(@RequestBody Hello data) {}
        ```
## @RequestBody
@RequestBody 애노테이션을 이용해 http의 request body를 읽어 HttpMessageConverter를 통해 request body를 객체로 변환할 수 있다.

```java
@PostMapping("/members")
public void handle(@RequestBody Member member) {
    // ...
}
```

## @ResponseBody
HttpMessageConverter를 통해 http의 body에 문자내용을 직접 반환할 때 사용할 수 있다. 기본 문자처리는 StringHttpMessageConverter가, 기본 객체 처리는 MappingJackson2HttpMessageConverter가 처리한다.

```java
@GetMapping("/members/{id}")
@ResponseBody
public Member handle(@PathVariable long id) {
    Member member = new Member(id);
    return member;
}
```

## 참고 자료
[스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard)

[스프링MVC ref](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-responsebody)