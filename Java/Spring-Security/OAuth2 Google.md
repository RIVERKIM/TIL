# OAuth2 Google

### Google OAuth2

**기본 로직**

- 사용자가 access_token을 헤더에 포함해서 요청
- JWT filter를 이용해서 토큰을 검증하고 oauth2Authentication을 설정.
- security configuration에서 oauth2Login() 을 통과하고 userService에서 우리가 설정한 Oauth2 Service로 유저 설정 후 요청한 페이지 controller 호출.

**application-oauth.yml**

```java
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 클라이언트 ID
            client-secret: 클라이언트 보안 비밀번호
            scope: profile, email
```

- scope 기본 값이 openid, profile,  email인데 openid라는 scope가 있으면 OpenId Provider로 인식하기 떄문에, 이렇게 되면 OpenId Provider인 서비스와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 한다.
- application.yml 에 spring.profiles.include: oauth를 포함하면 위의 파일 포함 가능.

**User**

```java
@Entity
@Getter
@NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;
        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```

**Role**

```java
@Getter
@RequiredArgsConstructor
public enum Role {
    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String value;
}
```

**UserJpaRepository**

```java
public interface UserJpaRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

**스프링 시큐리티 설정**

```java
//소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현시 필요한 의존성.
compile('org.springframework.boot:spring-boot-starter-oauth2-client')
```

**Security Config**

```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomeOAuth2UserService customeOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                .antMatchers("/", "/css/**", "/images/**", "/js/**").permitAll()
                .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                .anyRequest().authenticated()

                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .userInfoEndpoint() //oauth2 로그인 성공 후 가져올 때의 설정들
                //소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스 구현체 등록.
                //리소스 서버에서 사용자 정보를 가져온 상태에서 추가로 진행하고자하는 기능 명시.
                .userService(customeOAuth2UserService);

        super.configure(http);
    }
}
```

- oauth2Login(): formLogin() 과 그 형태가 같다. 만약 인증이 안된 사용자가 온다면 oauth2 로그인 하도록 유도.

**SessionUser**

- User클래스를 그대로 사용하면 직렬화를 구현하지 않았기 때문에 에러가 발생, 따라서 직렬화된 클래스 생성.
- User클래스는 다른 엔티티와 연관관계가 생길 수 있는 엔티티이기에 성능 이슈, 부수 효과가 발생할 확률이 높아지기 때문에 직렬화 하지 않는다.

```java
@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(String name, String email, String picture) {
        this.name = name;
        this.email = email;
        this.picture = picture;
    }
}
```

**OAuth2Attributes**

- OAuth2UserService를 통해 가져온 OAuth2User의 attributes를 담을 클래스

```java
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder

    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    public static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.USER)
                .build();
    }
}
```

**CustomOAuth2UserService**

- OAuth2UserService 사용자의 정보들을 기반으로 가입 및 정보 수정, 세션 저장등의 기능을 지원해줌.

```java
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        // OAuth2 서비스 id (구글, 카카오, 네이버)
        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        // OAuth2 로그인 진행 시 키가 되는 필드 값(PK)
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

        // OAuth2UserService
        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());
        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user)); // SessionUser (직렬화된 dto 클래스 사용)

        return new DefaultOAuth2User(Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    // 유저 생성 및 수정 서비스 로직
    private User saveOrUpdate(OAuthAttributes attributes){
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());
        return userRepository.save(user);
    }
}
```

> OAuth2 사이트:
[https://www.baeldung.com/spring-security-5-oauth2-login](https://www.baeldung.com/spring-security-5-oauth2-login)
[https://galid1.tistory.com/582](https://galid1.tistory.com/582)
[https://loosie.tistory.com/300](https://loosie.tistory.com/300)
>