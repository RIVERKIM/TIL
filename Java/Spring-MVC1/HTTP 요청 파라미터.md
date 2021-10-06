# HTTP 요청 파라미터

### Http 요청 - 기본, 헤더 조회

**RequestHandlerController**

```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie) {
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);
        return "ok";

    }
}
```

- HttpServletRequest
- HttpServletResponse
- HttpMethod: HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
- Locale: Locale 정보를 조회한다.
- @RequestHeader MultiValueMap<String, String> headerMap
    - 모든 Http 헤더를 MultiValueMap 형식으로 조회
- @RequestHeader("")
    - 특정 Http 헤더 조회
    - 속성
        - 필수 값 여부: required
        - 기본 값: defaultValue

> 참고: 
 [@Conroller 의 사용 가능한 파라미터 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)
[@Controller의 사용 가능한 응답 값 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)
> 

### Http 요청 파라미터 - 쿼리 파라미터, HTML Form

**HTTP 요청 데이터 조회**

- GET - 쿼리 파라미터
    - /url?username=hello&age=20
    - 메시지 바디 없이, URL 의 쿼리 파라미터에 데이터를 포함해서 전달
- POST - HTML FORM
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello&age=23
- HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON 등

**@RequestParam**

- 파라미터 이름으로 바인딩
- String, int, Integer등의 단순 타입이면 @RequestParam도 생략 가능
    - 어노테이션 생략시 required = false로 처리된다.
- @RequestParam.required
    - 파라미터 필수 여부
    - 기본 값이 true
    - null은 통과되지 않지만 ""빈문자는 통과 된다.
- @RequestParam Map<String, Object>
    - 전체 param을 조회할 때 맵을 사용한다.
    - 다수의 값을 갖는 param일 경우 MultiValueMap을 사용하자

**@ModelAttribute**

- 실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다.
- 객체를 생성해서, 요청 파라미터의 이름으로 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
- 바인딩 오류: 타입과 다른 값이 들어가면 BindException이 발생한다. 이런 바인딩 오류는 따로 처리해 주어야 한다.
- 생략 가능하며, 단순 타입은 @RequestParam이 자동으로 붙고, 나머지는 @ModelAttribute가 자동으로 붙는다고 생각하면 된다.

> **프로퍼티**: 객체에 getXxx, setXxx 메서드가 있으면, 이 객체는 xxx라는 프로퍼티를 가지고 있다.
>