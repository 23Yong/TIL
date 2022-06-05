# DataSource
[여기서](./ConnectionPool.md) 커넥션을 얻는 방법은 JDBC의 DriverManager를 사용하거나 커넥션 풀을 사용하는 등 다양한 방법이 존재한다.

만약 JDBC의 Drvier Manager 방법을 통해 커넥션을 획득하다가 Connection Pool로 바꾸게 되면 어떻게 될까?

커넥션을 얻어오는 애플리케이션 코드를 변경해야 될 것이다. 이는 의존관계가 `DriverManager`에서 `Conncection Pool`로 변경되기 때문이다. 

## 커넥션 획득의 추상화
위와 같이 코드를 변경하지 않고 커넥션을 획득하기 위해 자바에서는 `javax.sql.DataSource`라는 인터페이스를 제공한다. `DataSource`는 커넥션을 획득하는 방법을 추상화하는 인터페이스이다. 

![](/img/DataSource.png)

DataSource 인터페이스의 핵심기능은 커넥션을 획득하는 것이다. 밑의 핵심 코드만 보자.
```java
public interface DataSource extends CommonDataSource, Wrapper {

    Connection getConnection() throws SQLException;

    // ...
}
```

## 정리
- 대부분의 커넥션 풀들은 `DataSource` 인터페이스를 구현해두었다. 따라서 개발자는 `DataSource`에만 의존해 코드를 작성하면 된다. 
- 커넥션 풀 기술을 교체하고 싶으면 DataSource의 구현체를 바꾸면 된다.
- `DriverManager`는 `DataSource`를 사용하지 않는다. 따라서 `DriverManager`를 사용하다가 커넥션 풀 기술로 변경하고 싶을 때는 코드를 교체해야 한다. 하지만 스프링은 이런 문제를 해결하기 위해 `DriverManager`도 `DataSource`를 사용할 수 있도록 `DriverManagerDataSource`라는 `DataSource`를 구현한 클래스를 제공한다.

## 참고자료
[인프런 - 스프링 DB 1편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard)
