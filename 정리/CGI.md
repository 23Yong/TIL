# CGI

CGI(Common Gateway Interface)
- 웹 서버와 애플리케이션 사이에 데이터를 주고받는 규약
- CGI 규칙에 따라서 만들어진 프로그램을 CGI 프로그램이라 한다.
- CGI 프로그램 종류로는 컴파일 방식과 인터프리터 방식이 있다.

- 인터프리터 방식 CGI 프로그램
    - (PHP, Python)
    - 웹 서버 <-> Script Engine <-> Script 파일
    
- 서블릿과 서블릿 컨테이너
    - (C, C++, Java)
    - 웹 서버 <-> Servlet Container <-> Servlet 파일<br>
    
    > 서블릿<br>
    > - 자바에서 웹 애플리케이션을 만드는 기술<br>
    > - 자바에서 동적인 웹 페이지를 구현하기 위한 표준

    > 서블릿 컨테이너
    > - 서블릿의 라이프 사이클을 관리
    > - 웹 서버와 소켓을 만들고 통신하는 과정을 대신 처리
    > - 서블릿 객체를 싱글톤으로 관리
    >   - 싱글톤이기 때문에 상태를 저장하면 안됨 -> Race Condition

## WAS vs Servlet Container
- WAS는 서블릿 컨테이너를 포함하는 개념
- WAS는 매 요청마다 스레드 풀에서 스레드를 사용
- WAS의 주요 튜닝 포인트는 max thread 수
- ex. 톰캣


