# SpringSecurity를 이용한 인증 및 권한 부여1

### Spring Security

- 인증 및 권한 부여를 통해 리소스 사용을 쉽게 컨트롤할 수 있는 프레임 워크
- Spring의 DispatcherServlet 앞단에 Filter를 등록시켜 요청을 가로챈 후 클라이언트에게 리소스 접근 권한이 없을 경우엔 인증 화면으로 리다이렉트 시킨다.

**SpringSecurity Filter**

- SpringSecurity는 기능별 필터의 집합으로 되어 있고 많은 필터가 있음.
- 기본적으로 리소스 요청시 접근 권한이 없는 경우 로그인 폼으로 보내는 역할을 하는 필터는  UsernamePasswordAuthenticationFilter

### ApI 인증 및 권한 부여, 제한된 리소스의 요청

- 인증을 위해 가입 및 로그인 api 구현
- 가입 시 제한된 리소스에 접근할 수 있는 ROLE_USER 권한을 회원에게 부여
- SpringSecurity 설정에는 접근 제한이 필요한 리소스에 대해서 ROLE_USER 권한을 가져야 접근 가능하도록 세팅.
- 권한을 가진 회원이 로그인 성공 시엔 리소스에 접근할 수 있는 Jwt 보안 토큰을 발급.
- Jwt 보안 토큰으로 회원은 권한이 필요한 api 리소스를 요청하여 사용.
- api 서버는 클라이언트에게 전달받은 JWT 토큰이 유효한지 확인하고 담겨있는 회원 정보를 확인하여 제한된 리소스를 제공하는데 이용할 수 있음.

**bundle.gradle에 springsecurity, jwt 라이브러리 추가**

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'io.jsonwebtoken:jjwt:0.9.1'
```

### JwtTokenProvider 생성

- Jwt 토큰 생성 및 유효성 검증을 하는 컴포넌트. (알고리즘과 비밀키로 토큰 생성)
- claim 정보에는 토큰에 부가적으로 실어 보낼 정보를 세팅할 수 있다.
- clain 정보에 회원을 구분할 수 있는 값을 세팅하였다가 토큰이 들어오면 해당 값으로 회원을 구분하여 리소스를 제공.
- expire 시간을 세팅하여 일정 시간 이후에는 토큰을 만료 시킴.

```java
package com.rest.api.config.security;

// import 생략

@RequiredArgsConstructor
@Component
public class JwtTokenProvider { // JWT 토큰을 생성 및 검증 모듈

    @Value("spring.jwt.secret")
    private String secretKey;

    private long tokenValidMilisecond = 1000L * 60 * 60; // 1시간만 토큰 유효

    private final UserDetailsService userDetailsService;

    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
    }

    // Jwt 토큰 생성
    public String createToken(String userPk, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(userPk);
        claims.put("roles", roles);
        Date now = new Date();
        return Jwts.builder()
                .setClaims(claims) // 데이터
                .setIssuedAt(now) // 토큰 발행일자
                .setExpiration(new Date(now.getTime() + tokenValidMilisecond)) // set Expire Time
                .signWith(SignatureAlgorithm.HS256, secretKey) // 암호화 알고리즘, secret값 세팅
                .compact();
    }

    // Jwt 토큰으로 인증 정보를 조회
    public Authentication getAuthentication(String token) {
        UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUserPk(token));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    // Jwt 토큰에서 회원 구별 정보 추출
    public String getUserPk(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    }

    // Request의 Header에서 token 파싱 : "X-AUTH-TOKEN: jwt토큰"
    public String resolveToken(HttpServletRequest req) {
        return req.getHeader("X-AUTH-TOKEN");
    }

    // Jwt 토큰의 유효성 + 만료일자 확인
    public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwtToken);
            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
}
```

JwtAuthenticationFilter

- Jwt가 유효한 토근인지 인증하기 위한 filter

```java
package com.rest.api.config.security;

// import 생략

public class JwtAuthenticationFilter extends GenericFilterBean {

    private JwtTokenProvider jwtTokenProvider;

    // Jwt Provier 주입
    public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    // Request로 들어오는 Jwt Token의 유효성을 검증(jwtTokenProvider.validateToken)하는 filter를 filterChain에 등록합니다.
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        String token = jwtTokenProvider.resolveToken((HttpServletRequest) request);
        if (token != null && jwtTokenProvider.validateToken(token)) {
            Authentication auth = jwtTokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(auth);
        }
        filterChain.doFilter(request, response);
    }
}
```

**SpringSecurityConfiguration**

- 서버에 보안 설정을 적용.

```java
package com.rest.api.config.security;

// import 생략

@RequiredArgsConstructor
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    private final JwtTokenProvider jwtTokenProvider;

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .httpBasic().disable() // rest api 이므로 기본설정 사용안함. 기본설정은 비인증시 로그인폼 화면으로 리다이렉트 된다.
            .csrf().disable() // rest api이므로 csrf 보안이 필요없으므로 disable처리.
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // jwt token으로 인증하므로 세션은 필요없으므로 생성안함.
            .and()
                .authorizeRequests() // 다음 리퀘스트에 대한 사용권한 체크
                    .antMatchers("/*/signin", "/*/signup").permitAll() // 가입 및 인증 주소는 누구나 접근가능
                    .antMatchers(HttpMethod.GET, "helloworld/**").permitAll() // hellowworld로 시작하는 GET요청 리소스는 누구나 접근가능
                    .anyRequest().hasRole("USER") // 그외 나머지 요청은 모두 인증된 회원만 접근 가능
            .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class); // jwt token 필터를 id/password 인증 필터 전에 넣는다

    }

    @Override // ignore check swagger resource
    public void configure(WebSecurity web) {
        web.ignoring().antMatchers("/v2/api-docs", "/swagger-resources/**",
                "/swagger-ui.html", "/webjars/**", "/swagger/**");

    }
}
```

**CustomUserDetailsService**

- 토큰에 세팅된 유저 정보로 회원 정보를 조회하는 UserDetailsService를 재정의

```java
@RequiredArgsConstructor
@Service
public class CustomUserDetailService implements UserDetailsService {

    private final UserJpaRepo userJpaRepo;

    public UserDetails loadUserByUsername(String userPk) {
        return userJpaRepo.findById(Long.valueOf(userPk)).orElseThrow(CUserNotFoundException::new);
    }
}
```

###