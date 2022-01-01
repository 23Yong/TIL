# HTTP헤더

HTTP헤더는 HTTP 전송에 필요한 모든 부가정보를 담고있다. (ex. 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보...)
HTTP헤더는 대소문자를 구분하지 않는 이름과 콜론 ':', 선택적 앞의 공백(OWS), 필드 값, 선택적 뒤의 공백(OWS)으로 이루어져있다.

## HTTP헤더 - RFC2616(과거)
### 헤더 분류
- General 헤더: 메시지 전체에 적용되는 정보
- Request 헤더: 요청 정보
- Response 헤더: 응답 정보
- Entity 헤더: 엔티티 바디 정보

### HTTP BODY
![](/img/HTTP헤더.png)
- 메시지 본문은 엔티티 본문을 전달하는데 사용한다.
- 엔티티 본문은 요청이나 응답에서 실제 전달한 데이터를 가리킨다.
- 엔티티 헤더는 엔티티 본문의 데이터를 해석할 수 있는 정보를 제공한다.

## HTTP헤더 - RFC7230
과거 RFC2616은 사라지고 2014년부터 RFC7230~7235가 등장했다.
RFC2616에서 RFC723x로의 변화는 
- 엔티티(Entity) -> 표현(Representation)
- Representation = Representation Metadata + Representation Data
    - 표현 = 표현 메타데이터 + 표현 데이터

### HTTP BODY
![](/img/HTTP-RFC7230.png)
- 메시지 본문을 통해 표현 데이터를 전달한다.
- 표현은 요청이나 응답에서 전달할 실제 데이터를 가리킨다.
- 표현 헤더는 표현 데이터를 해석할 수 있는 정보를 제공한다.
- 참고: 표현 헤더는 표현 메타데이터와, 페이로드 메시지를 구분해야한다. (여기서는 생략)

### 표현
표현은 특정 리소스의 다른 형식이다. 예를 들어, 동일한 데이터가 XML혹은 JSON과 같은 형식으로 지정되고, 특정 언어로 현지화 되거나 전송을 위해 압축되어 인코딩 될 수 있다. 특정 리소스는 동일하지만 표현이 다르다는 것이다.

HTTP의 목적에서 표현은 주어진 자원의 과거, 현재 또는 원하는 상태를 반영하기 위한 정보이며, 프로토콜을 통해 쉽게 통신할 수 있는 형식으로 표현 메타데이터의 집합과 표현 데이터의 스트림으로 구성된다.

- Content-Type: 표현 데이터의 형식
- Content-Encoding: 표현 데이터의 압축방식
- Content-Language: 표현 데이터의 자연 언어
- Content-Length: 표현 데이터의 길이

### Content-Type
표현 데이터의 형식 설명
```m
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3241

<html>
  <body>...<body>
</html>
```
```m
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 16

{"data":"hello"}
```

- 미디어 타입, 문자 인코딩

### Content-Encoding
표현데이터 인코딩
- 표현 데이터를 압축하기 위해 사용한다.
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더가 추가된다.
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제한다.

### Content-Language
표현 데이터의 자연 언어
- 표현 데이터의 자연 언어를 표현한다.
- ex.
    - ko
    - en
    - en-US

### Content-Length
표현 데이터의 길이
- 바이트 단위이다.
- Transfer-Encoding을 사용하면 Content-Length를 사용하면 안된다.


## 참고자료
[모든 개발자를 위한 HTTP웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard)

[RFC7231 스펙 - representation](https://datatracker.ietf.org/doc/html/rfc7231#section-3)