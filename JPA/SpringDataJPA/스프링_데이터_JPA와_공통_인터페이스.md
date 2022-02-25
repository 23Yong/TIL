# 스프링 데이터 JPA와 공통 인터페이스

## 스프링 데이터 JPA란
스프링 데이터 JPA는 스프링 프레임워크에서 JPA를 사용하기 쉽게 할 수 있도록 지원하는 프로젝트이다. 데이터 접근 계층을 개발할 때 지루하게 반복되는 코드들의 양을 상당히 줄여준다. 리포지토리 개발 시 인터페이스만 작성하면 실행 시점에 스프링 데이터 JPA가 구현 객체를 동적으로 생성해 주입해준다. 이에 따라 데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수 있다.

실제로 JpaRepository를 상속받는 MemberRepository를 만들고 @Autowired를 통해 주입받은 memberRepository를 출력해보면 다음과 같다.

```java
@Autowired
MemberRepository memberRepository;

public void test() {
    System.out.println("memberRepository = " + memberRepository.getClass());
}
```
```
memberRepository = class com.sun.proxy.$Proxy117
```

### 스프링 데이터 프로젝트
스프링 데이터 JPA는 스프링 데이터 프로젝트의 하위 프로젝트 중 하나이다. 스프링 데이터 프로젝트는 JPA, MongoDB, REDIS 등등 다양한 데이터 저장소에 대한 접근을 추상화해 반복되는 데이터 접근 코드를 줄여준다. 스프링 데이터 JPA는 JPA에 특화된 기능을 제공해준다.

## 스프링 데이터 핵심
스프링 데이터 리포지토리 추상화에서 핵심 인터페이스는 `Repository`이다. 이 인터페이스는 주로 작업할 유형들을 얻고 이 인터페이스를 상속받고 있는 인터페이스를 검색하는데 도움이 되는 마커 인터페이스 역할을 한다. `CrudRepository`인터페이스는 엔티티 클래스를 위한 정교한 CRUD 메서드를 지원한다.

### CrudRepository
```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {

  <S extends T> S save(S entity);      

  Optional<T> findById(ID primaryKey); 

  Iterable<T> findAll();               

  long count();                        

  void delete(T entity);               

  boolean existsById(ID primaryKey);   

  // … more functionality omitted.
}
```

### PagingAndSortingRepository
CrudRepository를 상속받고 있는 `PagingAndSortingRepository` 인터페이스는 엔티티의 페이징 처리를 쉽게 해주는 메서드들이 들어가 있다.

```java
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

## 스프링 데이터 JPA 설정
스프링 데이터 JPA를 사용하기 위해서는 JavaConfig를 설정해주어야 한다.
```java
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {} 
```
그런데 스프링 부트를 사용하면 이런 과정이 필요 없다. @SpringBootApplication 의 패키지와 하위 패키지를 인식하기 때문이다.

물론 위치가 달라지면 `@EnableJpaRepositories`를 사용해야 한다.

## 공통 인터페이스 기능
스프링 데이터 JPA는 `JpaRepository<T, ID>` 인터페이스를 제공해준다.
```java
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    // ...
}
```

스프링 데이터 JPA를 사용하는 가장 간단한 방법은 이 인터페이스를 상속받는 것이다.
```java
public interface MemberRepository extends JpaResitory<Member, Long> {
}
```
제네릭에 회원 엔티티와 회원 엔티티의 식별자 타입을 지정했다. 여기서 `@Repository` 애노테이션을 생략해도 되는데, 컴포넌트 스캔 과정을 스프링 데이터 JPA가 자동으로 처리해주기 때문이다.

이를 통해 스프링 데이터 JPA는 애플리케이션 실행 시 컴포넌트 스캔을 통해 리포지토리 인터페이스를 찾아 해당 인터페이스를 구현한 클래스를 동적으로 생성해준다. 이후 이를 빈으로 등록해준다.

이제 `MemberRepository`는 `JpaRepository`가 제공하는 다양한 기능을 사용할 수 있는데, JpaRepository의 계층 구조를 한 번 살펴보자.

![](/img/스프링데이터JPA계층구조.png)

스프링 데이터 모듈이 있고 그안에 Repository, CrudRepository, PagingAndSortingRepository가 있다. 이는 스프링 데이터 프로젝트가 공통으로 사용하는 인터페이스이다. 스프링 데이터 JPA가 제공하는 JpaRepository는 여기에 JPA에 특화된 기능을 추가적으로 제공한다.

실제로 `JpaRepository`의 패키지를 보면 `package org.springframework.data.jpa.repository;` 과 같고, 나머지들은 `package org.springframework.data.repository;`과 같다.


## 참고자료
[Spring Data JPA 2.6.2 레퍼런스](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories)

[인프런 실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard)

[자바 ORM 표준 JPA 프로그래밍](http://www.yes24.com/Product/Goods/19040233)