# MySQL 아키텍처 (2) - InnoDB 스토리지 엔진 아키텍처

## 학습할 것
- MVCC
- 잠금 없는 일관된 읽기


## MVCC (Multi Version Concurrency Control)
DB는 Lock을 기본으로 해 데이터 파일에 작성된 데이터들의 일관성을 보장해주지만 이것은 쓰기가 많이 일어나는 서버에서의 동시성을 크게 떨어뜨린다. 
이런 문제를 해결하기 위해 나온 것이 MVCC인데, MVCC의 가장 큰 목적은 잠금을 사용하지 않은 일관된 읽기를 제공하는 것이다.

RDBMS에서 구현하는 MVCC는 크게 두 가지로 나눌 수 있다.
1. Pessimetic Lock을 이용하는 방식
2. Undo log를 이용하는 방식

이중 InnoDB는 Undo log를 통해 MVCC를 구현한다.

메모리에 언두 영역을 따로 두고 최신 데이터는 데이터 영역에, 올드 데이터는 언두 영역에 두면서 레코드에 대한 버전을 관리하는 방식이다. 

예를 들어 살펴보자.

우선 다음과 같은 테이블에 한 건의 레코드를 INSERT하고 UPDATE해서 발생하는 변경 작업 절차를 확인해보자.
```sql
create table member (
    m_id int not null,
    m_name varchar(20) not null,
    m_area varchar(100) not null,
    primary key (m_id),
    index ix_area (m_area)
);

insert into member (m_id, m_name, m_area) values (12, '홍길동', '서울');
commit;
```

INSERT 문이 실행되면 데이터베이스는 다음과 같은 상태가 된다.
![](/img/MySQL_MVCC1.png)

이제 UPDATE 문이 실행될때의 처리 절차를 보자.
```sql
update member set m_area = '경기' where m_id=12;
```
![](/img/MySQL_MVCC2.png)

- 언두 로그에는 m_area 열의 변경 전 값만 복사된다.
- UPDATE문장이 실행되면 커밋의 실행여부와 상관없이 InnoDB의 버퍼 풀이 업데이트 된다.
- 디스크의 데이터 파일에는 체크포인트나 Write 스레드에 의해 새로운 값으로 업데이트되어 있을 수도 있고 아닐 수도 있다. (일반적으로는 InnoDB의 버퍼풀과 동일한 상태라 봐도 무방하다.)

이제 COMMIT이나 ROLLBACK 되지 않은 상황에서 레코드를 조회하면 어디에 있는 데이터가 조회될까?

```sql
select * from member where m_id=12;
```

MySQL 서버의 시스템 변수와 설정된 격리 수준에 따라 다르다.

격리 수준이 READ_UNCOMMITED (커밋되지 않은 데이터도 읽어올 수 있다.) 경우 InnoDB 버퍼 풀이 가지고 있는 데이터를 읽어 반환한다.

격리 수준이 READ_COMMITED나 그 이상인 경우 아직 커밋되지 않았기 때문에 언두 영역에 있는 데이터를 반환한다.
이런 과정을 MVCC라고 한다.

## 잠금 없는 일관된 읽기
InnoDB 스토리지 엔진은 위의 MVCC(Multi Version Concurrency Control)를 이용해 잠금 없는 일관된 읽기를 수행할 수 있다. 

A라는 사용자가 한 레코드를 수정하고 있는 도중에 B라는 사용자가 해당 레코드를 순수하게 조회하려 할 때 트랜잭션의 변경 작업과 관계없이 언두 로그를 사용해서 잠금을 대기하지 않고 바로 실행할 수 있다.

## InnoDB 버퍼 풀
InnoDB 버퍼 풀은 디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시해 두는 공간이다. 버퍼라는 말은 InnoDB가 쓰기 작업을 지연시켜 일괄로 작업을 처리할 수 있게 해주는 버퍼 역할도 하기 때문이다.

### 버퍼 풀의 구조와 LRU 알고리즘
InnoDB 스토리지 엔진은 메모리 공간을 `page`단위로 나누어 관리하며, 한 페이지에 여러 row가 속할 수 있다.
버퍼 풀의 페이지들을 관리하기 위해서 InnoDB는 다음과 같은 자료 구조를 관리한다.

- LRU (Least Recently Used) 리스트
- 프리 (Free) 리스트
- 플러시 (Flush) 리스트

#### LRU 리스트
LRU 리스트는 다음과 같은 구조를 띄고 있다.
![](/img/MySQL_LRU.png)

LRU 리스트를 관리하는 목적은 디스크로부터 한 번 읽어온 페이지를 최대한 오랫동안 InnoDB에 유지해 디스크 읽기를 최소화 시키는 것이다.

기본적으로 LRU 알고리즘은 다음과 같은 절차를 따른다.
- 버퍼 풀의 3/8은 Old sublist에 사용된다.
- Midpoint는 Old sublist의 Head와 New sublist의 Tail이 만나는 지점이다.
- InnoDB가 버퍼 풀에 페이지를 넣을 때, midpoint에 삽입하게 된다. 
- Old sublist에 있는 페이지에 접근할 경우 그 페이지를 New sublist의 Head로 이동시켜 `young`하게 만든다. 
- 데이터 베이스의 작업이 계속되면서 버퍼 풀에 있는 데이터 페이지들 중 사용되지 않는 페이지들은 자연스럽게 전체 리스트의 Tail로 이동하게 된다. Old sublist의 페이지들은 midpoint에 새로운 페이지가 삽입됨에 따라서 Tail과 가까워지고, 결국 버퍼 풀에서 삭제 된다.

[참고 - MySQL LRU](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html)

#### 프리 리스트
프리 리스트는 InnoDB 버퍼 풀에서 실제 사용자 데이터로 채워지지 않은, 비어 있는 페이지들의 목록이며 사용자의 쿼리가 새롭게 디스크의 페이지를 읽어와야 하는 경우 사용된다.

#### 플러시 리스트
플러시 리스트는 디스크로 동기화 되지 않은, 데이터 변경이 가해진 페이지(더티 페이지)의 목록을 관리한다. 

