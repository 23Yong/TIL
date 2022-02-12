# BeanValidation

[지난번에 작성했던 것](./검증.md)처럼 일일이 모든 것을 검증하는 로직을 컨트롤러에 넣는 것은 번거롭다.
따라서 애노테이션 기반의 검증 방법이 있는데, 이것이 BeanValidation이다. 이런 검증 로직을 모든 프로젝트에 적용할 수 있도록 공통화시켜놓았다.

## BeanValidation이란?
Bean Validation은 특정 구현체가 아니라 Bean Validation 2.0이라는 기술 표준이다.
자세한 내용은 [이곳](/SpringCore/Validation추상화.md)에 정리를 해두었다.

## 스프링부트와 Bean Validator
스프링 부트에 `implementation 'org.springframework.boot:spring-boot-starter-validation'` 다음과 같이 라이브러리를 추가하면 자동으로 Bean Validator를 인지하고 스프링과 통합한다. 

스프링 부트는 `LocalValidatorFactoryBean`을 글로벌 Validator로 등록해 @NotNull, @NotBlank같은 애노테이션을 보고 검증을 수행하게 된다. 이렇게 글로벌 Validator가 등록되어 있기 때문에 우리는 `@Validated`, `@Valid` 애노테이션만 적용해 쉽게 검증을 수행할 수 있는 것이다.

주의할 점이 하나 있는데, 우리가 직접 글로벌 Validator를 등록하게 되면 스프링 부트는 Bean Validator를 글로벌 Validator로 등록하지 않기 때문에 이 점을 주의하자.

### BeanValidator 검증
BeanValidator는 바인딩에 실패한 것은 검증하지 않는다. 타입 변환에 성공해 바인딩이 된 필드여야 BeanValidation이 의미가 있기 때문에 당연한 것이다. 

만약 검증에 성공해 오류 메시지를 자세하게 띄워주고 싶을 때는 어떻게 할 수 있을까?
errors.properties파일을 통해 이것을 가능하게 할 수 있는데, 검증을 통해 들어온 검증 오류 코드를 보면 @NotBlank같은 경우 MessageCodesResolver를 통해 다음과 같은 메시지 코드가 순서대로 들어온다.
- @NotBlank
    - NotBlank.item.itemName : 애노테이션이름.오브젝트이름.필드이름
    - NotBlank.itemName
    - NotBlank.java.lang.String
    - NotBlank

이를 통해 errors.properties에 다음과 같은 메시지를 등록할 수 있다.
```
NotBlank= 공백은 허용되지 않습니다.
Range= {0}, {2} ~ {1} 하용
```
{0}은 필드명이고 {1}, {2}는 애노테이션 마다 다르다.

### 동일한 모델 객체를 다르게 검증
동일한 모델 객체에 대해서 추가할 때는 quantity에 @Max(9999) 수정할 때는 자유롭게 값을 넣을 수 있도록 수정하고 싶다. 이때 사용할 수 있는 방법이 2가지가 있는데 다음과 같다.
1. BeanValidation의 groups를 이용하는 것
2. 도메인 모델을 직접 사용하지 않고 폼 전송을 위한 별도의 객체를 만들어 사용하는 것

groups를 적용하는 것을 먼저 살펴보자.

먼저 2개의 인터페이스를 만든다.
```java
public interface SaveCheck {}
```
```java
public interface UpdateCheck {}
```

그리고 도메인 모델에 다음과 같이 적용한다.
```java
@Data
public class Item {

    @NotNull(groups = UpdateCheck.class)
    private Long id;

    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Max(value = 9999, groups = SaveCheck.class)
    private Integer quantity;

    ...
}
```

이후 컨트롤러에서 다음과 같이 사용하면 해당되는 그룹만 검증이 적용된다.
```java
@PostMapping("/add")
public String addItem(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult) {
    ///...
}
```

하지만 이는 실무에서는 잘 사용하지 않는다. 등록 시 폼에서 전달하는 데이터가 도메인 객체와 딱 맞는 경우가 별로 없기 때문이다. 

그래서 폼의 데이터 객체를 따로 만들어 컨트롤러에 전달하는 방식을 이용한다. 예를 들어 ItemSaveForm, ItemUpdateForm 이런식으로 말이다. 이후 컨트롤러에서는 필요한 데이터를 가지고 도매인 객체를 생성한다.

## 참고 자료
[김영한- MVC2](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2/dashboard)
