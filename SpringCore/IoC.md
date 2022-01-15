# IoC Container
IoC는 DI로도 알려져 있다. 어떤 객체가 사용하는 의존 객체를 직접 만들어서 사용하는 것이 아닌, 주입받아 사용하는 방법을 말한다.

`org.framework.beans` 와 `org.springframework.context` 패키지의 기본은 IoC 컨테이너이다. IoC 컨테이너는 애플리케이션 컴포넌트의 중앙저장소이고, 빈 설정파일을 통해 빈 정의를 읽어들이고 빈을 구성하고 제공한다. `BeanFactory` 인터페이스는 Spring 빈 컨테이너에 접근하기 위한 루트 인터페이스이다. `ApplicationContext`는 `BeanFactory`를 상속받고 있으며 추가 기능들을 제공한다.

## ApplicationContext
ApplicationContext 인터페이스는 빈의 인스턴스화, 구성을 담당한다. 컨테이너는 configuration 메타데이터를 읽음으로써 빈의 인스턴스화, 구성 등에 대한 명세를 가져온다. configuration 메타데이터는 XML, 자바 애노테이션, 혹은 자바 코드를 통해 나타날 수 있다.
- ClassPathXmlApplicationContext(XML)
- AnnotationConfigApplicationContext(Java)

## 참고 자료
[SpringCore IoC컨테이너 doc](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-introduction)
