# SpringSecurity를 이용한 인증 및 권한 부여2

### 예외 처리 보안

1. Jwt 토큰 없이 api 호출 하는 경우
2. 형식에 맞지 않거나 만료된 Jwt토큰으로 api를 호출한 경우
3. Jwt 토큰으로 Api를 호출하였으나 해당 리소스에 대한 권한이 없는 경우

**커스텀 예외 처리가 적용이 안되는 경우**

- 커스텀으로 적용한 예외 처리가 적용이 안되는 이유는 필터링 순서 때문이다.
- 커스텀 예외처리는 ControllerAdvice 즉 Spring이 처리 가능한 영역까지 request가 도달해야 처리할 수 있었다.
- SpringSecurity는 Spring 앞단에서 필터링을 하기 때문에, 해당 상황의 exception이 spring의 dispatcherServlet까지 오지 않음.

**1, 2 번 해결책**

- 온전한 jwt 전달이 안될 경우는 토큰 인증 처리 자체가 불가능하기에, 해당 예외를 잡아내려면 SpringSecurity에서 제공하는 AuthenticationEntryPoint를 상속받아 재정의 해야 한다.
- 예외가 발생한 경우 예외 처리를 위해 포워딩 필요.

**CustomAuthenticationEntryPoint**

```java
package com.rest.api.config.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException ex) throws IOException,
            ServletException {
        response.sendRedirect("/exception/entrypoint");
    }
}
```

**ExceptionController**

```java
package com.rest.api.controller.exception;

// import 생략

@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/exception")
public class ExceptionController {

    @GetMapping(value = "/entrypoint")
    public CommonResult entrypointException() {
        throw new CAuthenticationEntryPointException();
    }
}
```

**ExceptionAdvice**

```java
@ExceptionHandler(CAuthenticationEntryPointException.class)
public CommonResult authenticationEntryPointException(HttpServletRequest request, CAuthenticationEntryPointException e) {
        return responseService.getFailResult(Integer.valueOf(getMessage("entryPointException.code")), getMessage("entryPointException.msg"));
}
```

**SpringSecurityConfiguration**

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
        http
            .httpBasic().disable() // rest api 이므로 기본설정 사용안함. 기본설정은 비인증시 로그인폼 화면으로 리다이렉트 된다.
            .csrf().disable() // rest api이므로 csrf 보안이 필요없으므로 disable처리.
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // jwt token으로 인증할것이므로 세션필요없으므로 생성안함.
            .and()
                .authorizeRequests() // 다음 리퀘스트에 대한 사용권한 체크
                    .antMatchers("/*/signin", "/*/signup").permitAll() // 가입 및 인증 주소는 누구나 접근가능
                    .antMatchers(HttpMethod.GET, "/exception/**", "helloworld/**").permitAll() // hellowworld로 시작하는 GET요청 리소스는 누구나 접근가능
                .anyRequest().hasRole("USER") // 그외 나머지 요청은 모두 인증된 회원만 접근 가능
            .and()
                .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())
            .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class); // jwt token 필터를 id/password 인증 필터 전에 넣어라.
}
```

### 3번 해결책

- SpringSecurity에서 제공하는  AccessDeniedHandler를 상속받아 커스터 마이징 해야 한다.
- 예외가 발생할 경우 handler에서는 /exception/accessdenied로 포워딩 하도록 해야한다.

**CustomAccessDeniedHandler**

```java
package com.rest.api.config.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
	@Override
	public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException exception) throws IOException,
            ServletException {
		response.sendRedirect("/exception/accessdenied");
	}
}
```

**ExceptionController**

```java
@GetMapping(value = "/accessdenied")
public CommonResult accessdeniedException() {
        throw new AccessDeniedException("");
}
```

**ExceptionAdvice**

```java
@ExceptionHandler(AccessDeniedException.class)
public CommonResult AccessDeniedException(HttpServletRequest request, AccessDeniedException e) {
        return responseService.getFailResult(Integer.valueOf(getMessage("accessDenied.code")), getMessage("accessDenied.msg"));
}
```

**SpringSecurityConfiguration**

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
        http
            .httpBasic().disable() // rest api 이므로 기본설정 사용안함. 기본설정은 비인증시 로그인폼 화면으로 리다이렉트 된다.
            .csrf().disable() // rest api이므로 csrf 보안이 필요없으므로 disable처리.
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // jwt token으로 인증할것이므로 세션필요없으므로 생성안함.
            .and()
                .authorizeRequests() // 다음 리퀘스트에 대한 사용권한 체크
                    .antMatchers("/*/signin", "/*/signup").permitAll() // 가입 및 인증 주소는 누구나 접근가능
                    .antMatchers(HttpMethod.GET, "/exception/**", "helloworld/**").permitAll() // hellowworld로 시작하는 GET요청 리소스는 누구나 접근가능
                .antMatchers("/*/users").hasRole("ADMIN")
                .anyRequest().hasRole("USER") // 그외 나머지 요청은 모두 인증된 회원만 접근 가능
            .and()
                .exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler())
            .and()
                .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())
            .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class); // jwt token 필터를 id/password 인증 필터 전에 넣어라.
}
```

