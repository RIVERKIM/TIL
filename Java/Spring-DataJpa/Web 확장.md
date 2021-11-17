# Web 확장

### 도메인 클래스 컨버터

- HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩

```java
@RestController
@RequiredArgsConstructor
public class MemberController {
 private final MemberRepository memberRepository;
 @GetMapping("/members/{id}")
 public String findMember(@PathVariable("id") Member member) {
 return member.getUsername();
 }
}
```

- HTTP 요청은 회원 Id도 받지만 도메인 컨버터 클래스가 중간에 동작해서 회원 엔티티 객체를 반환
- 도메인 클래스 컨버터도 리파지토리를 사용해서 엔티티를 찾음.

> 주의: 파라미터로 받은 엔티티는 단순 조회용으로만 사용해야한다.
(트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영 x)
> 

### 페이징과 정렬

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
 Page<Member> page = memberRepository.findAll(pageable);
 return page;
}
```

- 파라미터로 Pageable을 받을 수 있다.
- Pageable은 인터페이스, 실제는 org.springframework.data.domain.PageRequest 객체 생성

**요청 파라미터**

- 예) /members?page=0&size=3&sort=id,desc&sorc=username,desc
- page: 현재 페이지, 0부터 조회
- size: 한 페이지에 노출할 데이터 건수
- sort: 정렬 조건을 정의한다.

**기본값**

```java
spring.data.web.pageable.default-page-size=20 /# 기본 페이지 사이즈/
spring.data.web.pageable.max-page-size=2000 /# 최대 페이지 사이즈/
```

**개별 설정**

- @PageableDefault 어노테이션 사용

```java
@RequestMapping(value = "/members_page", method = RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = “username”,
 direction = Sort.Direction.DESC) Pageable pageable) {
 ...
}
```

**접두사**

- 페이징 정보가 둘 이상이면 접두사로 구분
- @Qualifier에 접두사명 추가 "{접두사명}_xxx"
- 예제: /members?member_page=0&order_page=1

```java
public String list(
 @Qualifier("member") Pageable memberPageable,
 @Qualifier("order") Pageable orderPageable, ...
```

Page 내용을 DTO로 변환하기

- 엔티티를 API로 노출하면 다양한 문제 발생하기에 반드시 DTO로 변환
- Page는 map()을 지원해서 내부 데이터를 다른 것으로 변경 가능

**MemberDto**

```java
@Data
public class MemberDto {
 private Long id;
 private String username;
 public MemberDto(Member m) {
 this.id = m.getId();
 this.username = m.getUsername();
 }
}
```

Page.map() 사용

```java
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
 return memberRepository.findAll(pageable).map(MemberDto::new);
}
```

**Page를 1부터 시작하기**

- 스프링 데이터는 Page를 0부터 시작
1. Pageable, Page를 파리미터와 응답 값으로 사용히지 않고, 직접 클래스를 만들어서 처리한다. 그리고 직접 PageRequest(Pageable 구현체)를 생성해서 리포지토리에 넘긴다. 물론 응답값도 Page 대신에 직접 만들어서 제공해야 한다.
2. spring.data.web.pageable.one-indexed-parameters 를 true 로 설정한다. 그런데 이 방법은 web에서 page 파라미터를 -1 처리 할 뿐이다. 따라서 응답값인 Page 에 모두 0 페이지 인덱스를 사용하는 한계가 있다