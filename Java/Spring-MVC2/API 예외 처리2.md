# API 예외 처리2

### 스프링 HandlerExceptionResolver

1. ExceptionHandlerExceptionResolver
    1. @ExceptionaHandler를 처리.
2. ResponseStatusExceptionResolver
    1. HTTP 상태 코드를 지정
3. DefaultHandlerExceptionResolver
    1. 기본 내부 예외를 처리

**ResponseStatusExceptionResolver**

- @ResponseStatus가 달려있는 예외
- ResponseStatusException 예외

```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException {}
```

- BadRequestException 예외가 컨트롤러 밖으로 넘어가면 ResponseStatusExceptionResolver 예외가 해당 에노테이션을 확인해서 오류 코드를 HttpStatus.BAD_REQUET으로 변경하고, 메시지도 담는다.
- 실제로 내부에서 response.sendError(statusCode, resolveReason)을 호출한다.
- reason을 MessageSource에서 찾는 기능도 제공

**ResponseStatusException**

- @ResponseStatus는 개발자가 직접 변경할 수 있는 예외에는 적용할 수 없음.
- 이때는 ResponseStatusException 예외를 사용

```java
@GetMapping("api/response-status-ex2")
public String responseStatusEx2() {
	throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArugmentException())
}
```

**DefaultHandlerExceptionResolver**

- 스프링 내부에서 발생하는 스프링 예외를 해결
- 대표적으로 파라미터 바인딩 시점에 타입이 맞지 않으면 내부에서 TypeMismatchException이 발생 시 400오류로 변경
- response.sendError()를 통해서 문제 해결하기에 WAS에서 다시 오류 페이지를 요청

### ExceptionHandlerExceptionResolver

- @ExceptionHandler 애노테이션을 선언하고, 해당 컨트롤러에서 처리하고 싶은 예외를 지정해주면 된다.
- 해당 컨트롤러에서 예외가 발생하면 이 메서드가 호출됨.
- 스프링은 항상 자세한 것이 우선순위가 높기 때문에 부모클래스를 지정하면 자식 클래스까지 처리.

```java
@ExceptionHandler(IllegalArgumentException.class)
public ErrorResult illegaExHandle(IllegaArugmentException ex) {}
```

- 다양한 예외 한꺼번 처리

```java
@ExceptionHandler({AException.class, BException.class})
```

- 예외 생략

```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {}
```

- HTML 오류 화면 처리

```java
@ExceptionHandler(ViewException.class)
public ModelAndView ex(ViewException e) {
 log.info("exception e", e);
 return new ModelAndView("error");
}
```

- [파라미터 및 응답 지정](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args)

**예외 처리실행 흐름**

- 컨트롤러 호출한 결과 예외가 컨트롤러 밖으로 던져진다.
- 예외가 발생했으므로 ExceptionResolver가 작동하고 우선순위가 높은 ExceptionHandlerExceptionResolver가 작동
- ExceptionHandlerExceptionResolver 는컨트롤러에서 해당 예외를 처리할 수 있는 @ExceptionHandler가 있는 지 확인하고 있다면 처리.

### @ControllerAdvice

- 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여해주는 역할
- 대상을 지정하지 않으면 모든 컨트롤러에 적용
- @RestControllerAdvice는 ControllerAdvice에서 ResponseBody가 추가되어 있는 경우

```java
@RestControllerAdvice
public class ExControllerAdvice {
 @ResponseStatus(HttpStatus.BAD_REQUEST)
 @ExceptionHandler(IllegalArgumentException.class)
 public ErrorResult illegalExHandle(IllegalArgumentException e) {
 log.error("[exceptionHandle] ex", e);
 return new ErrorResult("BAD", e.getMessage());
 }
 @ExceptionHandler
 public ResponseEntity<ErrorResult> userExHandle(UserException e) {
 log.error("[exceptionHandle] ex", e);
 ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
 return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
 }
 @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
 @ExceptionHandler
 public ErrorResult exHandle(Exception e) {
 log.error("[exceptionHandle] ex", e);
 return new ErrorResult("EX", "내부 오류");
 }
}
```

**컨트롤러 지정**

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}
// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}
// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class,
AbstractController.class})
public class ExampleAdvice3 {}
```