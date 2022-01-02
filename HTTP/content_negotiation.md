# 협상(Content negotiation)
HTTP에서 content negotiation이란 동일한 URI에서 리소스의 서로 다른 버전을 전달하기 위해 사용되는 메커니즘으로, 클라이언트가 선호하는 표현 요청이 무엇인지를 명시할 수 있다.

HTTP/1.1 표준은 서버 주도 협상을 시작하는 표준 헤더 목록들을 가지고 있다.
- Accept: 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어

협상 헤더는 요청시에만 사용하는 것을 기억해두자.

##  협상과 우선순위

### Quality Values(q)

```
GET /event
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
```
Quality values는 값의 우선순위를 매길 떄 사용한다. 0~1 사이의 값으로, 클수록 높은 우선순위를 가지고 있다.(생략시 1)
- Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
    1. ko-KR;q=1(q가 생략)
    2. ko;q=0.9
    3. en-US;q=0.8
    4. en;q=0.7

### 구체적인 것이 우선
```
GET /event
Accept: text/*, text/plain, text/plain;format=flowed, */*
```
Accept: text/*, text/plain, text/plain;format=flowed, */ *
1. text/plain;format=flowed
2. text/plain
3. text/*
4. */ *


## 참고자료

[모든 개발자를 위한 HTTP웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard)