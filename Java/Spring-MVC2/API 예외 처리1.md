# API 예외 처리1

### JSON 형식으로 에러처리

**ErrorPageController**

```java
@RequestMapping(value = "/error-page/500", produces =
MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest
request, HttpServletResponse response) {
 log.info("API errorPage 500");
 Map<String, Object> result = new HashMap<>();
 Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
 result.put("status", request.getAttribute(ERROR_STATUS_CODE));
 result.put("message", ex.getMessage());
 Integer statusCode = (Integer)
request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
 return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
}
```

- produces = MediaType.APPLICATION_JSON_VALUE를 추가함으로써 클라이언트가 요청하는 HTTP Header의 Accept의 값이 application/json일 때 해당 메서드가 호출되도록한다.

### 스프링 부트 기본 오류 처리

**BasicErrorController**

```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse
response) {}
@RequestMapping
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

- 클라이언트 요청의 accept 헤더 값이 text/html인 경우네는 errorHtml()을 호출해서 view 제공
- 그 외에는 error()가 호출되고 ResponseEntity로 HttpBody에 JSON 데이터 반환

**에러 발생시 응답 값**

```java
{
 "timestamp": "2021-04-28T00:00:00.000+00:00",
 "status": 500,
 "error": "Internal Server Error",
 "exception": "java.lang.RuntimeException",
 "trace": "java.lang.RuntimeException: 잘못된 사용자\n\tat
hello.exception.web.api.ApiExceptionController.getMember(ApiExceptionController
.java:19...,
 "message": "잘못된 사용자",
 "path": "/api/members/ex"
}
```

- BasicErrorController 확장하면 JSON 메세지도 변경할 수 있으나 @ExceptionHandler가 더 좋다.

### HandlerExceptionResolver

- 스프링 MVC는 컨트롤러 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공. 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 HandlerExceptionResolver를 사용.

![Untitled](API%20%E1%84%8B%E1%85%A8%E1%84%8B%E1%85%AC%20%E1%84%8E%E1%85%A5%E1%84%85%E1%85%B51%209114a47cdf12466f89377284bd222a71/Untitled.png)

- ExceptionResolver로 예외를 해결해도 postHandle() 호출되지 않음.

**HandlerExceptionResolver**

```java
public interface HandlerExceptionResolver {
 ModelAndView resolveException(
 HttpServletRequest request, HttpServletResponse response,
 Object handler, Exception ex);
}
```

**MyHandlerExceptionResolver**

```java
@Slf4j
public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
 @Override
 public ModelAndView resolveException(HttpServletRequest request,
HttpServletResponse response, Object handler, Exception ex) {
 try {
 if (ex instanceof IllegalArgumentException) {
 log.info("IllegalArgumentException resolver to 400");
 response.sendError(HttpServletResponse.SC_BAD_REQUEST,
ex.getMessage());
 return new ModelAndView();
 }
 } catch (IOException e) {
 log.error("resolver ex", e);
 }
 return null;
 }
}
```

- ExceptionResolver가 ModelAndView를 반환하는 이유는 마치 try, catch를 하듯이, Exception을 처리해서 정상 흐름처럼 변경하는 것이 목적.

**반환 값에 따른 동작 방식**

- 빈 ModelAndView
    - 뷰를 렌더링 하지 않고, 정상 흐름으로 서블릿이 리턴한다.
- ModelAndView 지정:
    - ModelAndView에 View, Model 등의 정보를 지정해서 반환하면 뷰를 렌더링 한다.
- null: null을 반환하면, 다음 ExceptionResolver를 찾아서 실행한다. 만약 처리할 수 있는

**ExceptionResolver 활용**

- 예외 상태 코드 변환
    - 예외를 response.sendError() 호출로 변경해서 서블릿에서 상태 코드에 따른 오류를 처리하도록 위임
    - 이후 WAS는 서블릿 오류 페이지를 찾아서 내부 호출 한다.
- 뷰 템플릿 처리
    - ModelAndView에 값을 채워서 예외에 따른 새로운 오류 화면 뷰 렌더링해서 제공
- API 응답 처리
    - response.getWriter().println()처럼 HTTP 응답 바디에 직접 데이터를 넣어주는 것도 가능하다.

**UserHandlerExceptionResolver**

```java
@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        try {

            if (ex instanceof UserException) {
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept");
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);

                if ("application/json".equals(acceptHeader)) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());
                    String result = objectMapper.writeValueAsString(errorResult);

                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);
                    return new ModelAndView();
                } else {
                    // TEXT/HTML
                    return new ModelAndView("error/500");
                }
            }

        } catch (IOException e) {
            log.error("resolver ex", e);
        }

        return null;
    }
}
```

**HandlerExceptionResolver 추가**

```java
public class WebConfig extends WebMvcConfigurer {

	@Override
	public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
		resolvers.add(new UserHandlerExceptionResolver());
}
}
```

- 예외가 발생해도 서블릿 컨테이너까지 예외가 전달되지 않고, 스프링 MVC에서 예외 처리는 끝이 난다.