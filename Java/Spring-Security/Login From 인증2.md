# Login From 인증2

### Login Form 인증 과정

![Untitled](Login%20From%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC2%2049eedfc30c9f474cb89f692144364f7a/Untitled.png)

- Authentication 객체

**UsernamePasswordAuthenticationFilter 역할**

![Untitled](Login%20From%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC2%2049eedfc30c9f474cb89f692144364f7a/Untitled%201.png)

- [FilterChainProxy.java](http://FilterChainProxy.java): 필터들을 관리하는 빈이다.
- 각각의 필터들은 설정 클래스 + 기본 으로 필터들이 생성된다.
- 사용자 요청은 각각의 필터를 거치면서 처리하게 된다.

### 인증 API - Logout

![Untitled](Login%20From%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC2%2049eedfc30c9f474cb89f692144364f7a/Untitled%202.png)

**SecurityConfig.java**

```java
http
                .logout() //원칙적으로 post방식으로 처리
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login")
                .addLogoutHandler(new LogoutHandler() {
                    @Override
                    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
                        HttpSession session = request.getSession();
                        session.invalidate();
                    }
                })
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        response.sendRedirect("/login");
                    }
                })
                .deleteCookies("remember me");
```

- logout(): 로그아웃 처리
- logoutUrl("/logout"): 로그아웃 처리 URL
- logoutSuccessUrl: 로그아웃 성공 후 이동페이지
- deleteCookies("JSESSIONID", "remember-me") : 로그아웃 후 쿠키 삭제
- addLogoutHandler(new LogoutHandler()):  로그아웃 핸들러
- logoutSuccessHandler(new LogoutSuccessHandler): 로그아웃 성공 후 핸들러

![Untitled](Login%20From%20%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%8C%E1%85%B3%E1%86%BC2%2049eedfc30c9f474cb89f692144364f7a/Untitled%203.png)

- [LogoutFilter.java](http://LogoutFilter.java) 파일에는 실제 이 과정을 수행하는 것들을 알 수 있음.