# Auditing

- 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적할 때 사용.
    - 등록자
    - 수정자
    - 등록일
    - 수정일

**순수 JPA 코드**

```java
package study.datajpa.entity;
@MappedSuperclass
@Getter
public class JpaBaseEntity {
 @Column(updatable = false)
 private LocalDateTime createdDate;
 private LocalDateTime updatedDate;
 @PrePersist
 public void prePersist() {
 LocalDateTime now = LocalDateTime.now();
 createdDate = now;
 updatedDate = now;
 }
 @PreUpdate
 public void preUpdate() {
 updatedDate = LocalDateTime.now();
 }
}

//Member에 위의 클래스를 확장하도록
public class Member extends JpaBaseEntity {}
```

**순수 JPA 테스트 코드**

```java
@Test
public void JpaEventBaseEntity() throws Exception {
 //given
 Member member = new Member("member1");
 memberRepository.save(member); //@PrePersist
 Thread.sleep(100);
 member.setUsername("member2");
 em.flush(); //@PreUpdate
 em.clear();
 //when
 Member findMember = memberRepository.findById(member.getId()).get();
 //then
 System.out.println("findMember.createdDate = " +
findMember.getCreatedDate());
 System.out.println("findMember.updatedDate = " +
findMember.getUpdatedDate());
}
```

JPA 주요 이벤트 어노테이션

- @PrePersist, @PostPersist
- @PreUpdate, @PostUpdate

**스프링 데이터 JPA 사용**

- @EnableJpaAuditing → 스프링 부트 설정 클래스에 적용
- @EntityListeners(AuditingEntityListener.class) → 엔티티에 적용
- 어노테이션
    - @CreatedDate
    - @LastModifiedDate
    - @CreatedBy
    - @LastModifiedBy
    

**스프링 데이터 Auditing 적용 - 등록일, 수정일**

```java
package study.datajpa.entity;
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity {
 @CreatedDate
 @Column(updatable = false)
 private LocalDateTime createdDate;
 @LastModifiedDate
 private LocalDateTime lastModifiedDate;
}
```

**스프링 데이터 Auditing 적용 - 등록자, 수정자**

```java
package jpabook.jpashop.domain;
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseEntity {
 @CreatedDate
 @Column(updatable = false)
 private LocalDateTime createdDate;
 @LastModifiedDate
 private LocalDateTime lastModifiedDate;
 @CreatedBy
 @Column(updatable = false)
 private String createdBy;
 @LastModifiedBy
 private String lastModifiedBy;
}
```

- 등록자, 수정자를 처리해주는 AuditingAware 스프링 빈 등록

```java
@Bean
public AuditorAware<String> auditorProvider() {
//실무에서는 세션 정보나 스프링 시큐리티 로그린 정보에서 ID를 받음.
	return () -> Optional.of(UUID.randomUUID().toString());
}
```

> 참고: 실무에서 대부분의 엔티티는 등록시간, 수정시간이 필요하지만, 등록자, 수정자는 없을 수도 있다. 따라서 다음과 같이 Base타입을 분리하고, 원하는 타입을 상속해서 처리
> 

```java
public class BaseTimeEntity {
 @CreatedDate
 @Column(updatable = false)
 private LocalDateTime createdDate;
 @LastModifiedDate
 private LocalDateTime lastModifiedDate;
}
public class BaseEntity extends BaseTimeEntity {
 @CreatedBy
 @Column(updatable = false)
 private String createdBy;
 @LastModifiedBy
 private String lastModifiedBy;
}
```