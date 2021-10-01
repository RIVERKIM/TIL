# MVC 패턴

### 서블릿과 JSP 한계

서블릿으로 개발할 때는 뷰 화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡했다.

JSP를 사용한 덕분에 뷰를 생성하는 HTML 작업을 깔끔하게 가져가고, 중간중간 동적으로 변경이 필요한 부분에만 자바 코드를 적용했다. 그런데 코드를 잘 보면 JAVA코드, 데이터 조회 리포지토리 등등 다양한 코드가 모두 JSP에 있다. 이는 JSP가 너무 많은 역할을 담당하게 함으로써 유지보수가 어렵다.

### MVC 패턴의 등장

**변경의 라이프 사이클**

- UI를 일부 수정하는 일과 비지니스 로직을 수정하는 일은 각각 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다.
- 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다.

**Model View Controller**

- MVC 패턴은 하나의 서블릿이나 , JSP로 처리하던 것을 컨트롤러와 뷰라는 영역으로서로 역할을 나눈 것을 말한다.
- **컨트롤러**: HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
- **모델**: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비지니스 로직이나 데이터 접근을 몰라도 되고, 화면을 렌더링 하는 일에 집중할 수 있다.
- **뷰**: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다.

![Untitled](MVC%20%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%203b9128ade8a54f1bbda5116a1ed98809/Untitled.png)

> **참고**
컨트롤러에 비즈니스 로직을 둘 수도 있지만, 이렇게 되면 컨트롤러가 너무 많은 역할을 담당한다. 그래서 일반적으로 비즈니스 로직은 서비스(Service)라는 계층을 별도로 만들어서 처리한다. 그리고 컨트롤러는 비즈니스 로직이 있는 서비스를 호출을 담당한다. 비즈니스 로직을 변경하면 비즈니스 로직을 호출하는 컨트롤러의 코드도 변경될 수 있다.
> 

### MVC 패턴 -적용

- 서블릿은 컨트롤러를 사용하고, JSP를 뷰로 사용해서 MVC패턴을 사용해보자.
- Model은 HttpServletRequest객체를 사용한다.

**회원 등록**

```java
package hello.servlet.web.servlet.servletmvc;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

- dispatcher.forward(): 다른 서블릿이나 jsp로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생.

> /WEB-INF
이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서 JSP를 호출하는 것이다
> 

> **Redirect vs Forward**
리다이렉트는 실제 클라이언트에 응답이 나갔따가, 클라이언트가 redirect경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, URL 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.
> 

**회원 등록 폼**

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
 username: <input type="text" name="username" />
 age: <input type="text" name="age" />
 <button type="submit">전송</button>
</form>
</body>
</html>
```

**회원 저장**

```java
package hello.servlet.web.servlet.servletmvc;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //model에 데이터 보관
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

**회원 저장 뷰**

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
 <meta charset="UTF-8">
</head>
<body>
성공
<ul>
 <li>id=${member.id}</li>
 <li>username=${member.username}</li>
 <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

### MVC 패턴 - 한계

- MVC 패턴을 적용한 덕분에 컨트롤러의 역할과 뷰를 렌더링 하는 역할을 명확하게 구분할 수 있다.
- 특히 뷰는 화면을 그리는 역할을 충실한 덕분에, 코드가 깔끔하고 직관적이다.
- 그런데 컨트롤러는 중복이 많고, 필요하지 않는 코드가 다수 포함되어 있다.

**컨트롤러의 단점**

- 포워드 중복
    - View로 이동하는 코드가 항상 중복 호출되어야 한다. 이 부분을 메서드로 공통화해도 되지만, 해당 메서드도 항상 직접 호출해야 한다.
    
    ```java
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
    ```
    
- ViewPath에 중복
    
    ```java
    String viewPath = "/WEB-INF/views/new-form.jsp";
    ```
    
- 사용하지 않는 코드
    - 이런 HttpServletRequest, HttpServletResonse를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.
    
    ```java
    HttpServletRequest request, HttpServletResponse response
    ```
    
- **공통 처리가 어렵다**
    - 기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 된다. 또한 호출도 중복이다.
    - 이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이 필요하다. **프론트 컨트롤러 패턴(Front Controller)**을 도입하면 이런 문제를 깔끔하게 해결할 수 있다. 스프링MVC 핵심도 바로 이 프론트 컨트롤러에 있다.