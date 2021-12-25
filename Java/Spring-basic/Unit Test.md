# Unit Test

**build.gradle**

```java
// junit, mockito 등
testImplementation 'org.springframework.security:spring-security-test'
//springsecurity test
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

### JPA Test를 위한 @DataJpaTest

- 다른 컴포넌트들은 로드하지 않고, @Entity를 읽어 Repository 내용을 테스트할 수 있는 환경을 만들어준다.
- @Transactional을 포함하고 있어 테스트가 완료되면 따로 롤백을 할 필요가 없음.
- @AutoConfigurationTestDatabse(replace = Replace.NONE)을 추가하면 메실제 데이터베이스에서 테스트도 가능.

```java
package com.rest.api.repo;

import com.rest.api.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Collections;
import java.util.Optional;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@DataJpaTest
public class UserJpaRepoTest {

    @Autowired
    private UserJpaRepo userJpaRepo;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Test
    public void whenFindByUid_thenReturnUser() {
        String uid = "angrydaddy@gmail.com";
        String name = "angrydaddy";
        // given
        userJpaRepo.save(User.builder()
                .uid(uid)
                .password(passwordEncoder.encode("1234"))
                .name(name)
                .roles(Collections.singletonList("ROLE_USER"))
                .build());
        // when
        Optional<User> user = userJpaRepo.findByUid(uid);
        // then
        assertNotNull(user);// user객체가 null이 아닌지 체크
        assertTrue(user.isPresent()); // user객체가 존재여부 true/false 체크
        assertEquals(user.get().getName(), name); // user객체의 name과 name변수 값이 같은지 체크
        assertThat(user.get().getName(), is(name)); // user객체의 name과 name변수 값이 같은지 체크
    }
}
```

### SpringBoot Test

- @SpringBootTest 어노테이션의 설정만으로 Boot의 Configuration을 자동으로 설정 가능.
- @AutoConfiureMockMvc는 Controller 테스트 시 MockMvc를 간편하게 사용할 수 있도록 해준다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class SignControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Before
    public void setUp() throws Exception {
        userJpaRepo.save(User.builder().uid("happydaddy@naver.com").name("happydaddy").password(passwordEncoder.encode("1234")).roles(Collections.singletonList("ROLE_USER")).build());
    }

    @Test
    public void signin() throws Exception {
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("id", "happydaddy@naver.com");
        params.add("password", "1234");
        mockMvc.perform(post("/v1/signin").params(params))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.code").value(0))
                .andExpect(jsonPath("$.msg").exists())
                .andExpect(jsonPath("$.data").exists());
    }

    @Test
    public void signup() throws Exception {
        long epochTime = LocalDateTime.now().atZone(ZoneId.systemDefault()).toEpochSecond();
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("id", "happydaddy_" + epochTime + "@naver.com");
        params.add("password", "12345");
        params.add("name", "happydaddy_" + epochTime);
        mockMvc.perform(post("/v1/signup").params(params))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.code").value(0))
                .andExpect(jsonPath("$.msg").exists());
    }
}
```

### Spring Security Test

- 유저에게 리소스의 사용 권한이 있는지의 상태에 따른 테스트 용도.
- @WithMockUser로 가상의 유저를 만들어 권한 요청 테스트

```java
@Test
@WithMockUser(username = "mockUser", roles = {"ADMIN"}) // 가상의 Mock 유저 대입
    public void accessdenied() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders
                .get("/v1/users"))
                .andDo(print())
                .andExpect(status().isOk())
             .andExpect(forwardedUrl("/exception/accessdenied"));
}
```