# 다양한 HTTP 요청

### HTTP 요청 메시지 - 단순 텍스트

**RequestBodyStringController**

```java
package hello.springmvc.basic.request;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.thymeleaf.util.StringUtils;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.io.Writer;
import java.nio.charset.StandardCharsets;

@Controller
@Slf4j
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String message = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("message = {}", message);
        response.getWriter().write("ok");
    }

    @PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer writer) throws IOException {
        String message = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        log.info("message = {}", message);
        writer.write("ok");
    }

    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {
        String messageBody = httpEntity.getBody();
        log.info("message = {}", messageBody);

        return new HttpEntity<>("ok");
    }

//    @PostMapping("/request-body-string-v3")
//    public ResponseEntity<String> requestBodyStringV3(RequestEntity<String> httpEntity) throws IOException {
//        String messageBody = httpEntity.getBody();
//        log.info("message = {}", messageBody);
//
//        return new ResponseEntity<>("ok", HttpStatus.CREATED);
//    }

    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {
        log.info("message = {}", messageBody);

        return "ok";
    }

}
```

- InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- OuputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
- HttpEntity: Http header, body정보를 편리하게 조회
    - 메시지 바디 정보를 직접 조회
    - 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam, @ModelAttribute
    - 응답에도 사용가능
        - 메시지 바디 정보 직접 반환
        - 헤더 정보 포함 가능
- RequestEntity
    - HttpMethod, url 정보가 추가, 요청에서 사용
- ResponseEntity
    - Http상태 코드 설정 가능, 응답에서 사용
- **@RequestBody**
    - Http메시지 바디 정보를 편리하게 조회할 수 있다.
    - 헤더 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 사용하면 된다.
    - 이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam,@ModelAttribute와 전혀 관계가 없다.
- @ResponseBody
    - 응답 결과를 Http 메시지 바디에 직접 담아서 전달할 수 있다.

> **참고**:
요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 데이터가 넘어오는 경우는@RequestParam , @ModelAttribute 를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)
> 

> **참고**:
스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이 때 HTTP 메시지 컨버터(HttpMessageConverter) 라는 기능을 사용한다.
> 

### HTTP 요청 메시지 - JSON

**RequestBodyJsonController**

```java
package hello.springmvc.basic.request;

import com.fasterxml.jackson.databind.ObjectMapper;
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMaper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody = {}", messageBody);
        HelloData helloData = objectMaper.readValue(messageBody, HelloData.class);
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

        response.getWriter().write("ok");
    }

    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

        log.info("messageBody = {}", messageBody);
        HelloData helloData = objectMaper.readValue(messageBody, HelloData.class);
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> data) throws IOException {
        HelloData helloData = data.getBody();
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
        return "ok";
    }

    @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData helloData) throws IOException {
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
        return helloData;
    }

}
```

- 문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper를 사용해서 자바 객체로 변환한다.
- 문자로 된 JSON 데이터인 messageBody를  objectMapper를 통해서 자바 객체로 변환한다.
- **@RequestBody 객체 파라미터**
    - @RequestBody에 직접 만든 객체를 지정할 수 있다.
    - HttpEntity나 @RequestBody를 사용하면 Http 메시지 컨버터가 Http 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
    - 생략이 불가능하다.
- @ResponseBody 객체 리턴
    - 객체를 Http 메시지 바디에 직접 넣어줄 수 있다.

### HTTP 응답 - 정적 리소스, 뷰 템플릿

**정적 리소스**

- 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.
    - /static , /public , /resources , /META-INF/resources
- src/main/resources 는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다
- 정적 리소스 경로
    - src/main/resources/static
- 정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

**뷰 템플릿**

- 뷰 템플릿을 겨처서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
- 뷰 템플릿 경로
    - src/main/resources/templates

**src/main/resources/templates/response/hello.html**

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p th:text="${data}">empty</p>
</body>
</html>
```

**ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러**

```java
package hello.springmvc.basic.response;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1() {
        ModelAndView mv = new ModelAndView("response/hello").addObject("data", "hello");

        return mv;
    }

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model) {
        model.addAttribute("data", "hello");
        return "response/hello";
    }

    @RequestMapping("/response/hello")
    public void responseViewV3(Model model) {
        model.addAttribute("data", "hello");
    }

}
```

- String을 반환하는 경우 - View or Http 메시지
    - @ResponseBody가 없으면 /response/hello로 뷰 리졸버가 실행되어서 뷰를 찾고 렌더링 한다.
    - 뷰의 논리 이름인 response/hello를 반환하면
        - templates/response/hello.html이 렌더링 된다.
- Void를 반환하는 경우
    - 요청 url을 참고해서 논리 뷰 이름으로 사용한다.
    - 요청 url: : /response/hello
        - 실행: templates/response/hello.html

**Thymeleaf 스프링 부트 설정**

build.gradle

```java
`implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'`
```

- 스프링 부트가 자동으로 ThymeleafViewResolver와 필요한 스프링 빈들을 등록한다.

application.properties

```java
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

> [Thymeleaf관련 추가 설정 정보](https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-application-properties.html#common-application-properties-templating)
> 

### HTTP 응답 - HTTP API, 메시지 바디에 직접 입력

**ResponseBodyController**

```java
package hello.springmvc.basic.response;

import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
//@Controller
//@ResponseBody
@RestController
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }

    @GetMapping("/response-body-string-v2")
    public ResponseEntity responseBodyV2() throws IOException {
        return new ResponseEntity("ok", HttpStatus.OK);
    }

    //@ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("df");
        helloData.setAge(21);
        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    //@ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("df");
        helloData.setAge(21);
        return helloData;
    }

}
```

- @ResponseStatus() 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
    - 단 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity를 사용하면 된다.
- **@RestController**
    - 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다.
    - 뷰 템플릿을 따로 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다.