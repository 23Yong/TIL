# 스프링 타입 컨버터(Converter)

웹 애플리케이션 개발 시 타입을 변환해야 하는 경우가 있다. 이런 경우 사용할 수 있는 것이 `Converter`이다. 이에 대해 알아보자.

## Converter 인터페이스
```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S source);
}
```

다음은 스프링이 제공하는 인터페이스이다. S를 T로 전환하는 convert 메서드를 제공한다. 함수적 인터페이스이기 떄문에 람다식 혹은 메서드 참조의 대상으로 사용할 수도 있다.

각 convert() 메서드의 S는 null이 아님을 보장한다. 만약 사용자가 커스텀한 converter가 conversion에 실패했다면 uncheckedException이 던져진다. Converter의 구현은 Thread-safe해야된다는 것을 주의하자.  

스프링은 문자, 숫자, boolean, Enum등 일반적인 타입에 대한 컨버터를 기본으로 제공하고 있다는 것을 참고하자.

## ConversionService
스프링은 개별 컨버터들을 모아두고 묶어서 관리할 수 있는 기능을 제공해주는데 그것이 ConversionService이다. 

```java
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

ConversionService는 컨버팅이 가능한지의 여부와 convert해주는 기능을 제공한다. 대부분의 ConversionService의 구현체들은 ConverterRegistry 인터페이스도 구현하고 있는데, 이 인터페이스는 컨버터 등록에 초점을 맞춘 인터페이스이다. 이렇게 인터페이스를 분리해 컨버터를 사용하는 클라이언트와 컨버터를 등록하는 클라이언트의 관심을 분리할 수 있다. (ISP 원칙)

## 스프링에 Converter 사용
스프링은 내부에서 ConversionService를 제공하고 있다. WebConfigurer가 제공하는 addFormatters() 메서드를 사용해 제공하는 컨버터를 등록하면 스프링은 내부에서 ConversionService에 컨버터를 등록해준다.

ConversionService는 Controller의 @RequestParam, @ModelAttribute, @PathVariable 등에서 자주 사용한다. 처리과정은 이를 처리하는 ArgumentResolver에서 ConversionService를 사용해 타입을 변환해 처리해준다고 이해하면 된다.

## 참고자료
[스프링 공식문서](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert-Converter-API)

[스프링MVC 2편 - 김영한(인프런)](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
