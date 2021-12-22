# JVM

## JVM, JDK, JRE의 차이
---


### JVM(Java Virtual Machine)
자바 바이트 코드를 어떻게 실행하는지에 대한 표준 스펙이라고 생각할 수 있다. 자바의 가상 머신으로 자바 바이트 코드(.class)를 OS에 특화된 코드로 변환하여 실행하는 것에 대한 표준이고 구현체이다. 이렇기 때문에 OS에 의존적이라 할 수 있고, 특정 플랫폼에 종속적이라고 할 수 있다. 그리고 이러한 JVM을 구현하는게 여러 벤더(Oracle, 아마존, Azul...)에 따라 다   르다. \
또 JVM은 Java만의 것이 아닌 것이 되었다. Kotlin, JRuby, Jython, Scala... 등등이 사용할 수 있다.


### JRE(Java Runtime Environment)
JVM과 라이브러리를 더한 것이라고 생각할 수 있다. \
위의 JVM은 혼자서 제공되지 않고 최소한의 배포단위가 JRE이다. JRE의 배포판의 목적은 자바 애플리케이션을 실행하는 것에 목적을 두고 있다. 그렇기 때문에 핵십 라이브러리와 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나 리소스 파일을 가지고 있다. 목적이 이렇기 때문에 자바를 개발하는데 관련된 툴들은 포함하고 있지 않다.


### JDK(Java Development Kit)
JRE와 개발에 필요한 툴들을 더한 것이다. 오라클은 Java11 부터 따로 JRE를 제공하지 않는다고 한다..

#### 바이트코드란 :dart: 
바이트코드는 JVM이 언어들을 컴파일한 결과로 나오는 코드로 JVM이 기계로 읽어서 번역할 수 있는 코드이다. 

## JVM의 구조
---
JVM은 클래스 로더 시스템, 런타임 데이터 영역, 실행엔진, JNI, 네이티브 메서드 라이브러리로 구분할 수 있다.

### 클래스 로더 시스템
.class 파일에서 바이트 코드를 읽고 메모리에 저장한다. 
클래스로더는 계층 구조로 이루어져 있으며, 세 가지 클래스 로더가 제공된다.
- BootStrap ClassLoader
    - JAVA_HOME\lib에 있는 코어 자바 API를 제공한다. 최상위 우선순위를 가진 클래스 로더이다.
- Extension ClassLoader
    - JAVA_HOME\lib\ext 폴더 또는 java.ext.dirs 시스템 변수에 해당하는 위치에 있는 클래스를 읽는다.
- Application ClassLoader
    - 애플리케이션 class path(애플리케이션을 실행할 때 주는 -classpath 옵션 혹은 java.class.path 환경변수의 값에 해당하는 위치)에서 클래스를 읽는다.
    
클래스 로더는 다음과 같은 3가지 동작으로 진행된다.
- 로딩 : 클래스를 읽어오는 과정
- 링크 : 레퍼런스를 연결하는 과정
- 초기화 : static 값 초기화 및 변수에 할당

#### 로딩
클래스로더가 .class 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만든 후 '메서드' 영역에 저장한다. 이때 메서드 영역에 저장하는 데이터는 FCQN(Full Qualified Class Name), 클래스/인터페이스/이늄, 메서드와 변수이다.
로딩이 끝나면 해당 클래스 타입의 Class 객체를 생성하여 heap 영역에 저장한다.

#### 링크
verify, prepare, resolve 세 단계로 이루어져 있다.
- verify : 바이트코드 검증기가 생성된 바이트코드가 제대로 되어있는지 검증한다. 만약 검증에 실패하면 검증 에러를 낸다.
- prepare : 모든 static 변수들의 메모리가 할당되고 default 값으로 초기화 한다.
- resolve : 모든 심볼릭 메모리 참조가 메서드 영역의 원래 참조로 대체된다.

#### 초기화
static 변수의 값을 할당한다.

### 런타임 데이터 영역
- 스택(Stack)
    - Thread 마다 런타임 스택을 만들고, 그 안에 메소드 호출을 스택 프레임이라 부르는 블럭으로 쌓는다. Thread를 종료하면 런타임 스택도 사라진다.
- PC(PC registers)
    - Thread 마다 Thread 내의 현재 실행할 instruction의 위치를 가리키는 포인터를 가진다.
- 네이티브 메서드 스택(Native method stack)
    - 네이티브 메서드 정보를 가지고 있다. 모든 Thread 마다 네이티브 메서드 스택을 가지고 있다.
- 힙(heap)
    - 객체를 저장하고 공유하는 영역이다.
- 메서드(method)
    - 클래스 수준의 정보 (클래스 이름, 부모 클래스 이름, 메서드, 변수)를 저장, 공유하는 자원이다.

### 실행엔진
런타임 데이터 영역에 할당된 바이트코드를 읽어서 기계어로 번역해 실행하는 영역이다.
실행엔진은 다음과 같은 세 가지 요소를 가지고 있다.
- 인터프리터 : 바이트코드를 한 라인씩 읽어서 실행
- JIT 컴파일러 : 인터프리터의 단점을 보완. 인터프리터의 효율을 높이기 위해, 인터프리터가 반복되는 코드를 발견하면 JIT 컴파일러로 반복되는 코드를 모두 네이티브 코드로 바꿔준다. 그 다음부터 인터프리터는 네이티브 코드로 컴파일된 코드를 바로 사용한다.
- GC(Garbage Collector) : 더 이상 참조되지 않는 객체들을 모아서 정리한다.

### JNI(Java Native Interface)
자바 어플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법을 제공한다. Native 키워드를 사용해서 메서드를 호출한다.

### 네이티브 메서드 라이브러리
- C, C++로 작성된 라이브러리

## 참고자료
---
[더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation/dashboard)

[DZone JVM Architecture Explain](https://dzone.com/articles/jvm-architecture-explained)

[JVM 스펙](https://docs.oracle.com/javase/specs/jvms/se11/html/)
