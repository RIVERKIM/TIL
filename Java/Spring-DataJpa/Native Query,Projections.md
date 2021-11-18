# Native Query + Projections

### Projections

- 엔티티를 대신에 DTO를 편리하게 조회할 때 사용
- 전체 엔티티가 아니라 일부분만 조회하고 싶을 경우

```java
public interface UsernameOnly {
	String getUsername();
}
```

- 조회할 엔티티의 필드를 getter형식으로 지정하면 해당 필드만 선택해서 조회(Projection)

```java
public interface MemberRepository ... {
 List<UsernameOnly> findProjectionsByUsername(String username);
}
```

- 메서드 이름은 자유, 반환 타입으로 인지

```java
@Test
public void projections() throws Exception {
 //given
 Team teamA = new Team("teamA");
 em.persist(teamA);
 Member m1 = new Member("m1", 0, teamA);
 Member m2 = new Member("m2", 0, teamA);
 em.persist(m1);
 em.persist(m2);
 em.flush();
 em.clear();
 //when
 List<UsernameOnly> result =
memberRepository.findProjectionsByUsername("m1");
 //then
 Assertions.assertThat(result.size()).isEqualTo(1);
}
```

**인터페이스 기반 Closed Projections**

- 프로퍼티 형식(getter)의 인터페이스를 제공하면, 구현체는 스프링 데이터 JPA가 제공

```java
public interface UsernameOnly {
 String getUsername();
}
```

**인터페이스 기반 Open Projections**

- 스프링 SpEL 문법도 지원

```java
public interface UsernameOnly {
 @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
 String getUsername();
}
```

- 단 모든 필드 조회해서 계산한다. - 최적화 안됨.

**클래스 기반 Projection**

- 구체적인 DTO 형식도 가능

```java
public class UsernameOnlyDto {
 private final String username;
 public UsernameOnlyDto(String username) {
 this.username = username;
 }
 public String getUsername() {
 return username;
 }
}
```

**동적 Projections**

- Generic type을 주면, 동적으로 프로젝션 데이터 변경 가능

```java
<T> List<T> findProjectionsByUsername(String username, Class<T> type);

List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1",
UsernameOnly.class);
```

**중첩 구조 처리**

```java
public interface NestedClosedProjection {
 String getUsername();
 TeamInfo getTeam();
 interface TeamInfo {
 String getName();
 }
}

select
 m.username as col_0_0_,
 t.teamid as col_1_0_,
 t.teamid as teamid1_2_,
 t.name as name2_2_
from
 member m
left outer join
 team t
 on m.teamid=t.teamid
where
 m.username=?
```

- 프로젝션 대상이 root 엔티티면, JPQL SELECT 절 최적화 가능
- root가 아니면
    - Left Outer join 처리
    - 모든 필드를 select해서 엔티티로 조회한 다음에 계산

**정리**

- 프로젝션 대상이 root 엔티티면 유용하다.
- 프로젝션 대상이 root 엔티티를 넘어가면 JPQL SELECT 최적화가 안된다.
- 실무의 복잡한 쿼리를 해결하기에는 한계가 있다.
- 단순할 때만 사용하고, 복잡하면 QueryDSL 사용

### 네이티브 쿼리

- 가급적 사용하지 않는 것이 좋음

**Projections 활용**

```java
@Query(value = "SELECT m.member_id as id, m.username, t.name as teamName " +
 "FROM member m left join team t",
 countQuery = "SELECT count(*) from member",
 nativeQuery = true)
Page<MemberProjection> findByNativeProjection(Pageable pageable);
```