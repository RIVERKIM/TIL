# 수정, EntityGraph, JpaHint 등

### 벌크성 수정 쿼리

**JPA를 사용한 벌크성 수정 쿼리**

```java
public int bulkAgePlus(int age) {

        return em.createQuery("update Member m set m.age = m.age + 1" +
                        "where m.age >= :age")
                .setParameter("age", age)
                .executeUpdate();
    }
```

**스프링 데이터 JPA를 사용한 벌크성 수정 쿼리**

```java
@Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
```

- 벌크성 수정, 삭제 쿼리는 @Modifying 어노테이션 사용
    - 사용하지 않으면 다음 예외 발생
- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화:  @Modifying(clearAutomatically = true)

> 참고: 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티의 상태가 달라질 수 있다.
1. 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산을 먼저 수행한다.
2. 벌크 연산 직후 영속성 컨텍스트를 초기화 한다.
> 

**@EntityGraph**

- 연관된 엔티티들을 SQL 한번에 조회하는 방법
- member → team은 지연 로딩 관계이다. 따라서 team의 데이터를 조회할 때마다 쿼리가 실행(N + 1) 문제

**JPQL 패치 조인**

```java
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();
```

- 스프링 데이터 JPA는 JPA가 제공하는 엔티티 그래프 기능을 편리하게 사용하게 도와준다. 이 기능을 사용하면 JPQL 없이 패치 조인을 사용할 수 있다.

**Entity Graph**

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();
//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();
//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username)
```

**EntityGraph 정리**

- 패치 조인의 간편한 버전
- Left outer join 사용

**NamedEntityGraph 사용 방법**

```java
@NamedEntityGraph(name = "Member.all", attributeNodes =
@NamedAttributeNode("team"))
@Entity
public class Member {}

@EntityGraph("Member.all")
@Query("select m from Member m")
List<Member> findMemberEntityGraph();
```

### JPA Hint

- JPA 쿼리 힌트(SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트)

**쿼리 힌트 사용**

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value =
"true"))
Member findReadOnlyByUsername(String username);
```

**쿼리 힌트 사용 확인**

```java
@Test
public void queryHint() throws Exception {
 //given
 memberRepository.save(new Member("member1", 10));
 em.flush();
 em.clear();
 //when
 Member member = memberRepository.findReadOnlyByUsername("member1");
 member.setUsername("member2");
 em.flush(); //Update Query 실행X
}
```

**쿼리 힌트 Page 추가 예제**

```java
@QueryHints = (value = {@QueryHint(name = "org.hibernate.readOnly", 
value = "true")}, forCounting = true)
Page<Member> findByUsername(String name, Pageable pageable);
```

- forCounting: 반환 타입으로 Page 인터페이스를 적용하면 추가로 호출하는 페이징을 위한 count 쿼리도 쿼리 힌트 적용

**Lock**

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByUsername(String name);
```