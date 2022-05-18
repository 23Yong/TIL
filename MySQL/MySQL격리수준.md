# MySQL의 격리 수준

트랜잭션의 네 가지 성질인 ACID에서 격리 수준(Isolation Level)이란 여러 트랜잭션이 동시에 처리될 때 해당 트랜잭션이 얼마나 고립되어 있는가를 나타낸다.

격리 수준은 다음과 같이 크게 네 가지가 있다.
- READ UNCOMMITED
- READ COMMITED
- REPETABLE READ
- SERIALIZABLE

각 격리 수준에 따라 부정합의 문제가 있는데, 이 부정합의 문제는 격리 수준의 레벨에 따라 발생할 수도 있고 발생하지 않을 수ㄷ 있다.

|| **DIRTY READ** | **NON-REPEATABLE READ** | **PHANTOM READ** |
|-| -------------- | ----------------------- | ---------------- |
| READ UNCOMMITED | 발생 | 발생 | 발생 |
| READ COMMITED | 없음 | 발생 | 발생 |
| REPEATABLE READ | 없음 | 없음 | 발생 (InnoDB는 없음) |
| SERIALIZABLE | 없음 | 없음 | 없음 |

REPEATABLE READ 격리 수준에서 PHANTOM READ가 발생할 수 있지만 MySQL InnoDB의 독특한 특성 때문에 발생하지 않는다.

이제 각 격리 수준에 대해 알아보자.

## READ UNCOMMITED
READ UNCOMMITED의 격리 수준에서는 말 그대로 커밋 여부에 상관 없이 다른 트랜잭션에서 변경 내용을 확인할 수 있다.

![](/img/READ_UNCOMMITED.png)

위의 그림에서 사용자 A는 emp_no이 2이고 first_name이 Kim인 직원을 INSERT하고 커밋하지 않았다. 커밋 전에 사용자 B가 emp_no이 2인 직원을 조회할 때 INSERT한 직원의 정보를 커밋하지 않은 상태에서도 읽을 수 있다. 

여기서 문제는 사용자 A가 INSERT한 내용을 롤백 한다하더라도, 사용자 B는 여전히 "Kim"을 읽어 무언가를 처리할 수 있다는 것이다.

### DIRTY READ
이처럼 다른 트랜잭션에서 커밋하지 않은 내용을 읽을 수 있는 현상을 `DIRTY READ`라고 한다. DIRTY READ가 나타날 수 있는 격리 수준은 `READ COMMITED`이며 RDBMS 포준에서는 트랜잭션의 격리 수준으로 인정하지 않을 정도로 정합성에 많은 문제가 있는 격리 수준이다.

## READ COMMITED
READ COMMITED는 오라클 DBMS에서 기본적으로 사용되는 격리 수준이며, 온라인 서비스에서 가장 많이 사용되는 격리 수준이다.
![](/img/READ_COMMITED.png)

그림 5.4에서 사용자 A는 emp_no=2 인 직원의 first_name을 'Lee'로 변경했다. 이때 새로운 값인 'Lee'는 테이블에 즉시 기록되고 이전 값인 'Kim'은 언두 영역으로 백업된다. 사용자 A가 UPDATE를 커밋전 사용자 B가 emp_no=2인 데이터를 SELECT 할 때 테이블이 아닌 언두 영역에 백업된 레코드에서 가져온 것이다. 이는 `READ COMMITED` 격리 수준에서는 변경한 내용이 커밋되기 전까지 다른 트랜잭션에서 그러한 변경 내역을 조회할 수 없기 때문이다.

> InnoDB 스토리지 엔진은 트랜잭션과 격리 수준을 보장하기 위해 DML로 변경되기 이전 버전의 데이터를 별도로 백업하는데, 이 백업된 데이터를 언두 로그라 한다.

### NON-REPEATABLE READ
`READ COMMITED` 격리 수준에서도 `NON-REPEATABLE READ`라는 부정합의 문제가 있다. 
![](/img/NON-REPEATABLE_READ.png)

다음과 같이 사용자 B가 테이블에서 first_name='Lee'인 SELECT 쿼리를 날렸을 때 일치하는 결과가 없었다. 하지만 사용자 A가 트랜잭션을 시작하고 emp_no=2인 레코드의 first_name을 'Lee'로 변경하고 커밋을 실행한다. 이후 사용자 B가 이전과 같은 쿼리를 날렸을 때는 결과가 1건이 조회된다.

이는 사용자 B가 같은 트랜잭션 내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져와야 한다는 `REPEATABLE-READ` 정합성에 어긋난다.

