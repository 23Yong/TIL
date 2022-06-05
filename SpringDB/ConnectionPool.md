# 커넥션 풀과 DataSource

## 학습할 것
- DB 커넥션
- DB 커넥션 획득 방법
- 커넥션 풀

## DB 커넥션
DB 커넥션은 DB와 애플리케이션 간 통신을 하기 위한 수단이다. 
Java 에서 DB 커넥션을 획득하기 위해 주로 JDBC를 사용하는데, URL 타입을 이용한다.

## DB 커넥션 획득 방법
사용자는 데이터베이스 커넥션을 획득하기 위해 다음과 같은 과정을 거치게 된다.
![](/img/DBConnection.png)

1. 애플리케이션 로직이 DB 드라이버를 통해 커넥션을 조회한다.
2. DB 드라이버가 데이터베이스와 TCP/IP 커넥션을 연결한다. (3 way handshake 같은 과정 발생)
3. DB 드라이버는 TCP/IP 커넥션이 연결되면 ID/PW 같은 부가 정보를 데이터베이스에 전달한다.
4. 이후 DB는 ID/PW를 통해 인증하고 세션을 생성한다.
5. DB는 커넥션 생성이 완료되었다는 응답을 보낸다.
6. DB 드라이버는 커넥션 객체를 생성해 클라이언트에 반환한다.

위와 같이 커넥션을 획득하기 위해 복잡한 과정을 거쳐야 한다. 그러면 사용자가 요청을 할 때마다 TCP/IP 연결을 하고 커넥션을 생성해야 하기 때문에 비효율적이다. 이런 문제를 해결하기 위해 커넥션 풀 이라는 것을 사용한다.

## 커넥션 풀
커넥션 풀은 위의 커넥션 객체들을 미리 생성해두고 사용하는 방법이다. 위와 같은 단계를 거치지 않고 커넥션을 획득할 수 있게 된다.
![](/img/DBCP.png)

애플리케이션 서버가 시작되면서 미리 정해진 커넥션 객체를 미리 만들어서 커넥션 풀에 저장했다가 클라이언트의 요청이 오면 Connection 을 빌려주고 작업이 끝나면 Connection 을 다시 반환받는 구조이다.

![](/img/DBCP2.png)

## HikariCP
위의 커넥션 풀 라이브러리의 구현체에는 Apache의 Common DBCP와 Tomcat-JDBC, BoneCP, HikariCP가 있다.
spring boot는 2.0 버전 이후에 HikariCP를 커넥션 풀의 default 구현체로 설정하였다. 

## 커넥션 풀의 유의 사항
애플리케이션의 동시 접속사 수가 많을 경우 커넥션 풀의 커넥션의 수는 한정이 되어 있기 때문에 사용자는 사용할 수 있는 커넥션이 반납될 때까지 기다려야 한다. 그렇다고 많은 커넥션을 생성하기에는 커넥션 또한 객체이기 떄문에 그만큼 메모리를 사용하게 되고, 프로그램의 성능이 떨어지게 된다.

Connection Pool의 크기를 어떻게 설정하는 것이 좋을까?
Connection의 주체는 Thread이기 때문에 Thread와 함께 생각해야 한다.

- Thread Pool의 크기 < Connection Pool
    - Thread Pool에서 트랜잭션을 처리하는 Thread가 사용하는 Connection외의 Connection은 메모리 공간만 차지하게 된다.
- Thread Pool과 Connection Pool 모두 증가
    - Thread의 개수 증가로 인해 Context Switching이 증가하게 된다.

데이터베이스 입장에서 Connection은 Thread와 어느 정도 일치한다. Connection이 많다는 것은 데이터베이스 서버가 Thread를 많이 사용한다는 것을 의미하고, 이에 따른 Context Switching이 많이 발생해 Connection Pool의 크기를 늘리더라도 성능적인 한계는 존재한다.


## 참고자료
[인프런 - 스프링 DB 1편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard)

[Connection Pool 이란?](https://steady-coding.tistory.com/564)