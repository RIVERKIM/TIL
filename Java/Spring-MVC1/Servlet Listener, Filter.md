# Servlet Listener, Filter

- SpringMVC는 Servlet 기반의 웹 어플리케이션을 쉽게 만들 수 있도록 도와주는 프레임 워크

![Untitled](Servlet%20Listener,%20Filter%204b876c9be5a34afca3bca56163e228e2/Untitled.png)

### ServletWeb

**의존성**

- javax.servlet-api

**Listener**

- Servlet의 Listener는 웹 어플리케이션의 주요한 변화를 감지하기 때문에 이벤트가 발생했을 때 특별한 작업 처리하도록 할 수 있다.

- ServletContextListener
    - Context의 라이프사이클을 감시하는 기능 제공
        - 웹 어플리케이션 시작과 종료시, contextInitialized(), contextDestroyed() 자동 호출

```java
public class MyListener implements ServletContextListener {
//앱 시작시
	public void contextInitialized(ServletContextEvent sce) {
		sce.getServletContext().setAttribute("name", "dfdf");
	}
//앱 종료시
	public void contextDestroyed(ServletContextEvent sce) {}
}

```

**Listener 등록**

```java
@Configuration 
public class WebMvcConfiguration implements WebMvcConfigurer { 
	@Bean 
ServletListenerRegistrationBean<ServletContextListener> servletListener() { 
		ServletListenerRegistrationBean<ServletContextListener> srb = 
		new ServletListenerRegistrationBean<>(); 
		srb.setListener(new MyListener()); 
		return srb; 
	} 
}

출처: https://lelecoder.com/133 [lelecoder]
```

- **클래스를 빈으로 등록해야만 이벤트를 리스닝 할 수 있음**

### Filter

![Untitled](Servlet%20Listener,%20Filter%204b876c9be5a34afca3bca56163e228e2/Untitled%201.png)

- Dispatcher Servlet에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부과적인 작업을 처리할 수 있는 기능을 제공.
- J2EE 표준 스펙

**Filter VS Interceptor**

- Filter
    - 웹 어플리케이션의 Context의 기능
    - 스프링과 무관하게 전역적으로 처리해야 하는 작업들 처리.
    - 다음 체인으로 넘기는 ServletRequest/ServletResponse 객체를 조작할 수 있음.
    - 일반적으로 인코딩, CORS, XSS, 모든 요청에 대한 LOG, 인증, 권한 등 보안 관련 공통 작업.
- Interceptor
    - 스프링의 Spring Context의 기능이며 일종의 빈
    - Dispatcher Servlet이 컨트롤러를 호출하기 전/후에 요청과 응답을 참조하거나 가공하는 기능.
    - Controller로 넘겨주는 정보의 가공.
    - API 호출에 대한 로깅 또는 감사.
    - 스프링 컨테이너이기에 다른 빈을 주입하여 활용성이 좋음
    - 다른 빈을 활용 가능하기에 인증, 권한을 구현.
    

**MyFilter**

```java
public class MyFilter implements Filter {
// ServleContainer에 filter가 등록되어 초기화 될 때 실행.
	public void init(FilterConfig filterConfig) throws ServletException {
		
	}
// Filter가 매핑된 Servlet에 사용자 요청이 들어왔을 때 Servlet에게 전달하기 전 실행
	public void doFilter(ServletRequest servletRequest, 
				ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
		filterChain.doFilter(servletRequest, servletResponse);
	}
// ServletContainer가 종료되기전 Filter가 삭제될 때 호출
	public void destroy() {
		
	}
}
```

**Filter 설정**

- FilterRegistrationBean

```java
@Bean
public FilterRegistrationBean setFilterRegistration() {
	FilterRegistration filterRegistration = new FilterRegistration(new MyFilter());
	filterRegistration.addUrlPatterns("/filter");
	return filterRegistration;
}
```

- @WebFilter + @ServletCompnentScan

```java
@ServletComponentScan
@SpringBootApplication
public class application {
	public static void main(String[] args) {
		SpringApplication.run(application.class, args);
	}
}

@WebFilter(urlPatterns = "/filter/*")
public class MyFilter implements Filter {
}
```