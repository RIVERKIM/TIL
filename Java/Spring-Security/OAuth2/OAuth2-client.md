# OAuth2-client

### ClientRegistration

- OAuth2 제공자(google 등)에 등록된 Client(우리 앱)의 정보를 나타내는 클래스
- registrationId를 통해 기초 설정이 된 ClientRegistration을 저장해두고 추후 사용  하는 곳에서 ClientRegistration.getBuilder(registrationId)을 이용해서 가져와 추가 설정할 수 있다.

### ClientRegisrationRepository

- OAuth2 제공자들에 등록된 우리 App의 정보들 (client_id, secret, redirect_uri) 들은 궁극적으로 OAuth2 제공자에 저장되고 소유되기에, 우리 App의 Server에서 요청하기 위해서는 이 정보들을 별도로 가지고 있어야 한다.
- OAuth2에 등록된 App의 정보들의 복사본이 저장되어 관리되는 클래스가 ClientRegistrationRepository이다. 구현체로는 InMemoryClientRegistrationRepository가 대표적

### OAuth2 Client 자동 등록

- SpringBoot에서는 OAuth2 인증 제공자에 대한 설정을 손쉽게 할 수 있또록 OAuth2ClientAutoConfiguration을 제공

**Oauth2ClientAutoConfiguration**

```java
@Configuration
@AutoConfigureBefore(SecurityAutoConfiguration.class)
@ConditionalOnClass({ EnableWebSecurity.class, ClientRegistration.class })
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@Import({ OAuth2ClientRegistrationRepositoryConfiguration.class, OAuth2WebSecurityConfiguration.class })
public class OAuth2ClientAutoConfiguration { }
```

- OAuth2Client에 대한 설정을 자동으로 해주는 클래스

```java
@Configuration
@EnableConfigurationProperties(OAuth2ClientProperties.class)
@Conditional(ClientsConfiguredCondition.class)
class OAuth2ClientRegistrationRepositoryConfiguration {

   private final OAuth2ClientProperties properties;

   OAuth2ClientRegistrationRepositoryConfiguration(OAuth2ClientProperties properties) {
      this.properties = properties;
   }

   @Bean
   @ConditionalOnMissingBean(ClientRegistrationRepository.class)
   public InMemoryClientRegistrationRepository clientRegistrationRepository() {
      List<ClientRegistration> registrations = new ArrayList<>(
            OAuth2ClientPropertiesRegistrationAdapter.getClientRegistrations(this.properties).values());
      return new InMemoryClientRegistrationRepository(registrations);
   }

}
```

- RegistrationRepository에 잘 알려진 OAuth2 Provider들 (google, facebook, github)에 대한 각각의 ClientRegistration을 설정해주는 역할.
- ClientRegistrationRepository 빈을 직접 설정 파일에 생성하여 주입하는 경우에는 Spring Boot에서 자동으로 설정해주는 값을 이용할 수 없다.
- 따라서, Spring Boot Security OAuth2 Client에서 제공하지 않는 다른 Provider(KAKAO)를 이용하면서 동시에 Google 등을 이용하고 싶다면 직접 설정이 필요.

### CommonOAuth2Provider

- Spring Security에서 잘 알려진 OAuth2 제공자를 좀 더 쉽게 설정하기 위해 제공되는 클래스.

```java
public enum CommonOAuth2Provider {

   GOOGLE {

      @Override
      public Builder getBuilder(String registrationId) {
         ClientRegistration.Builder builder = getBuilder(registrationId,
               ClientAuthenticationMethod.BASIC, DEFAULT_REDIRECT_URL);
         builder.scope("openid", "profile", "email");
         builder.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth");
         builder.tokenUri("https://www.googleapis.com/oauth2/v4/token");
         builder.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs");
         builder.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo");
         builder.userNameAttributeName(IdTokenClaimNames.SUB);
         builder.clientName("Google");
         return builder;
      }
   },
   
   FACEBOOK, GITHUB...
```

### KAKAO OAuth2

**CustomOAuth2Provider**

```java
public enum CustomOAuth2Provider {
    KAKAO {
        @Override
        public ClientRegistration.Builder getBuilder(String registrationId) {
          ClientRegistration.Builder builder = getBuilder(registrationId, ClientAuthenticationMethod.POST, DEFAULT_LOGIN_REDIRECT_URL)
                                                .scope("profile")
                                                .authorizationUri("https://kauth.kakao.com/oauth/authorize")
                                                .tokenUri("https://kauth.kakao.com/oauth/token")
                                                .userInfoUri("https://kapi.kakao.com/v1/user/me")
                                                .userNameAttributeName("id")
                                                .clientName("Kakao");
          return builder;
      }
    };

    private static final String DEFAULT_LOGIN_REDIRECT_URL = "{baseUrl}/login/oauth2/code/{registrationId}";

    protected final ClientRegistration.Builder getBuilder(
                String registrationId, ClientAuthenticationMethod method, String redirectUri) {

        ClientRegistration.Builder builder = ClientRegistration.withRegistrationId(registrationId)
                .clientAuthenticationMethod(method)
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .redirectUriTemplate(redirectUri);

        return builder;
    }

    public abstract ClientRegistration.Builder getBuilder(String registrationId);
}
```

**SpringSecurityConfig**

```java
@Configuration
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/login", "/oauth2/**", "/")
                .permitAll()
                .anyRequest().authenticated()
                .and()
                .oauth2Login();
    }

    @Bean
    public ClientRegistrationRepository clientRegistrationRepository(
            @Value("${spring.security.oauth2.client.registration.kakao.client-id}") String kakaoClientId,
            @Value("${spring.security.oauth2.client.registration.kakao.client-secret}") String kakaoClientSecret) {

        List<ClientRegistration> registrations = new ArrayList<>();
        registrations.add(CustomOAuth2Provider.KAKAO.getBuilder("kakao")
                            .clientId(kakaoClientId)
                            .clientSecret(kakaoClientSecret)
                            .jwkSetUri("temp")
                            .build());

        return new InMemoryClientRegistrationRepository(registrations);
    }

}
```