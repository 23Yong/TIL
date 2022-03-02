# 스프링 데이터 JPA 분석

## 스프링 데이터 JPA 구현체 분석
JpaRepository 인터페이스를 구현하고 있는 JPA의 구현체를 살펴보자.
스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체는 `SimpleJpaRepository`이다.

`SimpleRepository`에서 살펴봐야 할 것이 몇 가지 있는데 우선 코드를 살펴보자.

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    // ...

    @Transactional
	@Override
	public <S extends T> S save(S entity) {

		Assert.notNull(entity, "Entity must not be null.");

		if (entityInformation.isNew(entity)) {
			em.persist(entity);
			return entity;
		} else {
			return em.merge(entity);
		}
	}

    // ...
}   
```

### @Repository
구현 클래스를 보면 `@Repository` 애노테이션을 달고 있음을 알 수 있다. 이것이 의미하는 바는 2가지가 있다.
1. 컴포넌트 스캔의 대상이 된다.
2. JPA의 예외들을 스프링이 추상화한 예외로 변환해준다.

1을 보면 컴포넌트 스캔의 대상이 되기 떄문에 따로 빈으로 등록해 줄 필요가 없다. 중요한 것은 2인데, 스프링이 추상화한 예외로 JPA의 예외들을 변환시켜주기 때문에 하부 기술에 종속적일 필요가 없다는 것이다. 후에 MyBatis나 JDBC로 바꾸어도 예외처리 부분의 로직을 바꿀 필요가 없다는 것을 의미한다.

### @Transactional(readOnly = true)
트랜잭션 애노테이션을 클래스레벨에서 단 것을 통해 JPA의 모든 동작은 트랜잭션 안에서 일어나는 것을 알 수 있다. 기본적으로 readOnly 옵션을 true로 주면서 변경(등록, 수정, 삭제)메서드들은 @Transactional을 통해 트랜잭션 처리를 한 것을 구현체에서 확인할 수 있다.

서비스 계층에서 트랜잭션을 시작하지 않아도 리포지토리 계층에서 트랜잭션이 시작되고, 서비스 계층에서 트랜잭션이 이미 있다면 해당 트랜잭션을 전파받아 그 트랜잭션을 사용한다.

readOnly 옵션을 true로 설정함으로써 최적화를 하고 있는데, readOnly가 true이면 후에 영속성 컨텍스트에 있는 엔티티를 조회하고 트랜잭션이 커밋되도 플러시가 일어나지 않는다. 이는 읽기 전용이라고 명시를 했기 때문에 영속성 컨텍스트는 스냅샷 인스턴스를 보관하지 않고 스냅샷이 없어 수정해도 데이터베이스에 반영되지 않기 때문이다.

### Persisting Entities
save메서드를 보면 isNew 메서드가 true 일때는 persist 메서드를 호출하지만, false 일때는 merge 메서드를 호출하고 있다.

스프링 데이터 JPA는 엔티티가 새로 생성된 것인지 아닌지를 판단하는 전략을 제공한다.
1. Version-Property와 Id-Property (default) 
    - 기본으로 스프링 데이터 JPA에서 non-primitive 타입은 `null`일 경우 새로운 엔티티로 판별된다. Version-Property 없이 스프링 데이터 JPA는 identifier property가 널인지 검사한다. `null`일 경우 새로운 엔티티로 취급하고 그렇지 않으면 이미 영속성 컨텍스트에 들어왔던 엔티티로 판별한다.
2. `Persistable` 인터페이스 구현
    - 만약 엔티티가 `Persistable` 인터페이스를 구현하고 있다면 스프링 데이터 JPA는 엔티티의 isNew() 메서드를 위임한다.
    - ```java
        package org.springframework.data.domain;
        public interface Persistable<ID> {
            ID getId();
            boolean isNew();
        }
        ```

참고로 식별자 생성 전략이 @GeneratedValue를 사용하면 save() 호출 시점에 새로운 엔티티로 인식해 정상 동작하지만, @Id 만 사용할 경우 값을 주고 시작하기 때문에 새로운 엔티티로 인식하지 못한다. 따라서 merge() 메서드가 호출이 된다. 
이런 경우 `Persistable` 인터페이스를 사용해 isNew 메서드를 새로 구현하는게 이득이다.

참고로 @CreatedDate를 조합해 사용하면 새로운 엔티티인지 편리하게 확인할 수 있다.


## 참고 자료
[스프링 데이터 JPA-Persisting Entities](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.entity-persistence)

[인프런 실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard)

[자바 ORM 표준 JPA 프로그래밍](http://www.yes24.com/Product/Goods/19040233)