이후 살펴볼 `REPEATABLE READ` 격리 수준에서는 기본적으로 `SELECT` 쿼리도 트랜잭션 범위 내에서만 작동한다. 즉, 같은 트랜잭션 안에서는 동일한 쿼리를 반복해서 실행해도 동일한 결과만 보게 된다.
이 차이로 데이터의 정합성이 깨지고 그로 인해 애플리케이션에 버그가 발생하면 찾아낼 수 있다.

## REPEATABLE READ
REPETABLE READ는 MySQL InnoDB 스토리지 엔진에서 기본으로 사용하고 있는 격리 수준이다. REPEATABLE READ는 MVCC(Multi Version Concurrency Control)을 위해 언두 영역에 백업된 이전 데이터를 이용해 동일 트랜잭션 내에서는 동일한 결과를 보여주는 것을 보장한다.
REPEATABLE READ와 READ COMMITED의 차이는 언두 영역에 백업된 레코드의 여러 버전 가운데 몇 번째 이전 버전까지 찾아 들어가야 하느냐에 달렸다.

모든 InnoDB의 트랜잭션은 고유한 트랜잭션 번호를 가지고, 언두 영역에 백업된 모든 레코드에는 변경을 발생시킨 트랜잭션의 번호가 포함돼 있다. 

> 언두 영역의 백업된 데이터는 InnoDB 스토리지 엔진이 불필요하다고 판단하는 시점에 주기적으로 삭제한다. `REPEATABLE` 격리 수준에서는 MVCC를 보장하기 위해 실행 중인 트랜잭션 가운데 가장 오래된 트랜잭션 번호보다 앞선 언두 영역의 데이터는 삭제할 수 없다. 그렇다고 가장 오래된 트랜잭션보다 앞선 번호의 트랜잭션에 의해 변경된 모든 언두 데이터가 필요한 것은 아니다. 특정 트랜잭션 번호 구간 내에서 백업된 언두 데이터가 보존되어야 한다. [RealMySQL 8.0 1권 中]

![](/img/REPEATABLE_READ.png)
사용자 A의 트랜잭션 번호는 12번이고 사용자 B의 트랜잭션 번호는 10이다. 이때 사용자 A가 'Kim' 사원을 'Lee'로 변경을 수행하고 커밋했다. 사용자 B는 A의 작업 전 후로 SELECT 쿼리를 날려 결과는 항상 'Kim'이라는 결과를 가져온다. 사용자 B가 10번 트랜잭션 번호를 부여받고, 10번 트랜잭션 안에서 실행되는 SELECT 쿼리는 10보다 작은 트랜잭션에서 변경한 것만 보이게 된다.

### PHANTOM READ
`REPEATABLE READ`에서도 다음과 같은 부정합이 발생할 수 있다.
![](/img/PHANTOM_READ.png)

사용자 B는 BEGIN 명령어로 트랜잭션을 시작하고 SELECT를 수행한다. 위의 `REPEATABLE READ`에 따르면 두 번의 쿼리에 의해 얻은 결과는 같아야 한다. 하지만 위를 보면 두 SELECT 쿼리의 결과가 다르다. 이렇게 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안 보였다하는 현상을 `PHANTOM READ` 라고 한다. 

`SELECT ... FOR UPDATE` 를 통해 레코드에 쓰기 잠금을 걸어야 하는데, 언두 레코드에는 잠금을 걸 수 없다. 따라서 `SELECT ... FOR UPDATE`나 `SELECT ... LOCK IN SHARE MODE`로 조회되는 레코드는 언두 영역의 변경 전 데이터가 아닌 현재 레코드의 값을 가져오는 것이다.

## SERIALIZABLE
가장 단순한 격리 수준이면서 가장 엄격한 격리 수준이다. 기본적으로 InnoDB에서 순수한 SELECT 작업은 아무런 레코드 잠금을 설정하지 않고 실행된다. 하지만 `SERIALIZABLE`로 설정되면 읽기도 공유 잠금을 획득해야 하고, 동시에 다른 트랜잭션은 레코드를 변경하지 못한다. 

## MySQL의 InnoDB와 PHANTOM READ
처음에 `REPEATABLE READ` 격리 수준에서 `PHANTOM READ`가 발생하지 않는다고 했다. 이는 InnoDB의 갭 락과 넥스트 키락 덕분에 `REPEATABLE READ` 격리 수준에서 이미 `PHANTOM READ`가 발생하지 않기 때문이다.

> - 갭 락(Gap lock)
>   - 레코드 자체에 락을 거는 것이 아닌, 레코드와 바로 인접한 레코드 사이의 간격을 잠그는 것을 의미한다.
> - 넥스트 키 락(Next Key Lock)
>   - 레코드 락과 갭 락을 합쳐 놓은 락
>   - 레코드도 잠그고 레코드 직전, 직후 레코드의 간격도 잠근다.

## 참고 자료
[Real MySQL 8.0 1권](http://www.yes24.com/Product/Goods/103415627)