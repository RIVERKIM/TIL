# MessageSource를 이용한 Exception 처리

### 메시지 국제화

- message properties를 yml로 작성하기 위한 라이브러리 추가
    - 기본 설정에서는 실제 다국어 메시지가 저장되는 파일은 message_ko.properties, message_en.properties 같은 형식

```java
implementation 'dev.akkinoc.util:yaml-resource-bundle:2.3.0'
```

- 스프링에서 제공하는 `LocaleChangeInterceptor`를 사용하여 locale 이라는  RequestParameter 가 요청에 있으면 해당 값을 읽어서 LocaleResolver로 처리하여 locale 정보 변경. (자세한건 위 함수 찾아보면 됨.)

LocaleResolver

- 스프링 MVC는 LocaleResolver를 이용해서 웹 요청과 관련된 locale을 추출하고, 이 Locale 객체를 이용해서 알 맞는 언어의 메시지를 선택함.

```java
public interface LocaleResolver{

	Locale resolveLocale(HttpServletRequest request);

	void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale);

}
```

- resolveLocale() 메서드는 요청과 관련된 locale을 리턴 (DispatcherServlet은 등록되어 있는 LocaleResolver의 resolveLocale() 메서드를 호출해서웹 요청을 처리할 떄 사용할 locale을 구한다.)
- setLocale()은 Locale을 변경할 때 사용 (쿠기나, 세션에 Locale 정보를 저장할 때 사용)

**MessageConfiguration**

```java
@Configuration
public class MessageConfiguration implements WebMvcConfigurer {

    @Bean // 세션에 지역설정. default는 KOREAN = 'ko'
    public LocaleResolver localeResolver() {
        SessionLocaleResolver slr = new SessionLocaleResolver();
        slr.setDefaultLocale(Locale.KOREAN);
        return slr;
    }

    @Bean // 지역설정을 변경하는 인터셉터. 요청시 파라미터에 lang 정보를 지정하면 언어가 변경됨.
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
        lci.setParamName("lang");
        return lci;
    }

    @Override // 인터셉터를 시스템 레지스트리에 등록
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }

    @Bean // yml 파일을 참조하는 MessageSource 선언
    public MessageSource messageSource(
            @Value("${spring.messages.basename}") String basename,
            @Value("${spring.messages.encoding}") String encoding
    ) {
        YamlMessageSource ms = new YamlMessageSource();
        ms.setBasename(basename);
        ms.setDefaultEncoding(encoding);
        ms.setAlwaysUseMessageFormat(true);
        ms.setUseCodeAsDefaultMessage(true);
        ms.setFallbackToSystemLocale(true);
        return ms;
    }

    // locale 정보에 따라 다른 yml 파일을 읽도록 처리
    private static class YamlMessageSource extends ResourceBundleMessageSource {
        @Override
        protected ResourceBundle doGetBundle(String basename, Locale locale) throws MissingResourceException {
            return ResourceBundle.getBundle(basename, locale, YamlResourceBundle.Control.INSTANCE);
        }
    }
}
```

**application.yml**

- i8n 경로 및 인코딩 정보 추가

```java
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/apitest
    driver-class-name: org.h2.Driver
    password:
    username: sa
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
        show_sql: true
  messages:
    basenames: i18n/exception
    encoding: UTF-8
```

**resources/i18n/exception_en.yml**

```java
# exception_en.yml
unKnown:
  code: "-9999"
  msg: "An unknown error has occurred."
userNotFound:
  code: "-1000"
  msg: "This member not exist"
```

**resources/i18n/exception_ko.yml**

```java
# exception_ko.yml
unKnown:
  code: "-9999"
  msg: "알수 없는 오류가 발생하였습니다."
userNotFound:
  code: "-1000"
  msg: "존재하지 않는 회원입니다."
```

**ExceptionAdvice**

- 에러 메시지를 MessageSource로 처리

```java
@RequiredArgsConstructor
@RestControllerAdvice
public class ExceptionAdvice {

    private final ResponseService responseService;

    private final MessageSource messageSource;

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    protected CommonResult defaultException(HttpServletRequest request, Exception e) {
        // 예외 처리의 메시지를 MessageSource에서 가져오도록 수정
        return responseService.getFailResult(Integer.valueOf(getMessage("unKnown.code")), getMessage("unKnown.msg"));
    }

    @ExceptionHandler(CUserNotFoundException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    protected CommonResult userNotFoundException(HttpServletRequest request, CUserNotFoundException e) {
        // 예외 처리의 메시지를 MessageSource에서 가져오도록 수정
        return responseService.getFailResult(Integer.valueOf(getMessage("userNotFound.code")), getMessage("userNotFound.msg"));
    }

    // code정보에 해당하는 메시지를 조회합니다.
    private String getMessage(String code) {
        return getMessage(code, null);
    }
    // code정보, 추가 argument로 현재 locale에 맞는 메시지를 조회합니다.
    private String getMessage(String code, Object[] args) {
        return messageSource.getMessage(code, args, LocaleContextHolder.getLocale());
    }
}
```