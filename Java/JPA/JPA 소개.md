# JPA 소개

생성일: 2021년 9월 22일 오후 10:38

### SQL 중심적인 개발의 문제점

**무한 반복, 지루한 코드**

자바 객체를 SQL로 SQL을 자바 객체로 변경

```java
public class Member {
	private String memberId;
	private String name;
}

INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES
SELECT MEMBER_ID, NAME FROM MEMBER M
```

**SQL에 의존적인 개발을 피하기 어렵다. - 객체 vs 관계형 DB 패러다임의 불일치**

1. 서로 다른 상속 관계

![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled.png)

- 저장: 객체를 저장하기 위해서 객체를 분해해서 각각의 테이블에 넣어야 한다.
- 조회
    - 각각의 테이블에 따른 조인 SQL 작성
    - 각각의 객체 생성
    - 그래서 DB에 저장할 객체에는 상속 관계를 쓰지않음.

1. 연관 관계
- 객체는 참조를 사용: member.getTeam()
- 테이블은 외래 키를 사용: JOIN ON [M.TEAM](http://m.TEAM)_ID = T.TEAM_ID//

```java
// 객체를 테이블에 맞추어 모델링
class Member {
	String id; // MEMBER_ID 컬럼
	Long teamId; //TEAM_ID FK 컬럼
	String username; // USERNAME 컬럼
}

class Team {
	Long id; //TEAM_ID pk 사용
	String name; // NAME 컬럼
}
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES ....

//객체다운 모델링
class Member {
	String id; // MEMBER_ID 컬럼
	Team team; //TEAM_ID FK 컬럼
	String username; // USERNAME 컬럼
	
	Team getTeam() {
		return team;
	}
}
class Team {
	Long id; //TEAM_ID pk 사용
	String name; // NAME 컬럼
}
//member.getTeam().getId();
INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES

//객체 모델링 조회
SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

public Member find(String MemberId) {
	//...sql 실행

	//데이터베이스에서 조회한 회원 관련 정보 입력
	Member member = new Member();

	//데이터베이스에서 조회한 팀 관련 정보를 모두 입력
	TEAM team = new Team();

	Member.setTeam(team);
	retrun member;
}
```

- 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다. 하지만 처음 실행하는 SQL에 따라 탐색 범위가 결정되어 버린다.

```java

SELECT M.*, T.*
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

member.getTeam(); //ok
member.getOrder(); // null
```

**엔티티 신뢰 문제**

```java
class MemberService {
 ...
 public void process() {
 Member member = memberDAO.find(memberId);
 member.getTeam(); //???
 member.getOrder().getDelivery(); // ???
 }
}

memberDAO.getMember(); //Member만 조회
memberDAO.getMemberWithTeam();//Member와 Team 조회
//Member,Order,Delivery
memberDAO.getMemberWithOrderWithDelivery();
```

- 모든 객체를 미리 로딩할 수는 없다.
- 상황에 따라 동일한 회원 조회 메서드를 여러벌 생성

**계층형 아키텍쳐 진정한 의미의 계층 분할이 어렵다.**

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
member1 == member2; //다르다.
class MemberDAO {

 public Member getMember(String memberId) {
 String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
 ...
 //JDBC API, SQL 실행
 return new Member(...);
 }
}
```

- 객체답게 모델링 할수록 매핑 작업만 늘어난다.

**객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까?**

### JPA (Java Persistence API)

자바 진영 ORM 표준 기술

**ORM**

- Object-relational mapping(객체 관계 매핑)
- 객체는 객체대로 설계
- 관계형 데이터베이스는 관계형 데이터베이스로 설계
- ORM 프레임워크가 중간에서 매핑
- 대중적인 언어에는 대부분 ORM 기술이 존재

**JPA는 애플리케이션과 JDBC 사이에서 동작**

![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%201.png)

- 저장

![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%202.png)

- 조회

![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%203.png)

**JPA는 표준 명세**

- JPA는 인터페이스의 모음
- JPA 2.1 표준 명세를 구현한 3가지 구현체
- Hibernate, EclipseLink, DataNucleus

**JPA 사용 이유**

- SQL 중심적인 개발에서 객체 중심으로 개발
- 생산성: JPA와 CRUD
    - 저장: jpa.persist(member)
    - 조회: Member member = jpa.find(memberId)
    - 수정: member.setName("")
    - 삭제: jpa.remove(member)
- 유지보수: 필드만 추가하면 SQL은 JPA가 처리
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준
- **패러다임 불일치 해결**
    - JPA와 상속

    ![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%204.png)

    - JPA와 연관관계
    - JPA와 객체 그래프 탐색

    ![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%205.png)

    - 신뢰할 수 있는 엔티티 계층

    ```java
    class MemberService {
     ...
     public void process() {
     Member member = memberDAO.find(memberId);
     member.getTeam(); //자유로운 객체 그래프 탐색
     member.getOrder().getDelivery();
     }
    }

    //동일한 트랜잭션에서 조회한 엔티티는 같음을 보장
    String memberId = "100";
    Member member1 = jpa.find(Member.class, memberId);
    Member member2 = jpa.find(Member.class, memberId);
    member1 == member2; //같다.
    ```

**JPA의 성능 최적화 기능**

- 1차 캐시와 동일성 보장
    - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
    - DB Isolation Level이 Read Commit이어도 애플리케이션에 Repeatable Read 보장

    ```java
    String memberId = "100";
    Member m1 = jpa.find(Member.class, memberId); //SQL
    Member m2 = jpa.find(Member.class, memberId); //캐시
    println(m1 == m2) //true
    //SQL 한번만 실행
    ```

- 트랜잭션을 지원하는 쓰기 지연
    - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
    - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

    ```java
    transaction.begin(); // [트랜잭션] 시작
    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);
    //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
    //커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
    transaction.commit(); // [트랜잭션] 커밋
    ```

- 지연 로딩

![Untitled](JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20da62ba2dac4e4b4c8cfed035d62859f5/Untitled%206.png)