# MVC 프레임워크 만들기

### 유연한 컨트롤러1 -V5

**어댑터 패턴**

- 호환이 불가능한 서로 다른 인터페이스 사이에서 연결해주는 역할

**V5 구조**

![Untitled](MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2032cdae81d83c41df899ec9078da6cc4b/Untitled.png)

- 핸들러 어댑터: 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다. 어댑터 역할을 해주는 덕분에 다양한 컨트롤러 호출 가능.
- 핸들러: 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떤 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.

**MyHandlerAdaper**

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

- boolean supports(Object handler)
    - hanlder는 컨트롤러를 말한다.
    - 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드
- ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException
    - 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
    - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 직접 생성해서라도 반환해야 한다.
    - 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만, 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.

**ControllerV3HandlerAdapter**

```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);

        return controller.process(paramMap);
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

- handler를 컨트롤러 V3로 변환한 다음에 V3형식에 맞추도록 호출한다.
    - supports()를 통해 ControllerV3만 지원하기 때문에 타입 변환은 걱정없이 실행해도 된다.

**FrontControllerServletV5**

```java
@WebServlet(name = "frontControllerV5", urlPatterns = "/front-controller/v5/*")
public class FrontServletControllerV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontServletControllerV5() {
        initHandlerMappingMap();

        initHandlerAdapters();
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        ModelView mv = adapter.handle(request, response, handler);

        String viewName = mv.getViewName();

        MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }

        throw new IllegalArgumentException("handler adapter를 찾을 수 없다. handler = " + handler);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    private MyView viewResolver(String viewName) {
        MyView view = new MyView("/WEB-INF/views/" + viewName + ".jsp");
        return view;
    }
}
```

**컨트롤러(Controller)  → 핸들러(Handler)**

- 이전에는 컨트롤러를 직접 매핑해서 사용했다.
- 이제는 어댑터를 사용하기 때문에, 컨트롤러 뿐만 아니라 어댑터가 지원하기만 하면 어떤 것이라도 URL에 매핑해서 사용할 수 있다.

### 유연한 컨트롤러2 - V5

- FrontControllerServletV5에 ControllerV4 기능 추가

```java
private void initHandlerMappingMap() {
 handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new
MemberFormControllerV3());
 handlerMappingMap.put("/front-controller/v5/v3/members/save", new
MemberSaveControllerV3());
 handlerMappingMap.put("/front-controller/v5/v3/members", new
MemberListControllerV3());
 //V4 추가
 handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new
MemberFormControllerV4());
 handlerMappingMap.put("/front-controller/v5/v4/members/save", new
MemberSaveControllerV4());
 handlerMappingMap.put("/front-controller/v5/v4/members", new
MemberListControllerV4());
}
private void initHandlerAdapters() {
 handlerAdapters.add(new ControllerV3HandlerAdapter());
 handlerAdapters.add(new ControllerV4HandlerAdapter()); //V4 추가
}
```

- 만약 여기서 handlerMappingMap과 handlerAdapters에 DI를 한다면 frontController를 손대지 않고도 변경 가능.

**ControllerV4HandlerAdapter**

```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }
}
```

- 어댑터는 뷰의 이름이 아니라 ModelView를 만들어서 반환해야 한다.
- ControllerV4는 뷰의 이름을 반환했지만, 어댑터는 이것을 ModelView로 만들어서 형식을 맞추어 반환한다.