# JPA 시작

생성일: 2021년 9월 22일 오후 10:38

### 실습용  H2 데이터베이스 설치와 실행

- [http://www.h2database.com/](http://www.h2database.com/)
- 가볍다.
- 웹용 쿼리툴 제공
- 시퀀스 AUTO INCREAMENT 기능 지원

### 메이븐 소개

- [https://maven.apache.org/](https://maven.apache.org/)
- 자바 라이브러리 빌드 관리

### 프로젝트 생성

**라이브러리 추가 - pom.xml**

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>jpa-basic</groupId>
    <artifactId>ex1-hello-jpa</artifactId>
    <version>1.0.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>16</source>
                    <target>16</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <!-- JPA 하이버네이트 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.5.7.Final</version>
        </dependency>
        <!-- H2 데이터베이스 -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.200</version>
        </dependency>
    </dependencies>

</project>
```

**JPA 설정하기 - persistence.xml**

- JPA 설정파일
- /META-INF/persistence.xml
- persistence-unit name으로 이름 지정
- javax.persistence로 시작: JPA 표준 속성
- hibernate로 시작: 하이버네이트 전용 속성

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

### 데이터 베이스 방언

- JPA는 특정 데이터베이스에 종속 x
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
- 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
- hibernate.dialect 속성에 지정

![Untitled](JPA%20%E1%84%89%E1%85%B5%E1%84%8C%E1%85%A1%E1%86%A8%2070063f2576044aba87add387096af22b/Untitled.png)

### JPA 구동 방식

![Untitled](JPA%20%E1%84%89%E1%85%B5%E1%84%8C%E1%85%A1%E1%86%A8%2070063f2576044aba87add387096af22b/Untitled%201.png)

### 객체와 테이블

- @Entity: JPA가 관리할 객체
- @Id: 데이터베이스 PK와 매핑

```java
package hellojpa;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Member {

    @Id
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

### 회원 저장

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();

try {
	//회원 등록
	Member member = new Member(1L, "hello");
	em.persist(member)
	//조회
	Member member = em.find(Member.class, 1L);
	//삭제
	em.remove(member)
	//수정
	member.setName("hello world");

	tx.commit();
}catch(Exception e) {
	tx.rollback();
}finally {
	em.close();
}

emf.close();
```

- 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에 공유
- 앤티티 매니저는 쓰레드간 공유x (사용하고 버려야 한다.)
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

### JPQL 소개

- JPA를 사용하면 엔티티 객체 중심으로 개발
- 문제는 검색 쿼리
- 검색을 할 때에도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 모든 DB데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에 불러올려면 결국 검색 조건이 포함된 SQL이 필요.
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 유사한 문법
- JPQL은 엔티티 객체를 대상으로 쿼리 → 특정 테이블 내부를 보지 않음.
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 DB에 SQL에 의존 x

```java
List<Member> members = em.createQuery("select m from member as m")
.setFirstResult(1)
.setMaxResult(10)
.getResultList();
```