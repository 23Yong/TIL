# TIL
Today I Learned 공부한 것들을 정리합니다.

## JAVA
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **1** |**[JVM](./자바/JVM.md)**|
| **2** |**[어노테이션 프로세서](./자바/어노테이션프로세서.md)**|
| **3** |**[enum](./자바/enum.md)**|
| **4** |**[람다식](./자바/람다식.md)**|
| **5** |**[ThreadLocal](./자바/ThreadLocal.md)**|
| **6** |**[String은 왜 불변객체 인가](./자바/String.md)**|
| **7** |**[GC란?](./자바/GC.md)**|
| **8** |**[자바는 Call By Reference가 없다](./자바/CallByValue.md)**|
| **9** |**[자바에서의 직렬화](./자바/직렬화.md)**|
| **10**|**[HashMap](./자바/HashMap.md)**|

## Effective-Java
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **1** | **[아이템2 - 생성자에 매개변수가 많다면 빌더를 고려하라](/자바/effective-java/item2.md)**
| **2** | **[아이템3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라](/자바/effective-java/item3.md)** |
| **3** | **[아이템5 - 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](/자바/effective-java/item5.md)** |

## 백기선님의 LIVE-STUDY
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **1** |**[1주차 - JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가](./자바/live-study/1주차.md)**|
| **2** |**[2주차 - 자바 데이터 타입, 변수 그리고 배열](./자바/live-study/2주차.md)**|
| **3** |**[3주차 - 자바가 제공하는 다양한 연산자](./자바/live-study/3주차.md)**|
| **4** |**[4주차 - 자바가 제공하는 제어문](./자바/live-study/4주차.md)**|
| **5** |**[5주차 - 클래스](./자바/live-study/5주차.md)**|
| **6** |**[6주차 - 상속](./자바/live-study/6주차.md)**|
| **7** |**[7주차 - 패키지](./자바/live-study/7주차.md)**|
| **8** |**[8주차 - 인터페이스](./자바/live-study/8주차.md)**|
| **9** |**[9주차 - 예외처리](./자바/live-study/9주차.md)**|
| **10** |**[10주차 - 멀티 쓰레드 프로그래밍](./자바/live-study/10주차.md)**|
| **11** |**[11주차 - Enum](./자바/live-study/11주차.md)**|
| **12** |**[12주차 - 애노테이션](./자바/live-study/12주차.md)**|
| **13** |**[13주차 - I/O](./자바/live-study/13주차.md)**|
| **14** |**[14주차 - 제너릭](./자바/live-study/14주차.md)**|
| **15** |**[15주차 - 람다](./자바/live-study/15주차.md)**|

## HTTP웹
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **1** |**[상태코드](/HTTP/HTTP상태코드.md)**|
| **2** |**[HTTP헤더](/HTTP/HTTP헤더.md)**|
| **3** |**[콘텐츠협상](/HTTP/content_negotiation.md)**|
| **4** |**[쿠키](/HTTP/쿠키.md)**|
| **5** |**[캐시](/HTTP/캐시.md)**|

## Spring Core
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **0** |**[IoC컨테이너](./SpringCore/IoC.md)**|
| **1** |**[빈](./SpringCore/Bean.md)**|
| **2** |**[@Autowired](./SpringCore/@Autowired.md)**|
| **3** |**[@ComponentScan](./SpringCore/컴포넌트와스캔.md)**|
| **4** |**[빈의 스코프](./SpringCore/빈의스코프.md)**|
| **5** |**[Environment](./SpringCore/Environment.md)**|
| **6** |**[MessageSource](./SpringCore/MessageSource.md)**|
| **7** |**[ApplicationEventPublisher](./SpringCore/ApplicationEventPublisher.md)**|
| **7** |**[ResourceLoader](./SpringCore/ResourceLoader.md)**|
| **8** |**[리소스 추상화](./SpringCore/리소스추상화.md)**|
| **9** |**[Validation 추상화](./SpringCore/Validation추상화.md)**|
| **10** |**[AOP개념](./SpringCore/AoP개념.md)**|
| **11** |**[AOP활용](./SpringCore/AoP활용.md)**|

## Spring Core 2
| **번호** |  **제목**  |
| ------ | ------------------------------------------------------------------ |
| **1** |**[JDK Dynamic Proxy와 CGLIB](./SpringCore/SpringCore_Advanced/JDK동적프록시_vs_CGLIB.md)**|

