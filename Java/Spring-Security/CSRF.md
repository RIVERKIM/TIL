# 사이트 간 요청 위조- CSRF

### Form 인증 - CSRF(사이트 간 요청 위조)

- 웹 어플리케이션 취약점 중 하나로 인터넷 사용자(희생자)가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 만드는 공격.

![Untitled](%E1%84%89%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%90%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%AD%E1%84%8E%E1%85%A5%E1%86%BC%20%E1%84%8B%E1%85%B1%E1%84%8C%E1%85%A9-%20CSRF%20534d74270b384fea8f23f286eb709d59/Untitled.png)

### CsrfFilter

- 모든 요청에 랜덤하게 생성된 토큰을 HTTP 파라미터로 요구
- 요청 시 전달되는 토큰 값과 서버에 저장된 실제 값과 비교한 후 만약 일치하지 않으면 요청은 실패.
- Client
    - <input type=”hidden” name = “${csrf.parameterName}” value = “${_csrf.token}”/>
    - HTTP 메소드 : PATCH, POST, PUT, DELETE
- Spring Security
    - http.csrf(): default가 활성화
    - http.csrf().disabled()