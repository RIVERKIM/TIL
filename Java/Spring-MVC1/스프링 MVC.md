# 스프링 MVC

### @RequestMapping

**SpringMemberFormControllerV1 - 회원 등록 폼**

```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

- @Controller
    - 스프링이 자동으로 스프링 빈으로 등록한다.
    - 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
- @RequestMapping
    - 요청 정보를 매핑한다.
    - 해당 URL이 호출되면 이 메서드가 호출된다.
    - 애노테이션을 기반으로 동작하기 때문에, 메서드 이름은 임의로 지으면 된다.
- ModelAndView
    - 모델과 뷰 정보를 담아서 반환하면 된다.
    

> 참고: RequestMappingHandlerMapping은 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다.
> 

위와 동일한 코드

```java
@Component //컴포넌트 스캔을 통해 스프링 빈으로 등록
@RequestMapping
public class SpringMemberFormControllerV1 {
 @RequestMapping("/springmvc/v1/members/new-form")
 public ModelAndView process() {
 return new ModelAndView("new-form");
 }
}
```

**SpringMemberSaveControllerV1 - 회원 저장**

```java
@Controller
public class SpringMemberSaveControllerV1 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }
}
```

- mv.addObject()
    - 스프링이 제공하는 ModelAndView를 통해 Model 데이터를 추가할 때는 addObject()를 사용.
    - 이 것은 이후 뷰 렌더링에 사용된다.

**SpringMemberListControllerV1 - 회원 목록**

```java
@Controller
public class SpringMemberListControllerV1 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process() {
        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```