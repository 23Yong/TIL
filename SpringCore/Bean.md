# Bean
먼저, 다음 내용은 스프링 Document을 읽은 것을 필요한 부분만 해석해 본 것이다.

Spring의 IoC컨테이너는 하나이상의 빈을 관리한다. IoC컨테이너 안에 있는 빈들은 Configuration metadata가 컨테이너에 제공하는 데이터로 인해 만들어진다. 

컨테이너 내에서 빈들의 정의는 ```BeanDefinition``` 객체로 표시되며, 다음과 같은 정보를 포함하고 있다.
- package-qualified 클래스 이름
    - 일반적으로 빈들이 정의되는 실제 구현 클래스
- 컨테이너 내에서 빈들이 어떻게 작동되어야 하는지 등에 대한 Bean 동작 Configuration(scope, 라이프 사이클 콜백 등..)
- 빈이 동작하기 위해 필요로 하는 다른 빈의 레퍼런스
- 새로 생성된 객체에 설정할 기타 설정
    - pool의 크기, 커넥션 풀을 관리하는 Bean에서 사용할 커넥션 수

메타 데이터는 각 빈 정의를 구성하는 프로퍼티의 집합으로 변환된다. 밑의 표는 이런 프로퍼티들을 나타낸다.

| **Property** | **다음 링크에서 설명** |
| --- | --- |
| Class | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname)|
| Scope | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructing Arguments | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)|
| Lazy initialization mode | [Lazy-intializied Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)|

그리고 빈을 생성하는 방법에 대한 정보를 포함하고 있는 빈 정의 외에도, ```ApplicationContext```구현체들은 컨테이너 밖에서 만든 객체들의 등록도 허용한다.(사용자에 의한 생성)

## Naming Beans
모든 빈들은 하나 이상의 식별자를 가지고 있다. 이 식별자들은 빈들을 호스팅하는 컨테이너 내에서 반드시 유일해야 한다. 일반적으로 빈들은 하나의 식별자만 가지지만, 두 개 이상이 필요하면 추가 식별자는 별명(aliases)으로 간주된다.

XML기반의 Configuration 메타데이터에서 빈들은 id나 name이 식별자로 지정된다.
- id는 하나의 id만을 지정한다.
- 빈에 다른 별명을 붙여주러면, ','나 ';' 혹은' '을 구분자로 사용해서 name속성에 지정해도 된다.

빈의 name이나 id를 반드시 지정하지 않아도 된다. 만약 name이나 id를 명시적으로 지정하지 않았다면, 컨테이너가 빈의 유일한 이름을 자동으로 생성한다. 하지만 ref나 Service Locator 스타일 검색을 통해서 이름으로 빈을 참조하려한다면 name을 명시해주어야 한다.

## Instantiating Beans
빈 정의는 하나 이상의 객체를 생성하기 위해서는 필수적이다. 컨테이너는 요청을 받은면 이름이 있는 빈의 레시피를 보고, 캡슐화된 Configuration 메타 데이터를 사용해 실제 객체를 생성(혹은 획득)한다.

### Instantiation with a Constructor
생성자를 통해 빈을 생성할 때, 모든 일반 클래스들은 Spring과 호환이 가능하다. 즉, 특정 인터페이스를 구현하거나 특정한 방식으로 코드를 짜지 않아도 된다. 간단하게 빈의 클래스를 지정하면 된다. 하지만 특정 빈을 사용하는 IoC유형에 따라, 기본 생성자가 필요한 경우가 있다.

스프링의 IoC 컨테이너는 거의 모든 클래스들을 관리할 수 있다. 컨테이너는 완전한 자바 빈만을 관리하지 않는다. 대부분의 스프링 사용자들은 기본 생성자와 적당한 getter, setter로 모델링된 JavaBeans을 선호한다. 

컨테이너에 평범한 빈 스타일과 다른 형태의 클래스들을 둘 수 있다. 예를 들어, JavaBean 스펙을 준수하지 않는 레거시 커넥션 풀을 사용하는 경우에도 Spring으로 관리할 수 있다.

## Determining a Bean’s Runtime Type
런타임 시간에 빈이 실제로 어떤 타입을 가지고 있는지는 쉽지 않은 문제이다. 빈 메타데이터 정의에 지정된 클래스는 단순히 초기화 클래스의 참조일 뿐이고, 명시된 팩토리 메서드나 ```FactoryBean```이 된 클래스들과의 조합으로 인해 런타임시 빈의 타입이 달라질 수 있다. 혹은 인스턴스 레벨 팩토리 메서드를 사용했다면, 런타임 타입을 지정하지 않는다.
그리고 AOP프록시 빈 인스턴스를 인터페이스 기반 프록시로 감쌀 수 있다.

특정 빈이 어떤 타입을 가지고 있는지 알아내기 위해 추천하는 방법은 다음과 같다.
- 빈의 이름으로 ```BeanFactory.getType``` 메서드를 호출
    - 이 방법은 모든 경우를 고려해 객체의 타입을 반환하고 반환 값은 ```BeanFactory.getBean``` 메서드가 반환하는 것과 같은 빈이다.


## 참고 자료 
[스프링 코어 레퍼런스](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition)
