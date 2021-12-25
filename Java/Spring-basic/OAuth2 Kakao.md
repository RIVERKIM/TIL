# OAuth2 Kakao

### OAuth Flow

![Untitled](OAuth2%20Kakao%2063d50044661549d2857a8deb063c82f2/Untitled.png)

**카카오 연동 순서**

- 사전 작업 - 카카오 개발자 센터에 App 생성
- 로그인 페이지 및 콜백 페이지 연동
- accessToken으로 가입 및 로그인 처리

> [https://developers.kakao.com/](https://developers.kakao.com/)
> 

### 카카오 로그인 및 콜백 페이지 연동

- 앱 등록 정보로 카카오 로그인 페이지를 띄움
- 카카오 로그인을 완료 시 앱 연동 화면이 나오고 동의를 통해 앱과 카카오 계정을 연결
- 앱에 설정한 콜백 페이지가 인증 코드와 함께 호출
- 전달된 콜백 페이지에서 인증코드로 token을 얻어 화면에 표시

**Kakao와 통신을 위한 RestTemplate 추가**

```java
@SpringBootApplication
public class SpringRestApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringRestApiApplication.class, args);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

**Json 라이브러리 추가**

```java
implementation 'com.google.code.gson:gson'
```

- 결과 json을 객체로 매핑하기 위해 Gson 라이브러리 사용.

**application.yml**

```java
social:
  kakao:
    client_id: XXXXXXXXXXXXXXXXXXXX # 앱생성시 받은 REST API 키
    redirect: /social/login/kakao
    url:
      login: https://kauth.kakao.com/oauth/authorize
      token: https://kauth.kakao.com/oauth/token
      profile: https://kapi.kakao.com/v2/user/me
url:
  base: http://localhost:8080
```

- Kakao api와 연동하기 위한 설정 정보 추가

**Social Controller**

```java
package com.rest.api.controller.common;

// import 생략

@RequiredArgsConstructor
@Controller
@RequestMapping("/social/login")
public class SocialController {

    private final Environment env;
    private final RestTemplate restTemplate;
    private final Gson gson;
    private final KakaoService kakaoService;

    @Value("${spring.url.base}")
    private String baseUrl;

    @Value("${spring.social.kakao.client_id}")
    private String kakaoClientId;

    @Value("${spring.social.kakao.redirect}")
    private String kakaoRedirect;

    /**
     * 카카오 로그인 페이지
     */
    @GetMapping
    public ModelAndView socialLogin(ModelAndView mav) {

        StringBuilder loginUrl = new StringBuilder()
                .append(env.getProperty("spring.social.kakao.url.login"))
                .append("?client_id=").append(kakaoClientId)
                .append("&response_type=code")
                .append("&redirect_uri=").append(baseUrl).append(kakaoRedirect);

        mav.addObject("loginUrl", loginUrl);
        mav.setViewName("social/login");
        return mav;
    }

    /**
     * 카카오 인증 완료 후 리다이렉트 화면
     */
    @GetMapping(value = "/kakao")
    public ModelAndView redirectKakao(ModelAndView mav, @RequestParam String code) {
        mav.addObject("authInfo", kakaoService.getKakaoTokenInfo(code));
        mav.setViewName("social/redirectKakao");
        return mav;
    }
}
```

- 카카오 로그인 팝업을 띄우기 위해 버튼만 존재하는 페이지.
- 관련 api  URL 포멧은 다음을 참고
- [https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api)

**token 결과를 받는 모델 생성**

```java
package com.rest.api.model.social;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class RetKakaoAuth {
    private String access_token; // api 통신시 인증 대신 실어보내는 정보
//. 헤더에 토큰 타입 방식으로 실어보내면 api 결과를 알 수 있음. ex) Authorization: bearer ~
    private String token_type; // 기본 bearer
    private long expires_in;
    private String scope; // 토큰으로 접근할 수 있는 리소스에 대한 권한 정보. 
}
```

> 참고:
RestAPI 서버에서는 카카오 로그인 화면 연동에 대한 처리가 필요 없다. 실제로 연동할 때에는 클라이언트에서 로그인까지 완료하고 access_token을 api 서버에 전달해주도록 프로세스가 진행되어야 한다. api서버에서는 access_token을 가지고 다음 작업을 진행.
> 

### Access_token 사용법

**access_token으로 로그인하는 api 생성**

- access_token으로 카카오에 profile api 정보를 호출
- 카카오 정보로 가입이 되어있는지 DB 확인
- 비 가입자의 경우엔 로그인 실패 처리
- 기 가입자일 경우 JWT 토큰 발급

**access_token으로 신규가입을 하는 api 생성**

- access_token으로 카카오에 profile api 정보를 호출
- 카카오 정보로 가입이 되어 있는지 DB 확인
- 기 가입자의 경우엔 가입 실패 처리
- 비 가입자의 경우는 신규 가입 처리 후 JWT 토큰 발급

**User**

```java
@Id // pk
 @GeneratedValue(strategy = GenerationType.IDENTITY)
 private long msrl;
 @Column(nullable = false, unique = true, length = 50)
 private String uid;
 @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
 @Column(length = 100)
 private String password;
 @Column(nullable = false, length = 100)
 private String name;
 @Column(length = 100)
 private String provider;
```

- 회원의 서비스 제공자를 알기 위해 Provider 필드를 추가
- 소셜 로그인은 password가 null 허용으로 변경.

**Kakao 유저 정보 담을 객체**

```java
@Getter
@Setter
@ToString
public class KakaoProfile {
    private Long id;
    private Properties properties;

    @Getter
    @Setter
    @ToString
    private static class Properties {
        private String nickname;
        private String thumbnail_image;
        private String profile_image;
    }
}
```

**KakaoService**

```java
package com.rest.api.service.user;

import 생략

@RequiredArgsConstructor
@Service
public class KakaoService {

    private final RestTemplate restTemplate;
    private final Environment env;
    private final Gson gson;

    @Value("${spring.url.base}")
    private String baseUrl;

    @Value("${spring.social.kakao.client_id}")
    private String kakaoClientId;

    @Value("${spring.social.kakao.redirect}")
    private String kakaoRedirect;

    public KakaoProfile getKakaoProfile(String accessToken) {
        // Set header : Content-type: application/x-www-form-urlencoded
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.set("Authorization", "Bearer " + accessToken);

        // Set http entity
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(null, headers);
        try {
            // Request profile
            ResponseEntity<String> response = restTemplate.postForEntity(env.getProperty("spring.social.kakao.url.profile"), request, String.class);
            if (response.getStatusCode() == HttpStatus.OK)
                return gson.fromJson(response.getBody(), KakaoProfile.class);
        } catch (Exception e) {
            throw new CCommunicationException();
        }
        throw new CCommunicationException();
    }

    public RetKakaoAuth getKakaoTokenInfo(String code) {
        // Set header : Content-type: application/x-www-form-urlencoded
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        // Set parameter
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("grant_type", "authorization_code");
        params.add("client_id", kakaoClientId);
        params.add("redirect_uri", baseUrl + kakaoRedirect);
        params.add("code", code);
        // Set http entity
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);
        ResponseEntity<String> response = restTemplate.postForEntity(env.getProperty("spring.social.kakao.url.token"), request, String.class);
        if (response.getStatusCode() == HttpStatus.OK) {
            return gson.fromJson(response.getBody(), RetKakaoAuth.class);
        }
        return null;
    }
}
```

- kakao api 연동 메서드들을 Service로 생성하여 사용.

**SignController**

```java
@ApiOperation(value = "소셜 로그인", notes = "소셜 회원 로그인을 한다.")
@PostMapping(value = "/signin/{provider}")
public SingleResult<String> signinByProvider(
            @ApiParam(value = "서비스 제공자 provider", required = true, defaultValue = "kakao") @PathVariable String provider,
            @ApiParam(value = "소셜 access_token", required = true) @RequestParam String accessToken) {

        KakaoProfile profile = kakaoService.getKakaoProfile(accessToken);
        User user = userJpaRepo.findByUidAndProvider(String.valueOf(profile.getId()), provider).orElseThrow(CUserNotFoundException::new);
        return responseService.getSingleResult(jwtTokenProvider.createToken(String.valueOf(user.getMsrl()), user.getRoles()));
}
```

**SpringSecurityConfiguration**

```java
.authorizeRequests()
     .antMatchers("/*/signin", "/*/signin/**", "/*/signup", "/social/**").permitAll()
```

**SignController**

```java
@ApiOperation(value = "소셜 계정 가입", notes = "소셜 계정 회원가입을 한다.")
@PostMapping(value = "/signup/{provider}")
public CommonResult signupProvider(@ApiParam(value = "서비스 제공자 provider", required = true, defaultValue = "kakao") @PathVariable String provider,
                               @ApiParam(value = "소셜 access_token", required = true) @RequestParam String accessToken,
                               @ApiParam(value = "이름", required = true) @RequestParam String name) {

        KakaoProfile profile = kakaoService.getKakaoProfile(accessToken);
        Optional<User> user = userJpaRepo.findByUidAndProvider(String.valueOf(profile.getId()), provider);
        if(user.isPresent())
            throw new CUserExistException();

        userJpaRepo.save(User.builder()
                .uid(String.valueOf(profile.getId()))
                .provider(provider)
                .name(name)
                .roles(Collections.singletonList("ROLE_USER"))
                .build());
        return responseService.getSuccessResult();
}
```

- 가입 기능 추가