# JPA join types

JPA의 조인 종류를 알아보자.

일단 사용할 샘플 데이터모델들이다.

```java
@Entity
@Getter @Setter
public class Employee {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private int age;

    @ManyToOne
    private Department department;

    @OneToMany(mappedBy = "employee")
    private List<Phone> phones;
}
```

```java
@Entity
@Getter @Setter
public class Department {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

```java
@Entity
@Getter @Setter
public class Phone {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String number;

    @ManyToOne
    private Employee employee;
}
```

---

## Inner Join

inner join은 둘 이상의 엔티티가 inner join될 경우 조인 조건과 일치하는 레코드만 결과에 수집된다.
![](/img/innerjoin.png)

### 단일 값 연관경로 탐색 : implicit inner join
Inner join은 암시적으로 이루어진다. 이름에서 알 수 있듯이 개발자는 따로 inner join을 지정하지 않는다.
우리가 연관탐색을 이용한 단일 값 조회를 할 때마다 JPA는 자동으로 암시적 조인을 만든다.

다음을 보자.
```java
@Test
public void innerJoinImplicitTest() throws Exception {
    List<Department> resultList = em.createQuery(
            "select e.department from Employee e", Department.class)
            .getResultList();

    // Assertions..
}
```
다음과 같이 실행했을 때 나가는 쿼리문은 다음과 같다.
```SQL
select
        department1_.id as id1_0_,
        department1_.name as name2_0_ 
    from
        employee employee0_ 
    inner join
        department department1_ 
            on employee0_.department_id=department1_.id
```
Employee 엔티티는 Department와 다대일(@ManyToOne)관계를 맺고 있는데 `e.department`를 이용해 Employee 엔티티에서 Department로 이동하는 경우 단일 값 연결을 탐색하게 된다. 결과적으로 JPA는 inner join을 생성하게 된다.

### 단일 값 연관 경로 탐색 : explicit inner join
명시적으로 `join` 키워드를 사용해 짠 JPQL 쿼리문을 보자.
```java
@Test
public void innerJoinExplicitTest() throws Exception {
    List<Department> resultList = em.createQuery(
            "select d from Employee e join e.department d", Department.class)
            .getResultList();
}
```
이때 나가는 쿼리는 다음과 같다.
```SQL
select
        department1_.id as id1_0_,
        department1_.name as name2_0_ 
    from
        employee employee0_ 
    inner join
        department department1_ 
            on employee0_.department_id=department1_.id
```
이번엔 from 절에서 `join` 키워드를 명시적으로 적어 실행하였다. 결과는 위와 같다는 것을 확인할 수 있고 JPA는 join 키워드를 사용하지 않으면 자동적으로 inner join을 사용한다는 것을 확인할 수 있다.

### 컬렉션 값 연관 경로 탐색
Employee 엔티티를 보면 일대다(@OneToMany)관계를 가지고 있는 phones를 찾을 수 있다. 
Phone엔티티를 얻기 위해 다음과 같은 쿼리를 작성할 수도 있을 것이다.
```SQL
SELECT e.phones FROM Employee e
```
하지만 이는 생각대로 Phone의 엔티티를 얻어오지 못한다. 대신 컬렉션 형태인 Phone의 List를 얻어온다.
다음과 같은 코드로 보자.
```java
@Test
public void innerJoinExplictCollectionTest() throws Exception {
    List<Collection> resultList = em.createQuery(
                    "select e.phones from Employee e", Collection.class)
            .getResultList();
}
```
실행되는 쿼리는 다음과 같다.
```SQL
select
        phones1_.id as id1_2_,
        phones1_.employee_id as employee3_2_,
        phones1_.number as number2_2_ 
    from
        employee employee0_ 
    inner join
        phone phones1_ 
            on employee0_.id=phones1_.employee_id
```
그리고 우리가 특정 행을 얻기위해 where절을 사용할 수 있는데, JPA는 여기서 이를 허용하지 않는다.
e.phones.number와 같이 객체 그래프 탐색이 컬렉션에서 허용되지 않기 때문이다.
대신 명시적으로 inner join을 설정하고 Phone에 대한 alias를 만들어야 한다. 다음 코드를 보자.

```java
@Test
public void innerJoinExplicitCollctionWhereTest() throws Exception {
    em.createQuery(
            "select p from Employee e" +
            " join e.phones p" +
            " where p.number LIKE '1%'", Phone.class).getResultList();
}
```

## Outer Join
두 개이상의 엔티티를 outer-join할 때 행은 왼쪽 엔티티의 레코드 뿐만 아니라 join 조건을 만족하는 레코드도 결과로 나온다.

다음 코드를 보자.
```java
@Test
public void outerJoinTest() throws Exception {
    List<Department> resultList = em.createQuery(
                    "select distinct d from Department d left join d.employees e", Department.class)
            .getResultList();
}
```
실행되는 쿼리는 다음과 같다.
```SQL
select
        distinct department0_.id as id1_0_,
        department0_.name as name2_0_ 
    from
        department department0_ 
    left outer join
        employee employees1_ 
            on department0_.id=employees1_.department_id
```
결과는 Employee와 연관관계를 가지고 있는 Department뿐만 아니라 가지고 있지 않는 Department가 출력될 것이다. 

## Fetch Join
fetch join은 SQL에서 이야기 하는 조인의 종류는 아니고 JPQL에서 성능 최적화를 하기 위해 지공하는 기능이다. 이것은 연관된 엔티티나 컬렉션을 한 번에 조회하는 기능인데 `join fetch`명령어로 사용할 수 있다. 주로 lazy-loading에서 사용한다.
```java
@Test
public void fetchJoinTest() throws Exception {
    List<Department> resultList = em.createQuery(
            "select d from Department d join fetch d.employees", Department.class)
            .getResultList();
}
```
다음 코드는 위에서 했던 코드들과 비슷해 보이지만 차이점이 존재한다. 바로 Employee 가 eager-loading된다는 것이다. 실행되는 쿼리문을 살펴보자.
```SQL
select
        department0_.id as id1_0_0_,
        employees1_.id as id1_1_1_,
        department0_.name as name2_0_0_,
        employees1_.age as age2_1_1_,
        employees1_.department_id as departme4_1_1_,
        employees1_.name as name3_1_1_,
        employees1_.department_id as departme4_1_0__,
        employees1_.id as id1_1_0__ 
    from
        department department0_ 
    inner join
        employee employees1_ 
            on department0_.id=employees1_.department_id
```
하지만 trade-off가 발생하는 것을 반드시 알아야 한다. 한 번의 쿼리를 통해 데이터를 얻어 효율적으로 보일 수 있지만, 모든 Department와 그 Employee까지 메모리에 한 번에 로드 된다는 것을 알아야 한다.

## 참고 자료
[밸덩 JPA join types](https://www.baeldung.com/jpa-join-types)