> 참고:  SpringSecurity annotation 권한 설정.
> 

### @PreAuthorize, @Secured

```java
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
@RequiredArgsConstructor
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
생략...
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .httpBasic().disable() // rest api 이므로 기본설정 사용안함. 기본설정은 비인증시 로그인폼 화면으로 리다이렉트 된다.
            .csrf().disable() // rest api이므로 csrf 보안이 필요없으므로 disable처리.
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // jwt token으로 인증할것이므로 세션필요없으므로 생성안함.
//            .and()
//                .authorizeRequests() // 다음 리퀘스트에 대한 사용권한 체크
//                    .antMatchers("/*/signin", "/*/signin/**", "/*/signup", "/*/signup/**", "/social/**").permitAll() // 가입 및 인증 주소는 누구나 접근가능
//                    .antMatchers(HttpMethod.GET, "/exception/**", "/helloworld/**","/actuator/health", "/v1/board/**", "/favicon.ico").permitAll() // 등록된 GET요청 리소스는 누구나 접근가능
//                    .anyRequest().hasRole("USER") // 그외 나머지 요청은 모두 인증된 회원만 접근 가능
            .and()
                .exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler())
            .and()
                .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())
            .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class); // jwt token 필터를 id/password 인증 필터 전에 넣어라.

    }
생략 ...
}
```

- @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)를 추가하고 authorizeRequest는 주석 처리해야 한다.

```java
@PreAuthorize("hasRole('ROLE_USER')") 또는 @Secured("ROLE_USER")
@Api(tags = {"2. User"})
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/v1")
public class UserController {
내용 생략....
}

@Api(tags = {"2. User"})
@RequiredArgsConstructor
@RestController
@RequestMapping(value = "/v1")
public class UserController {

    private final UserJpaRepo userJpaRepo;
    private final ResponseService responseService; // 결과를 처리할 Service

    @Secured("ROLE_USER")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "X-AUTH-TOKEN", value = "로그인 성공 후 access_token", required = true, dataType = "String", paramType = "header")
    })
    @ApiOperation(value = "회원 리스트 조회", notes = "모든 회원을 조회한다")
    @GetMapping(value = "/users")
    public ListResult<User> findAllUser() {
        // 결과데이터가 여러건인경우 getListResult를 이용해서 결과를 출력한다.
        return responseService.getListResult(userJpaRepo.findAll());
    }

    @PreAuthorize("hasRole('ROLE_USER')")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "X-AUTH-TOKEN", value = "로그인 성공 후 access_token", required = true, dataType = "String", paramType = "header")
    })
    @ApiOperation(value = "회원 단건 조회", notes = "회원번호(msrl)로 회원을 조회한다")
    @GetMapping(value = "/user")
    public SingleResult<User> findUser() {
        // SecurityContext에서 인증받은 회원의 정보를 얻어온다.
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String id = authentication.getName();
        // 결과데이터가 단일건인경우 getSingleResult를 이용해서 결과를 출력한다.
        return responseService.getSingleResult(userJpaRepo.findByUid(id).orElseThrow(CUserNotFoundException::new));
    }
  생략...
}
```