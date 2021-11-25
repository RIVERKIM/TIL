# Login Form 인증1

### 사용자 정의 보안 기능 구현

![Untitled](Login%20Form%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC1%2046a0c440e89d4dd4a9d0960cea514285/Untitled.png)

- **WebSecurityConfigurerAdapter**
    - 스프링 시큐리티의 웹 보안 기능 초기화 및 설정
- **HttpSecurity**
    - 세부적인 보안 기능을 설정할 수 있는 API 제공

**Spring Security 기본 계정 설정**

```java
//application.yml
spring:
	security:
	    user:
	      name: user
	      password: 1111
```

**SecurityConfig 설정 - 사용자 세부적인 보안 기능 설정**

```java
@Configuration
@EnableWebSecurity // web 보안 활성화
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated(); // 인가 방식
        http
                .formLogin(); // 인증 방식
    }
}
```

### Form 인증

![Untitled](Login%20Form%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC1%2046a0c440e89d4dd4a9d0960cea514285/Untitled%201.png)

- HTTP는 자체적인 인증 관련 기능을 제공하며 HTTP 표준에 정의된 가장 단순한 인증 기법이다.
- 간단한 설정과 Stateless가 장점 - Session Cookie 사용하지 않음
- 보호차원 접근시 서버가 클라이언트에게 401 응답과 함께 WWW-Authenticate header를 기술해서 인증요구를 보냄
- Client는 ID:Password 값을 Base64로 Encoding한 문자열을 Authorization Header에 추가한 뒤 Server에게 Resource를 요청
- ID, Password가 Base64로 Encoding되어 있어 , ID, Password가 외부에 쉽게 노출되는 구조이기 때문에 SSL이나 TLS는 필수.

**Http Basic 인증**

```java
protected void configure(HttpSecurity http) throws Exception {
		http.httpBasic();
}
```

**BasicAuthenticationFilter**

![Untitled](Login%20Form%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC1%2046a0c440e89d4dd4a9d0960cea514285/Untitled%202.png)

**Form Login 인증 custom 코드**

- SecurityConfig.java

```java
@Configuration
@EnableWebSecurity // web 보안 활성화
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated(); // 인가 방식
        http
                .formLogin() // 인증 방식
                //.loginPage("/loginPage") // 사용자 정의 로그인 페이지
                .defaultSuccessUrl("/") // 로그인 성공 후 이동 페이지
                .failureUrl("/login") // 로그인 실패 후 이동 페이지
                .usernameParameter("userId") // 아이디 파라미터명 설정
                .passwordParameter("passwd") // 패스워드 파라미터명 설정
                .loginProcessingUrl("/login_proc") // 로그인 Form Action Uril
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override // 로그인 성공 후 핸들러
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        System.out.println("authentication = " + authentication.getName());
                        response.sendRedirect("/");
                    }
                })
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override // 로그인 실패 후 핸들러
                    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
                        System.out.println("exception = " + exception.getMessage());
                        response.sendRedirect("/login");
                    }
                })
                .permitAll(); // 로그인 페이지는 누구에게나 허용하도록.
    }
}
```