# 페이징

### 순수 JPA 페이징과 정렬

**JPA 페이징 리포지토리 코드**

```java
public List<Member> findByPage(int age, int offset, int limit) {
        return em.createQuery(
                "select m from Member m where m.age =:age order by m.username desc", Member.class)
                .setParameter("age", age)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }

    public long totalCount(int age) {
        return em.createQuery("select count(m) from Member m where m.age =: age", Long.class)
                .setParameter("age", age)
                .getSingleResult();
    }
```

**JPA 페이징 테스트 코드**

```java
@Test
    public void paging() {
        //given
        memberJpaRepository.save(new Member("member1", 10));
        memberJpaRepository.save(new Member("member2", 10));
        memberJpaRepository.save(new Member("member3", 10));
        memberJpaRepository.save(new Member("member4", 10));
        memberJpaRepository.save(new Member("member5", 10));

        //page 1 offset = 0, limit = 10, page2 -> offset = 10, limit =10

        int age = 10;
        int offset = 0;
        int limit = 3;

        //when
        List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
        long totalCount = memberJpaRepository.totalCount(age);

        //페이지 계산 공식 적용...
        // totalPage = totalCount / size...
        //마지막 페이지 ...
        //최초 페이지 ...

        //then
        assertThat(members.size()).isEqualTo(3);
        assertThat(totalCount).isEqualTo(5);

    }
```

### 스프링 데이터 JPA 페이징과 정렬

**페이징과 정렬 파라미터**

- org.springframework.data.domain.Sort: 정렬 기능
- org.srpingframework.data.domain.Pageable: 페이징 기능 (내부에 Sort 포함)

**특별한 반환 타입**

- org.springframework.data.domain.Page: 추가 count쿼리 결과를 포함하는 페이징
- org.springframework.data.domain.Slice: 추가 count쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit + 1 조회)
- List(자바 컬렉션): 추가 count 쿼리 없이 결과만 반환

**페이징과 정렬 사용 예제**

```java
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
안함
List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
안함
List<Member> findByUsername(String name, Sort sort);
```

**Page 사용 예제 정의 코드**

```java
public interface MemberRepository extends Repository<Member, Long> {
 Page<Member> findByAge(int age, Pageable pageable);
}
```

**Page 사용 예제 실행 코드**

```java
@Test
    public void paging() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        int age = 10;
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        //when

        //Page<Member> page = memberRepository.findByAge(age, pageRequest);
        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        //반환은 항상 DTO로 해야한다.
        Page<MemberDto> toMap = page.map(member -> new MemberDto(
                member.getId(), member.getUsername(), null));

        //then
        List<Member> content = page.getContent();
        long totalElement =page.getTotalElements();

        assertThat(content.size()).isEqualTo(3);
        //assertThat(page.getTotalElements()).isEqualTo(3);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getTotalPages()).isEqualTo(2);
        assertThat(page.isFirst()).isTrue();
        assertThat(page.hasNext()).isTrue();
    }
```

- Pageable은 인터페이스다. 따라서 실제 사용할 떄는 해당 인터페이스를 구현한 org.springframework.data.domain.PageRequest 객체를 사용
- PageRequest(현재페이지, 조회할 데이터 수, (추가 정렬 정보)) - 페이지는 0부터 시작

**Page 인터페이스**

```java
public interface Page<T> extends Slice<T> {
 int getTotalPages(); //전체 페이지 수
 long getTotalElements(); //전체 데이터 수
 <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

**Slice 인터페이스**

```java
public interface Slice<T> extends Streamable<T> {
 int getNumber(); //현재 페이지
int getSize(); //페이지 크기
int getNumberOfElements(); //현재 페이지에 나올 데이터 수
List<T> getContent(); //조회된 데이터
boolean hasContent(); //조회된 데이터 존재 여부
Sort getSort(); //정렬 정보
boolean isFirst(); //현재 페이지가 첫 페이지 인지 여부
boolean isLast(); //현재 페이지가 마지막 페이지 인지 여부
boolean hasNext(); //다음 페이지 여부
boolean hasPrevious(); //이전 페이지 여부
Pageable getPageable(); //페이지 요청 정보
Pageable nextPageable(); //다음 페이지 객체
Pageable previousPageable();//이전 페이지 객체
<U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

**Count 쿼리 분리**

```java
@Query(value = “select m from Member m”,
 countQuery = “select count(m.username) from Member m”)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

- entry 개수가 많으면 count를 하는데 너무 시간이 오래걸리기 때문에 분리한다.