# 프로젝트 환경 설정

생성일: 2021년 9월 15일 오전 12:41

### 스프링 부트 스타터 사이트에서 프로젝트 생성

[https://start.spring.io/](https://start.spring.io/)

프로젝트 선택

- Project: Gradle project
- Spring boot: 2.5.4
- Language: java
- Packaging: jar
- java 16

Project metadata

- groupId: hello
- artifactId: hello-spring

Dependencies: Spring web, Thymeleaf(Html template)

### Gradle 전체 설정

```java
//build.gradle
plugins {
	id 'org.springframework.boot' version '2.5.4'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '16'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}
```

### 라이브러리 살펴보기

> Gradle은 의존 관계가 있는 library를 다운로드 한다. 라이브러리를 다운로드 할 때 그 라이브러리와 의존 관계가 있는 다른 라이브러리를 다운.

**스프링 부트 라이브러리**

- spring-boot-starter-web
    - spring-boot-starter-tomcat: 톰캣(웹서버)
    - spring-webmvc: 스프링 웹 MVC
- spring-boot-starter-thymeleaf: 타임리프 템플릿 엔진(view)
- spring-boot-starter( 공통): 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
        - logback, slf4j

**테스트 라이브러리**

- spring-boot-starter-test
    - junit: 테스트 프레임워크
    - mockito: 목 라이브러리
    - assertj: 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test: 스프링 통합 테스트 지원

### View 환경 설정

- 스프링 부트가 제공하는 Welcome page 기능
    - static/index.html 올려두면 Welcome page 기능 제공.
    - [https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features)
- thymeleaf 템플릿 엔진
    - [공식 사이트](https://www.thymeleaf.org/)
    - [스프링 공식 튜토리얼](https://spring.io/guides/gs/serving-web-content/)

### 간단한 예제

```java
//HelloController.java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {

    @GetMapping("hello")
    public String hell(Model model) {
        model.addAttribute("data", "Hello");
        return "hello";
    }
}
<--------------------------->
//hello.html
<!DOCTYPE html>
<html xmlns:th="https://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content= "text/html; charset=UTF-8">
    <title>Hello</title>
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}">안녕하세요 손님</p>
</body>
</html>
```

![Untitled](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%20%E1%84%92%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A7%E1%86%BC%20%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8C%E1%85%A5%E1%86%BC%20c9784e25f4e94442859fe7345115ca18/Untitled.png)

- 컨트롤러에서 리턴 값으로 문자를 반환하면 뷰 리졸버(viewResolver)가 화면을 찾아서 처리한다.
    - 스프링 부트 템플릿엔진 기본 viewName 매핑
    - resources:template/ + {ViewName} + .html

> spring-boot-devtools 라이브러리를 추가하면, html 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능하다.

### 빌드하고 실행하기

- 콘솔로 이동 명령 프롬프트(cmd)로 이동
- gradlew.bat 를 실행하면 됩니다.
- 명령 프롬프트에서 gradlew.bat 를 실행하려면 gradlew 하고 엔터를 치면 됩니다.

```java
./gradlew.bat build or (./gradlew.bat clean build)
cd build/libs
java -jar hello-spring-0.0.1-SNAPSHOT.jar
```