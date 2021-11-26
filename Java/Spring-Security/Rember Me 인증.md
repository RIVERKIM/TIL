# Rember Me 인증

### Remember Me 인증

- 세션이 만료되고 웹 브라우저가 종료된 후에도 어플리케이션이 사용자를 기억하는 기능
- Remember-Me 쿠키에 대한 Http 요청을 확인한 후 토큰 기반 인증을 사용해 유효성을 검사하고 토큰이 검증되면 사용자는 로그인 된다.
- 사용자 라이프 사이클
    - 인증 성공(Remember-Me 쿠키 설정)
    - 인증 실패(쿠키가 존재하면 쿠키 무효화)
    - 로그 아웃(쿠키가 존재하면 쿠키 무효화)

SecurityConfig.java

```java
@Autowired
UserDetailsService userDetailsService;

http
                .rememberMe()
                .rememberMeParameter("remember")
                .tokenValiditySeconds(3600)
                .userDetailsService(userDetailsService);
```

- rememberMeParameter("name"): 기본 파라미터 명은 remember-me
- tokenValiditySeconds(3600): 기본은 14일
- alwaysRememberMe(true): 리멤버 미 기능이 활성화되지 않아도 항상 실행
- userDetailsService(userDetailsService)

**기존 처리 과정**

- 로그인(인증 완료) → 사용자 세션이 생성됨., 세션은 사용자 인증 정보를 담고 있음
- 서버는 사용자에게 JSESSIONID를 응답 헤더에 실어서 보낸다.
- 클라이언트가 이후에 접근시 가져오는 JSESSIONID를 서버가 가져가서 이 아이디와 매칭 되는 세션을 가져오고, 그 세션안에 SecurityContext가 있고 그 안에 인증 객체가 있다.
- 그 정보를 가지고 인증된 사용자인지 처리.

**Remember me 설정 시 처리 과정**

- 인증 시, JSESSIONID 뿐만아니라 remember-me라는 쿠키 생성해서 발급해준다.
- JSESSION ID가 없어도  remember me가 있다면, 이 값을 추출해서 userId와 password를 얻어서 재인증 시도 → 인증 성공 다시 JSESSIONID 발급

### RememberMeAuthenticationFilter

![Untitled](Rember%20Me%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%207efd3d9dc7fd4ed19ac1bbfa54f687c0/Untitled.png)

- RememberMeAuthenticationFilter는 세션이 만료되었거나, 브라우저 종료시세션이 끊긴 경우 등 더이상 세션이 활성화 되지 않아서 SecurityContext안에서 사용자 세션을 찾기 못하는 경우에 사용자의 인증을 유지하기 위해 인증 시도하여 인증을 유지하도록 함.
- request Header에 remember-me 쿠키 값을 가지고 온 경우, 인증 객체가 없는 경우에 필터가 사용 된다.

'

![Untitled](Rember%20Me%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%207efd3d9dc7fd4ed19ac1bbfa54f687c0/Untitled%201.png)

- 처음 로그인 시도 시, AbstractAuthenticationProcessingFilter.java에서 시작 이후 위와 같은 과정을 처리.

### 참고: AnonymousAuthenticationFilter

![Untitled](Rember%20Me%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC%207efd3d9dc7fd4ed19ac1bbfa54f687c0/Untitled%202.png)

- 익명 사용자 인증 처리
- 익명 사용자와 인증 사용자를 구분해서 처리하기 위한 용도로 사용
- 화면에서 인증 여부를 구현할 때 isAnonymous()와 isAuthenticated로 구분해서 사용
- 인증 객체를 세션에 저장하지 않는다.

**처리 과정**

- 사용자 요청 시 AnonymousAuthenticationFilter가 요청을 받음.
- 요청한 사용자가 인증 객체(SecurityContext 안에)를 가지고 있는 지 여부를 판단
- 익명 사용자 일지라도 인증 객체를 생성해서 저장.
- 판단은 isAnonymous()등과 같이 Security Context안에 인증 객체가 익명 객체인지 판단해서 처리.