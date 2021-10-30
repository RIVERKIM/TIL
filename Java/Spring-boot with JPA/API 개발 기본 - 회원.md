# API 개발 기본 - 회원

### 회원 등록 API

**회원 등록 API V1**

```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import javax.validation.constraints.NotEmpty;

@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    //문제점: entity 와 API 스펙이 1대 1 매팅이 되어있음. -> entity 변경시 api 스펙이 변경
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```

- V1 문제점:
    - 요청 값으로 Member 엔티티를 직접 받는다.
    - 엔티티에 프레젠테이션 계층을 위한 로직이 추가 된다
    - 엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등)
    - **엔티티를 변경하면 API 스펙이 변한다.**
- 해결책
    - API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다.

**회원 등록 API V2**

```java
@PostMapping("/api/v2/members")
    //DTO를 사용함으로써 Entitiy 변경이 용이
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        @NotEmpty
        private String name;
    }
```

- CreateMemberRequest를 Member 엔티티 대신에 RequestBody와 매핑한다.
- 엔티티와 프레젠테이션 계층을 위한 로직을 분리할 수 있다.
- 엔티티와 API 스펙을 명확히 분리할 수 있다.
- 엔티티가 변해도 API 스펙이 변하지 않는다.

### 회원 수정 API

```java
@PostMapping("/api/v2/members/{id}")
    public UpdateMemberResponse updateMemberV2(@PathVariable("id") Long id,
                                               @RequestBody @Valid UpdateMemberRequest request) {
        //커맨드와 쿼리를 분리해서 작성 update는 변경만 반환 없이.
        memberService.update(id, request.getName());
        Member findMember = memberService.findOne(id);
        return new UpdateMemberResponse(findMember.getId(), findMember.getName());
    }

    @Data
    @AllArgsConstructor
    static class UpdateMemberResponse {
        private Long id;
        private String name;
    }

    @Data
    static class UpdateMemberRequest {
        @NotEmpty
        private String name;
    }
```

- 회원 수정도 DTO를 요청 파라미터에 매핑

```java
public class MemberService {
 private final MemberRepository memberRepository;

@Transactional
    public void update(Long id, String name) {
        Member member = memberRepository.findOne(id);
        member.setName(name);
    }
}
```

- 변경 감지를 이용해서 데이터 수정
- update는 변경만 하고 값의 반환은 안되도록 해야한다.
    - **커맨드와 쿼리를 분리하자.**
    

> 참고: 
PUT 방식은 전체 업데이트를 할 때 사용하는 것이 맞다. 부분 업데이트를 하려면 PATCH를 사용하거나 POST를 사용하는 것이 REST 스타일에 맞다.
> 

### 회원 조회 API

**회원 조회V1**

```java
@GetMapping("/api/v1/members")
    public List<Member> membersV1() {
        return memberService.findMembers();
    }
```

- 문제점
    - 응답 값으로 엔티티를 직접 외부로 노출
    - 기본적으로 엔티티 모든 값이 노출된다.
    - 응답 스펙을 맞추기 위해 로직이 추가된다.(@JsonIgnore, 별도의 뷰 로직 등)
    - 추가로 컬렉션을 직접 반환하면 API 스펙을 변경하기 어렵다. (별도의 Result 클래스 생성으로 해결)
- 결론
    - API 응답 스펙에 맞추어 별도의 DTO를 반환한다.

**회원 조회V2**

```java
@GetMapping("/api/v2/members")
    public Result memberV2() {
        List<Member> findMembers = memberService.findMembers();
        List<MemberDto> collect = findMembers.stream()
                .map(m -> new MemberDto(m.getName()))
                .collect(Collectors.toList());

        return new Result(collect.size(), collect);
    }

    //값을 한번 감싸주어서 처리, 리스트를 바로 반환하면 JSON 배열로 나감.
    @Data
    @AllArgsConstructor
    static class Result<T> {
        int count;
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String name;
    }
```

- Result 클래스로 컬렉션을 감싸서 향후 필요한 필드를 추가할 수 있다.