# 왜 스트링은 Java에서 불변인가?

## 불변객체란?

불변객체는 객체가 생성된 후 객체 내부의 상태가 일정하게 유지되는 객체를 말한다. 이는 객체가 변수에 할당되면, 우리는 어떤 방법으로도 객체에 대한 참조를 업데이트하거나 내부 상태를 변경할 수 없는 것을 말한다.<br>
또한 불변객체는 객체의 라이프사이클 동안 항상 같은 행위를 하는 것을 보장할 수 있다.

### final 키워드

자바에서 `final` 키워드를 사용해서 불변성을 만족시킬 수 있다.<br>
자바에서 변수들은 기본적으로 mutable하다. 이는 변수가 들고 있는 값을 변경하는 것이 가능하다는 것이다.

변수를 선언할 때 `final` 키워드를 사용하게 되면 Java 컴파일러는 변수의 값을 변경하는 것을 허용하지 않고, 컴파일 타임에 에러를 발생시킨다. 

이렇게 불변 객체가 가진 특성은 thread-safe 하며 side-effect에 대해 자유롭다.

## Java에서 String은 왜 불변인가?

`String` 클래스를 불변하게 두는데 있어서 이점은 caching, security, synchronization, performance 측면에 있다.

### String Pool
`String` literal 자체를 캐싱해두고 재사용하는 것은 Heap 메모리 공간을 절약할 수 있다. 이는 `String` 변수가 `String pool`에서 같은 객체를 참조하기 떄문이다.  

`String Pool`은 문자열이 JVM에 의해 저장되는 특별한 메모리 공간이다. `String`은 자바에서 불변하기 때문에, JVM은 각 리터럴 문자열의 복사본 하나만 풀에 저장하여 할당된 메모리 공간을 최적화한다.
<br>
이러한 과정을 `interning`이라고 부른다.

아래코드와 그림을 보자.

```java
@Test
void stringTest() {
    String s1 = "HelloWorld";
    String s2 = "HelloWorld";
    String s3 = new String("HelloWorld");

    assertThat(s1 == s2).isTrue();
    assertThat(s1 == s3).isFalse();
}
```

![출처 - baeldung](/img/String.png) [출처 - 밸덩]

s1을 literal로 생성후 s2도 동일한 literal로 생성하게 되면, JVM은 같은 값을 가지고 있는 String을 `String Pool`에서 찾게 된다. 만약 찾았다면 자바 컴파일러는 추가적인 메모리 할당 없이 해당 문자열의 메모리 주소를 반환하게 된다.<br>
만약 찾지 못했다면 pool에 추가되고 참조값이 반환되게 된다.

### new 연산자를 통한 String
우리가 new 연산자를 통해 String 변수를 만들게 되면 새로운 객체가 JVM의 힙 메모리에 저장되게 된다. <br>
이렇게 new 연산자를 통해 String 객체를 만들게 되면 항상 새로운 객체를 힙 메모리에 저장하게 된다.

> Java 9 부터의 String
>Java8까지는 String이 character 배열, UTF-16으로 인코딩되어 모든 character가 메모리의 2byte씩 사용하고 있었다. <br>
>Java9부터는 `Compact Strings`라는 것이 새롭게 나왔다.
>
>String을 1byte, LATIN-1로 저장 가능하면 1byte로 저장한다. 그 외의 경우는 UTF-16(2byte)로 저장한다.
>
>즉, 문자열을 만들 때마다 문자열의 모든 문자가 바이트 LATIN-1을 사용하여 표현될 수 있으면 바이트 배열이 내부적으로 사용되어 하나의 문자에 대해 1바이트가 주어진다.<br>
>그런데 어떤 문자가 8비트 이상을 요구한다면 모든 문자는 각각 UTF-16을 사용해서 2바이트로 저장된다.
>
>String 클래스를 확인해보면 대부분의 메서드가 LATIN을 체크하는 것을 확인할 수 있다.
>```java
>public int indexOf(int ch, int fromIndex) {
>    return isLatin1() ? StringLatin1.indexOf(value, ch, fromIndex)
>                        : StringUTF16.indexOf(value, ch, fromIndex);
>}
>```

