# 스프링 MVC 구조

### 스프링 MVC 전체 구조

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%204d5d1c9fad8c4c4bacbe467e7bb194ce/Untitled.png)

**DispatcherServlet**

- org.springframework.web.servlet.DispatcherServlet
- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
- 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿이다.

**DispatcherServlet  서블릿 등록**

- DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
    - DispatcherServlet → FrameworkServlet → HttpServletBean → HttpServlet
- 스프링 부트는 DispatcherServlet을 서블릿으로 자동으로 등록하면서 모든 경로 urlPatterns = "/"에 대해서 매핑한다.
    - 참고: 더 자세한 경로가 우선순위가 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.

**요청 흐름**

- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
- 스프링 MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service(를 오버라이드해두었다.
- FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.

**DispatcherServlet.doDispatch()**

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse
response) throws Exception {
HttpServletRequest processedRequest = request;
HandlerExecutionChain mappedHandler = null;
ModelAndView mv = null;
// 1. 핸들러 조회
mappedHandler = getHandler(processedRequest);
if (mappedHandler == null) {
noHandlerFound(processedRequest, response);
return;
}
// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
processDispatchResult(processedRequest, response, mappedHandler, mv,
dispatchException);
}
private void processDispatchResult(HttpServletRequest request,
HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView
mv, Exception exception) throws Exception {
// 뷰 렌더링 호출
render(mv, request, response);
}
protected void render(ModelAndView mv, HttpServletRequest request,
HttpServletResponse response) throws Exception {
View view;
String viewName = mv.getViewName();
// 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
// 8. 뷰 렌더링
view.render(mv.getModelInternal(), request, response);
}
SpringMVC 구조
동작 순서
1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한
```

**스프링 MVC 동작 순서**

- 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러를 조회한다.
- 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
- 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
- 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
- ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
- ViewResolver 호출: 뷰 리졸버를 찾고 실행한다.
    - JSP의 경우: InternalResourceViewResolver가 자동 등록되고, 사용된다.
- View 반환:  뷰 리졸버는 뷰의 논리 이름을 물리이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
    - JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
- 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

**인터페이스 살펴보기**

- 스프링 MVC의 큰 강점은 DispatcherServlet 코드의 변경 없이, 원하는 기능을 변경하거나 확장할 수 있다는 점이다.

### 핸들러 매핑과 핸들러 어댑터

**Controller 인터페이스**

- 과거 버전 스프링 컨트롤러: org.springframework.web.servlet.mvc.Controller

```java
public interface Controller {
ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse
response) throws Exception;
}
```

**OldController**

```java
package hello.servlet.web.springmvc.old;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
    }
}
```

- @Component: 이 컨트롤러는 /springmvc/old-controller라는 이름의 스프링 빈으로 등록되었다.
- 빈의 이름으로 URL을 매핑할 것이다.

**스프링 MVC**

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20MVC%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%204d5d1c9fad8c4c4bacbe467e7bb194ce/Untitled%201.png)

이 컨트롤러가 호출되려면 2가지가 필요하다.

- HandlerMapping
    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
    - 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
- HandlerAdapter
    - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    - 예) Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

**스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터**

- HandlerMapping
    - 0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    - 1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
- HandlerAdapter
    - 0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
    - 1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
    - 2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리

**핸들러 매핑으로 핸들러 조회**

- HandlerMapping 을 순서대로 실행해서, 핸들러를 찾는다.
- 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 BeanNameUrlHandlerMapping 가 실행에 성공하고 핸들러인 OldController 를 반환한다.

**핸들러 어댑터 조회**

- HandlerAdapter 의 supports() 를 순서대로 호출한다.
- SimpleControllerHandlerAdapter 가 Controller 인터페이스를 지원하므로 대상이 된다.

**핸들러 어댑터 실행**

- 디스패처 서블릿이 조회한 SimpleControllerHandlerAdapter 를 실행하면서 핸들러 정보도 함께 넘겨준다.
- SimpleControllerHandlerAdapter 는 핸들러인 OldController 를 내부에서 실행하고, 그 결과를 반환한다.

**HttpRequestHandler**

- HttpRequestHandler는 서블릿과 가장 유사한 형태의 핸들러이다.

```java
public interface HttpRequestHandler {
void handleRequest(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException;
}
```

**MyHttpRequestHandler**

```java
@Component("/springmvc/request-handler")
public class MyHttpRequestHandler implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
}
```

- MyHttpRequestHandler 핸들러매핑, 어댑터
    - MyHttpRequestHandler 를 실행하면서 사용된 객체는 다음과 같다.
    - HandlerMapping = BeanNameUrlHandlerMapping
    - HandlerAdapter = HttpRequestHandlerAdapter

### 뷰 리졸버

**OldController - View 조회할 수 있도록 변경**

```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return new ModelAndView("new-form");
    }
}
```

- application.properties에 다음 코드 추가
    - spring.mvc.view.prefix=/WEB-INF/views
    - spring.mvc.view.suffix=.jsp
- 스프링 부트는 InternalResourceViewResolver라는 뷰 리졸버를 자동으로 등록하는데, 이 때 application.properties에 등록한 위 두 값을 설정 정보에 사용하여 등록한다.

**스프링 부트가 자동 등록하는 뷰 리졸버**

- 1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성기능에 사용)
- 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.