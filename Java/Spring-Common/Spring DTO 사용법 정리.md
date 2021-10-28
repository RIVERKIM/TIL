# Spring DTO 사용법 정리

생성일: 2021년 10월 9일 오후 1:31

### DTO

- Data Transfer Object의 약자로 계층 간의 데이터 교환을 위한 Java Beans를 말하며 VO(Value Object) 라고도 불린다.

![Untitled](Spring%20DTO%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%87%E1%85%A5%E1%86%B8%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20d94ce46157c84253b3d9845d85c47486/Untitled.png)

- MVC 패턴 기준으로 했을 때
    - Presentation Layer - Controller
    - Business Layer - Service
    - Persistence Layer - JDBC, ORM
    - Database - MySql ~

**필요한 이유**

- 어떠한 동적인 조건, 혹은 INSERT, UPDATE 작업이 이루어질 경우, 새로이 생성하고 싶은 데이터는 각 칼럼에 들어가야 할 내용, 수정하고 싶다면 , 기존의 내용 혹은 수정하고자 하는 내용이 있어야 한다.
- 따라서 계층간의 데이터 교환에 쓰이는 형식을 정의한다.
- Entity는 변경되면 안되기 때문에 대신 DTO을 사용한다.

```jsx
// 유저 로그인 담당하는 DTO 정의
public class userDto {
    @Getter
    @NoArgsConstructor(access = AccessLevel.PUBLIC)
    public static class SignInReq {
        @Vaild
        public String userId;

        @Valid
        public String password;

        @Builder
        public SignInReq(String userId, String password) {
            this.userId = userId;
            this.password = password
        }
    }

    @Getter
    public static class SignInRes {
        private final AppToken token;
        private final AppToken refreshToken;
        private final Collection<? extends GrantedAuthority> authorities;

        @Builder
        public SignInRes(String token, Date tokenExpired,
                         String refreshToken, Date refreshTokenExpired,
                         Collection<? extends GrantedAuthority> authorities) {
            this.token = AppToken.builder().value(token).expired(tokenExpired).build();
            this.refreshToken = AppToken.builder().value(refreshToken).expired(refreshTokenExpired).build();
            this.authorities = authorities
        }
    }

}

// 실제 로그인을 담당하는 로그인 컨트롤러
@RepositoryRestController
@RequiredArgsController
@ResponseBody
public class UserController {
	@PostMapping
	@ResponseStatus(value = HttpStatus.OK)
	public UserDto.SignInRes signInUser(@RequestBody @Valid final UserDto.SignInReq dto) {
}
}
```

### DTO 문제점

- Api 별로 화면에 return 하는 데이터가 달라 많은 DTO 가 생성된다.
- Entity들의 데이터를 가공하여 DTO에 set method 혹은 builder로 매핑하는 코드가 길어진다.

### DTO를 Inner Class로 처리

```jsx
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

// DTO를 InnerClass로 정의하여 이를 포함하고 있는 User class
public class User {

    @Getter
    @AllArgsConstructor
    @Builder
    public static class Info {
        private int id;
        private String name;
        private int age;
    }

    @Getter
    @Setter
    public static class Request {
        private String name;
        private int age;
    }

    @Getter
    @AllArgsConstructor
    public static class Response {
        private Info info;
        private int returnCode;
        private String returnMessage;
    }
}
// 유저 컨트롤러
@RestController
@RequestMapping(“user”)
public class UserController {

    @GetMapping(“/{user_id}”)
    public User.Response getUser(@PathVariable(“user_id”) String userId) {

        return new User.Response(new User.Info(), 200, “success”);
    }

    @PostMapping
    public DefaultResponse addUser(@RequestBody User.Info info) {

        return new DefaultResponse();
    }
}
```

- Entity안에 DTO를 저장함으로써 조금 더 깔끔한 패키지를 만들 수 있고 DTO 명을 찾는 것이 수월해졌다.