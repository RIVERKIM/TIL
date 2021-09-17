# 회원가입 예제 MVC 개발

생성일: 2021년 9월 17일 오후 10:50

### 회원 웹 기능 - 홈 화면 추가

**홈 컨트롤러**

```java
// localhost:8080/ 에 index.html static 파일이 아닌 home.html template
//가 들어간다.
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

> 참고: 컨트롤러가 정적 파일보다 우선순위가 높다.

### 회원 웹 기능 - 등록

**회원 등록 폼 컨트롤러**

```java
@Controller
public class MemberController {
	private final MemberService memberService;

	@Autowired
	public void MemberController(MemberService memberService) {
		this.memberService = memberService;
	}
	
	@GetMapping(value="/members/new")
	public String createForm() {
		return "members/createMemberForm"
	}
}
```

**회원 등록 폼 HTML (resources/template/members/createMemberForm)**

```java
<body>
<div class="container">
    <form action="/members/new" method="post">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" placeholder="이름을
입력하세요">
        </div>
        <button type="submit">등록</button>
    </form>
</div> <!-- /container -->
</body>
</html>
```

**회원 등록 컨트롤러**

- 웹 등록 화면에서 데이터를 전달 받을 폼 객체

```java
public class MemberForm {
	private String name;

	public String getName() {
		return this.name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

- 회원 컨트롤러에서 회원을 실제 등록하는 기능

```java
@PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }
```

### 회원 웹 기능 - 조회

**회원 컨트롤러에서 조회 기능**

```java
@GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
```

**회원 리스트 HTML**

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <table>
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```