# ControllerAdvice를 이용한 Exception 처리

### @ControllerAdvice

- Spring에서 제공하는 annotation으로 Controller 전역에 적용되는 코드를 작성 할 수 있게 해준다.
- @ControllerAdvice와 @ExceptionHandler를 조합하여 예외 처리를 공통 코드로 분리하여 작성할 수 있음.

### @ExceptionHanlder(Exception.class)

- Exception이 발생하면 해당 handler로 처리하겠다고 명시하는 annotation
- @Controller, @RestController가 적용된 bean내에서 발생하는 예외를 잡아서 하나의 메서드로 처리해주는 기능.
- 괄호안에는 어떤 Exception이 발생할 때 적용할 지 해당 Exception class를 인자로 넣는다.

**예제 코드**

```java
@RequiredArgsConstructor
@RestControllerAdvice
public class ExceptionAdvice {

    private final ResponseService responseService;
//모든 예외에서 해당 handler 호출.
    @ExceptionHandler(Exception.class) 
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    protected CommonResult defaultException(HttpServletRequest request, Exception e) {
        return responseService.getFailResult();
    }
}
```

- 실무에서는 에러 반환 타입을 잘 정의해서 일정하게 사용될 수 있도록.

### Exception 고도화 - Custom Exception 정의

- 매번 정의된 Exception을 사용하는 것은 여러 가지 예외 상황을 구분하는데 적합하지 않을 수 있다.

```java
public class CUserNotFoundException extends RuntimeException{
    public CUserNotFoundException() {
    }

    public CUserNotFoundException(String message) {
        super(message);
    }

    public CUserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```