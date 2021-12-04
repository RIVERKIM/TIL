# Interceptor

### Interceptor

- Client에서 Server로 들어온 Request 객체를 Controller의 Handler로 도달하기 전에 가로채어, 원하는 추가 작업이나 로직을 수행 한 후 Handler로 보낼 수 있도록 해주는 Module 이다.
- Interceptor는 Controller의 Handler가 실행되기 전이나 후에 추가적인 작업이 수행되어야 할 때 사용한다.

![Untitled](Interceptor%20258b37d8ccb84d249e74442cee91c548/Untitled.png)

- DispatcherServlet은 HanlderMapping에게 Client Request를 수행 할 Handler를 찾도록 요청을 보낸다.
- 이 때 HandlerExceptionChain이 동작하는데, 이것은 하나 이상의 HandlerInterceptor를 거쳐 Controller가 실행되도록 구성되어 있다.
- HandlerInterceptor를 등록하지 않은 경우 바로 Controller 실행
- HandlerInterceptor를 거쳐 Request에 대해 원하는 작업, 로직을 수행한 후 Controller로 Request 객체를 전달한다.
- ex) Login Session 검증, Header 검증, Token 검증, URL Handling등등

**Interceptor 장점**

- 공통 코드 사용으로 코드 재사용성 증가
- 메모리 낭비, 서버 부하 감소
- 코드 누락에 대한 위험성 감소

```java
@AllArgsConstructor
@RequestMapping("admin")
@RestController
public class AdminController {

    private AdminService adminService;

    @PostMapping("/login")
    public ResponseEntity<CommonResponse> loginAdmin(@RequestHeader(value = "Authorization") String jwt, @RequestBody UserDTO userDTO) throws JwtException {
    	...jwt 검증 및 로그인 검사..
        return ResponseEntity.ok().body(new CommonResponse());
    }
    
    @PostMapping("/register")
    public ResponseEntity<CommonResponse> registerAdmin(@RequestHeader(value = "Authorization") String jwt, @RequestBody UserDTO userDTO) throws JwtException {
    	...jwt 검증 및 회원가입 수행..
        return ResponseEntity.ok().body(new CommonResponse());
    }
 
    ....
 
     @GetMapping("/{id}")
    public ResponseEntity<CommonResponse> detailAdmin(@PathVariable(name="id") String id){
    	...jwt 검증 및 세부 정보 ..
        return ResponseEntity.ok().body(new CommonResponse());
    }
}
```

- 모든 Handler에서 JWT 검증을 수행하며 JwtException을 Throw 하는 것 로직이 계속해서 늘어나게 된다.
- 모든 Handler에서 같은 로직을 수행하는 코드가 존재해 메모리, 서버 부하 증가.
- 위와 같은 상황에서 Interceptor 활용시 JWT 검증 로직을 Interceptor에서 한번만 작성하고, /admin/* 에 대한 요청에 Interceptor를 적용하여 작성 코드량을 줄여 메모리 낭비를 줄일 수 있다.
- Interceptor 적용의 기준 url을 명시해 줌으로써 일괄적으로 /admin/*에 해당하는 Handler들에 적용해주어 코드 누락의 위험성을 줄일 수 있다.

### Interceptor 구현하기

- Spring에서 Interceptor의 구현은 **`HandlerInterceptor(Interface)`**나 **`HandlerInterceptorAdapter(Abstract Class)`**로 구현할 수 있다.
- *`**HandlerInterceptorAdapter`** Abstract Clas는 `**HandlerInterceptor`** Interface를 상속받아 구현됨*
- 두가지 중 하나를 구현하는 class를 생성한 후, **`DispatcherServlet`**의 Context에 작성 Interceptor class를 Bean으로 등록하고, 적용 URI를 명시해주면 된다.

**구현 메서드**

- **PreHandle(HttpServletRequest request, HttpServletResponse response, Object handler)**
    - Controller 실행 직전에 동작하는 메서드
    - return 값이 true일 경우 정상적으로 진행 되고, false 일 경우 실행 종료- Controller 진입 X
    - Object handler는 HandlerMapping이 찾은 Controller Class 객체
- **PostHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)**
    - Controller 진입 후 View가 Rendering 되기 전 수행된다.
    - ModelAndView modelAndView를 통해 Data 등의 조작 가능
- **afterComplete(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)**
    - Controller 진입 후 View가 정상적으로 Rendering 된 후 수행된다.
- **afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler)**
    - 비동기 요청 시 PostHandle와 afterComplete 메서드를 수행하지 않고 이 메서드를 수행
    

```java
public class JwtInterceptor extends HandlerInterceptorAdapter {
	private static final Logger LOG = Logger.getLogger(JwtInterceptor.class);
	@Autowired
	private AuthService authService;

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws IOException, JwtException {
		LOG.info("jwt interceptor cateched!!!");
		String jwt = request.getHeader("Authorization");
		if (jwt == null) {
			LOG.error("JWT is null");
			throw new AuthenticationException("JWT is null");
		}
		try {
			authService.vertifyJwt(jwt);
		} catch (JwtException e) {
			LOG.error(jwt);
			LOG.error("JWT is not valid");
			throw new AuthenticationException("JWT is not valid");
		}
		return true;
	}
}
```

**설정**

- WebMvcConfigurationSupport를 상속 받아 등록.

```java
@Configuration
public class CustomWebMvcConfigurer extends WebMvcConfigurationSupport {

   @Override
   protected void addInterceptors(InterceptorRegistry registry) {
      registry.addInterceptor(new AdminInterceptor())
            .addPathPatterns("/admin/**")
            .excludePathPatterns("/admin/myPage");

      registry.addInterceptor(new UserInterceptor())
            .addPathPatterns("/user/?")
            .addPathPatterns("/user/*")
            .addPathPatterns("/user/**");
   }
}
```

- addPathPatterns: 추가할 주소
- excludePathPatterns: 예외 처리할 주소
- Url 패턴
    - *: 1 뎁스
    - **: 1뎁스 그 이상 주소 모든 경로
    - ?: 한개의 글자