### Security
String은 자바 애플리케이션에서 유저이름, 패스워드, url등 여러 곳에서 사용되고 있다. 다음의 코드 스니펫을 보자.
```java
void criticalMethod(String userName) {
    // perform security checks
    if (!isAlphaNumeric(userName)) {
        throw new SecurityException(); 
    }
	
    // do some secondary tasks
    initializeDatabase();
	
    // critical task
    connection.executeUpdate("UPDATE Customers SET Status = 'Active' " +
      " WHERE UserName = '" + userName + "'");
}
```
위 코드 스니펫에서 신뢰할 수 없는 곳에서 String (userName)을 받아 모든 로직을 처리하고 있다. 저 userName을 가지고 메서드를 호출한 곳은 아직 userName에 대한 참조를 가지고 있다. 즉, 만약 String이 mutable하다면 우리가 update 메서드를 실행할 때 보안 검사를 수행했다 하더라도 수신된 String이 안전한지 알 수 없다.

또한 String userName이 다른 스레드에 의해 보여질 수 있고 값이 변경될 수 있다.

하지만 `String`은 불변이기 때문에 이러한 걱정으로부터 자유롭다.

### Synchronized
불변하다는 것은 `String`을 thread-safe하게 만들어준다. 멀티스레드 환경에서도 값이 변하지 않는 것을 보장하기 때문이다. 

멀티스레드 환경에서 어떤 스레드가 공유하고 있는 값을 변경하면 공유되고 있는 값이 변경되는 것이 아니라, 새로운 `String`을 `String Pool`에 생성하기 때문에 멀티스레드 환경에서 안전하다고 할 수 있다.

### Hashcode Caching
`String`은 HashMap, HashTable, HashSet 등의 hash 구현체들에 의해 사용될 수 있다. 

불변성은 `String`이 그들의 값(value)이 변하지 않는 것을 보장한다. 그렇기 때문에 hashCode() 메서드는 캐싱을 가능하게 하기 위해 `String` 클래스에서 override되어 첫 번째 hashCode() 동안 해시가 계산되고 캐시되며 동일한 값을 반환한다.

```java
@Test
void hashTest() {
    String s1 = "Hello World";
    String s2 = new String("Hello World");
        
    assertThat(s1.hashCode()).isEqualTo(s2.hashCode()); // success
}
```

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        hash = h = isLatin1() ? StringLatin1.hashCode(value)
                              : StringUTF16.hashCode(value);
    }
    return h;
}
```
위 코드처럼 hash 값을 저장해 둔 뒤, 그 값을 계속해서 사용하고 있다.

이는 결과적으로 hash 구현체들이 `String` 객체를 가지고 연산을 수행할 때 성능적인 이점을 낼 수 있다.

### Performance
위에서 봤듯이, `String Pool`이 존재하기 때문에 힙 메모리를 절약하고, hash 연산을 `String` 객체를 가지고 할 때는 빠르게 접근할 수 있기 때문에 성능상 이점을 가진다.


## 정리
자바는 `String`을 불변 객체로서 디자인했다. <br>
이는 메서드의 파라미터로서 넘어온 `String` 객체의 변화에 대해 걱정하지 않아도 되고 멀티 스레드 환경에서 안전하게 사용될 수 있다는 것을 의미한다. <br>
또한 JVM의 힙 메모리에서 `String Pool`이라는 특별한 메모리 공간을 가지고 있는데, 동일한 literal로 생성된 `String`에 대해서는 `String Pool` 내부에 있는 동일한 literal에 대한 메모리 주소를 가지고 있어 힙 메모리 공간을 아낄 수 있다.

## 참고자료

[baeldung - java immutable object](https://www.baeldung.com/java-immutable-object)

[baeldung - java string pool](https://www.baeldung.com/java-string-pool)

[baeldung - why string is immutable](https://www.baeldung.com/java-string-immutable)