# JPA Auditing

생성일: 2021년 10월 11일 오전 12:52

> 참고 내용:
[스프링 시큐리티](https://docs.spring.io/spring-security-oauth2-boot/docs/2.0.0.RC2/reference/htmlsingle/#gradle), [스프링 oauth](https://spring.io/guides/tutorials/spring-boot-oauth2/)
[oauth 만들기](https://velog.io/@swchoi0329/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0%EC%99%80-OAuth-2.0%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84)
[oauth 코드](https://github.com/ChoiSeungWon/springboot-webservice)
> 

### 공통 로직

- 도메인들이  공통으로 가지고 있는 필드나 컬럼들이 존재한다.
- 대표적으로 생성일자, 수정일자, 식별자 같은 필드 및 컬럼이 있다.
- 이러한 코드는 결과적으로 중복적인 코드를 만든다.
- 이런 문제를 해결하기 위해 JPA는 Audit이라는 기능을 제공하고 있다.
- Spring Data JPA에서 시간에 대해서 자동으로 값을 넣어주는 기능이다.

### 예제 코드

**build.gradle**

```java
dependencies {
compile('org.springframework.boot:spring-boot-starter-web')
compile('org.projectlombok:lombok')
compile('org.springframework.boot:spring-boot-starter-data-jpa')
}
```

**BaseTimeEntity.java**

```java
@Getter
@MappedSuperclass
@EntityListener(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
	@CreatedDate
	private LocalDateTime createdDate;

	@LastModifiedDate
	privaet LocalDateTime modifiedDate;
}
```

- @MappedSuperclass
    - JPA Entity 클래스들이 해당 추상 클래스를 상속할 경우 createDate, ModifiedDate를 컬럼으로 인식
    - 단 객체 단에서만 그렇고, 실제 db에서는 각각의 데이터를 가지고 있다.
- @EntityListeners(AuditingEntityListener.class)
    - 해당 클래스에 Auditing 기능을 포함
- @CreatedDate
    - Entity가 생성되어 저장될 때 시간이 자동 저장
- @LastModifiedDate
    - 조회한 Entity의 값을 변경할 때 시간이 자동 저장

**Role.java**

```java
@Getter
@RequiredArgsConstructor
public enum Role {
    GUEST("Role_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```

- 스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야 한다.

**User.java**

```java
@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity{

    @Id
    @GeneratedValue
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

- @Builder를 붙이면 Builder 패턴을 사용할 수 있다.

**UserRepository.java**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

- 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메서드이다.