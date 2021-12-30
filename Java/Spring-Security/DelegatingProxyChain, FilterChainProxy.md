# DelegatingProxyChain, FilterChainProxy

### DeligatingFilterProxy

![Untitled](DelegatingProxyChain,%20FilterChainProxy%204c146f94237245a5bf7345b98ba755e6/Untitled.png)

- 요청이 들어오면 Servlet에서 요청을 실행하기전 ServletFilter에서 작업 처리
- ServletFilter는 스프링에서 정의된 빈을 주입해서 사용할 수 없음.
- 특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임
    - springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청을 위임
    - 실제 보안처리를 하지 않음.
    - 즉 먼저 요청을 받아서 스프링에게 처리를 위임.

### FIlterChainProxy

![Untitled](DelegatingProxyChain,%20FilterChainProxy%204c146f94237245a5bf7345b98ba755e6/Untitled%201.png)

- SpringSecurityFilterChain의 이름으로 생성되는 필터 빈
- DelegatingFilterProxy으로 부터 요청을 위임 받고 실제 보안 처리
- 스프링 시큐리티 초기화 시 생성되는 필터들을 관리하고 제어
    - 스프링 시큐리티가 기본적으로 생성하는 필터
    - 설정 클래스에서 API 추가 시 생성되는 필터
- 사용자의 요청을 필터 순서대로 호출하여 전달
- 사용자정의 필터를 생성해서 기존의 필터 전.후로 추가 가능
    - 필터의 순서를 잘 정의
- 마지막 필터까지 인증 및 인가 예외가 발생하지 않으면 보안을 통과하고 그 때 Servlet 자원에 접근할 수 있다.

![Untitled](DelegatingProxyChain,%20FilterChainProxy%204c146f94237245a5bf7345b98ba755e6/Untitled%202.png)