## Spring MVC
| **파트** |  **번호**  | **제목** |
| --------- | ------ |------------------------------------------------------------------ |
| **웹 애플리케이션의 이해** | **1** |**[웹서버와 웹에플리케이션 서버](/SpringMVC/웹애플리케이션의이해/웹서버&WAS.md)**|
| | **2** |**[서블릿](/SpringMVC/웹애플리케이션의이해/서블릿.md)**|
| **MVC 프레임워크의 이해** | **1** | **[FrontController](/SpringMVC/MVC프레임워크의이해/front-controller_pattern.md)**|
| | **2** |**[View 분리](/SpringMVC/MVC프레임워크의이해/view의분리.md)**|
| | **3** |**[Model 추가](/SpringMVC/MVC프레임워크의이해/view의분리.md)**|
| | **4** |**[단순하고 실용적인 컨트롤러](/SpringMVC/MVC프레임워크의이해/단순실용컨트롤러.md)**|
| | **5** |**[다양한 컨트롤러의 적용](/SpringMVC/MVC프레임워크의이해/여러컨트롤러적용.md)**|
| **Spring MVC의 이해** | **1** | **[스프링MVC 전체적인 구조](/SpringMVC/SpringMVC/스프링MVC구조.md)**|
| | **2** | **[@RequestMapping사용](/SpringMVC/SpringMVC/@RequestMapping.md)** |
| | **3** | **[요청 파라미터](/SpringMVC/SpringMVC/Http요청파라미터.md)** |
| | **4** | **[메시지 컨버터](/SpringMVC/SpringMVC/HttpMessageConverter.md)** |
| | **5** | **[HTTP Response](/SpringMVC/SpringMVC/Http응답.md)** |
| | **6** | **[RequestMappingHandlerAdapter의 구조](/SpringMVC/SpringMVC/RequestMappingHandlerAdapter구조.md)** |
| | **7** | **[검증(Validation)](/SpringMVC/SpringMVC2/검증.md)** |
| | **8** | **[BeanValidation](/SpringMVC/SpringMVC2/BeanValidation.md)** |
| | **9** | **[로그인](/SpringMVC/SpringMVC2/로그인.md)** |
| | **10** | **[필터와 인터셉터](/SpringMVC/SpringMVC2/필터와인터셉터.md)** |
| | **11** | **[예외처리-서블릿](/SpringMVC/SpringMVC2/예외처리_서블릿.md)** |
| | **12** | **[예외처리-@ExceptionHandler](/SpringMVC/SpringMVC2/@ExceptionHandler.md)** |
| | **13** | **[스프링의 타입컨버터](/SpringMVC/SpringMVC2/스프링Converter.md)** |
| | **14** | **[MultipartResolver](/SpringMVC/SpringMVC2/MultipartResolver.md)** |

## JPA
|  **번호**  | **제목** |
| --------- | ------ |
| **1** | **[엔티티매니저와 영속성 컨텍스트](/JPA/엔티티매니저.md)** |
| **2** | **[엔티티매핑](/JPA/엔티티매핑.md)** |
| **3** | **[단방향연관관계](/JPA/단방향연관관계.md)** |
| **4** | **[양방향연관관계](/JPA/양방향연관관계.md)** |
| **5** | **[상속관계](/JPA/상속관계매핑.md)** |
| **6** | **[즉시 로딩과 지연 로딩](/JPA/즉시로딩과지연로딩.md)** |
| **7** | **[영속성 전이](/JPA/영속성전이.md)** |
| **8** | **[JPQL](/JPA/JPQL.md)** |
| **9** | **[JPA join types](/JPA/JPA조인타입.md)** |
| **10** | **[N+1 문제](/JPA/N+1문제.md)** |
| **11** | **[페이징문제](/JPA/페이징과한계돌파.md)** |
| **12** | **[영속성 컨텍스트와 트랜잭션](/JPA/영속성컨텍스트와트랜잭션.md)** |
| **13** | **[OSIV](/JPA/OSIV.md)** |

## 스프링 데이터 JPA
| **번호** | **제목** |
| ------- | ------- |
| **1** | **[스프링 데이터 JPA와 공통 인터페이스](/JPA/SpringDataJPA/스프링_데이터_JPA와_공통_인터페이스.md)** |
| **2** | **[쿼리메서드 - 3가지 생성방법](/JPA/SpringDataJPA/쿼리메서드_시작.md)** |
| **3** | **[쿼리메서드 - 페이징과 정렬](/JPA/SpringDataJPA/쿼리메서드_페이징과정렬.md)** |
| **4** | **[쿼리메서드 - 벌크성 수정 쿼리](/JPA/SpringDataJPA/벌크성수정쿼리.md)** |
| **5** | **[Auditing](/JPA/SpringDataJPA/Auditing.md)** |
| **6** | **[스프링 데이터 JPA 구현체](/JPA/SpringDataJPA/스프링데이터JPA분석.md)** |

## 스프링 시큐리티
| **번호** | **제목** |
| ------- | ------- |
| **1** | **[스프링 시큐리티 들어가기 전 (Filter)](/SpringSecurity/Filter.md)** |
| **2** | **[DelegatingFilterProxy](/SpringSecurity/DelegatingFilterProxy.md)** |
| **3** | **[FilterChainProxy](/SpringSecurity/FilterChainProxy.md)** |

## 개발하다가 궁금한 것들..
| **번호** | **제목** |
| ------- | ----------------- |
| **1** | [API](/정리/API.md) |
| **2** | [DTO란?](/정리/DTO.md) |