# Auditing
데이터베이스에서 누가, 언제 엔티티를 생성하고 수정했는지 기록하는 것은 중요한 일이다. 거의 모든 테이블에 생성일자, 수정일자 등을 컬럼으로 두고 있다. 이를 JPA에서는 어떻게 기록하는지 알아보자.

## 순수 JPA - Auditing
순수 JPA에서는 도메인마다 공통되는 컬럼인 생성일자, 변경일자 등을 따로 클래스로 뺴 사용할 수 있다. 코드로 알아보자.

```java
@MappedSuperClass
@Getter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updateDate = LocalDateTime.now();
    }
}
```
생성일자와 변경일자를 컬럼으로 가지는 클래스를 선언해두고 이를 엔티티가 상속받게 해 컬럼들을 내린다.

```java
@Entity
public class Member extends JpaBaseEntity {
    // ...
}
```

- @PrePersist : 엔티티가 영속화되기 전 `@PrePersist` 애노테이션이 달린 메서드를 호출한다.
- @PreUpdate : update 쿼리가 실행되기 전 `@PreUpdate` 애노테이션이 달린 메서드를 호출한다.

이런 방식으로 생성날짜와 수정날짜를 기록해도 되지만 스프링 데이터 JPA에선 또 다른 방법을 제공한다.

## 스프링 데이터 JPA - Auditing
스프링 데이터 JPA는 Auditing 정보를 트리거하는데 사용할 수 있는 엔티티리스너를 지원한다. `@EntityListeners` 애노테이션이다.

이를 사용하기 위해선 `@EnableJpaAuditing` 애노테이션을 스프링 부트 클래스에 적용해주어야 한다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

이제 사용법을 살펴보자.
```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    private String lastModifiedBy;
}
```

애노테이션 기반의 Auditing 메타 데이터를 생성하고 있다. 사용되는 애노테이션은 
- `@CreatedDate`
- `@LastModifiedDate`
- `@CreatedBy`
- `@LastModifiedBy`

들이 있다. 

생성자 정보를 기록하기 위해서는 `AudtingAware` 스프링 빈을 등록해야 한다.
```java
@Bean
public AudtingAware<String> auditorProvider() {
    return () -> Optional.of(UUID.randomUUID().toString());
}
```
임시로 UUID를 랜덤으로 생성해 등록자, 수정자에 대한 정보를 기록하였다. 실제로는 세션과 같은 정보에서 ID를 받아서 사용한다.


## 참고 자료
[인프런 실전! 스프링 데이터 JPA](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84/dashboard)

[Auditing](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#auditing)
