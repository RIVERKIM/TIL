# CI/CD pipeline + Jenkins

생성일: 2021년 9월 2일 오후 11:19

### CI/CD

Continuous Integration ⇒ 여러 개발자들의 코드베이스를 계속해서 통합하는 것.

Continuous Delivery ⇒ 사용자에게 제품을 서비스를 지속적으로 배달한다.

코드베이스가 항상 배포가능한 상태를 유지하는 것.

Continuous Deployment ⇒ 코드베이스를 사용자가 사용가능한 환경에 배포하는 것을 자동화한다. (일반적으로 버전을 바꾸더라도 끊기지않게 서비스 → 보통 클라우드 서비스에서 해준다. ex) Ecs )

궁극적 목적: 코드만 작성했는데, 이미 누군가 이 서비스를 이용하고 있다.

개발자는 코드만 짜도록 귀찮은 작업은 자동화 하자.

즉, CI/CD란 각각의 개발자들이 개발을 하는 개발 환경을 사용자가 사용 가능한 서비스로 전달하는 모든 과정을 지속 가능한 형태로 또 가능하다면 자동으로 해서 개발자와 사용자 사이의 격차를 없애는 것이다. 이러한 과정에는 코드를 빌드하고, 테스트하고 배포하는 활동이 있다.

![https://blog.kakaocdn.net/dn/nL25N/btqFtwGIka1/kuiJPrwJmBHuDij2O4KjaK/img.png](https://blog.kakaocdn.net/dn/nL25N/btqFtwGIka1/kuiJPrwJmBHuDij2O4KjaK/img.png)

### 젠킨스

java runtime위에서 동작하는 자동화 서버. 빌드, 테스트, 배포 등 모든 것을 자동화 해주는 자동화 서버.

- java runtime environment에서 동작
- 다양한 플러그인들을 활용해서 각종 자동화 작업을 처리할 수 있음.
- 일련의 자동화 작업의 순서들의 집합인 Pipeline을 통해 CI/CD파이프라인을 구축함.

 

#Plugin

대표적인 것: Credentials Plugin, Git plugin, Pipeline 처음에는 remmend해주는 걸 깔면 된다.

Credential Plugin

Jenkins는 그냥 단지 서버일뿐이기 때문에 배포에 필요한 각종 리소스(가령 클라우드 리소스 호근 베어메탈에 ssh접근 등) 에 접근하기 위해서는 여러가지 중요한 정보들을 저장하고 있어야 한다. 

이런 중요한 정보(AWS token, git access token, etc...) 들을 저장해주는 플러그인

Docker Plugin and Docker pipeline

Docker agent를 사용하고 jenkins에서 도커를 사용하기 위함.

**Pipeline**

파이프라인이란 CI/CD 파이프라인을 젠킨스에 구현하기 위한 일련의 플러그인들의 집합이자 구성.

즉 여러 플러그인들을 이 파이프라인에서 용도에 맞게 사용하고 정의함으로써 파이프라인을 통해 서비스가 배포됨.

Pipeline DSL(Domain Specific Language)로 작성.

최신버전 Declaritive Pipeline syntax

**Pipeline Syntax**

Sections

- Agent section
- Post section
- Stages section
- Steps section

**Agent Section**

젠킨스는 많은 일을 해야 하기 때문에 혼자 하기 버겁다.

여러 slave node를 두고 일을 시킬 수 있는데, 이처럼 어떤 젠킨스가 일을 하게 할 것인지를 지정한다.

젠킨스 노드 관리에서 새로 노드를 띄우거나 혹은 docker 이미지등을 통해서 처리할 수 있음.

ex) 자바 빌드할 때, 노드에서 자바 도커 이미지를 받아서 그 안에서 빌드하도록 명령.

**Post Section**

스테이지가 끝난 이후의 결과에 따라 후속 조치를 취할 수 있다.

Ex) success, failure, always, cleanup

Ex) 성공시에 성공 이메일 등등

**Stages Section**

어떤 일들을 처리할 건지 일련의 stage를 정의함.

**Steps Section**

한 스테이지 안에서의 단계로 일련의 스탭을 보여줌.

Steps 내부는 여러가지 스텝들로 구성

여러 작업들을 실행가능

플러그인을 깔면 사용할 수 있는 스텝들이 생겨남.

**Declaratives → 각 stage안에서 어떤 일들을 할지 정의**

Environment, stage, options, parameters, triggers, when 등

Environment → 어떤 pipeline 이나 stage scope의 환경 변수 설정

Parameter → 파이프라인 실행시 파라미터 받음

Triggers → 어떤 형태로 트리거 되는가

When → 언제 실행되는가 

**개발 환경 종류**

- 개발자가 개발을 하는 Local 환경(자신의 workspace)
- 개발자들끼리 개발 내용에 대한 통합 테스트를 하는 Development 환경
- 개발이 끝나고 QA 엔지니어 및 내부 사용자들이 사용해 보기 위한 QA 환경
- 실제 유저가 사용하는 production 환경

**개발 프로세스**

1. 개발자가 자신의 pc에서 개발을 진행
2. 다른 개발자가 작성한 코드와 차이가 발생하지 않는지 내부테스트를 진행(git hook)
3. 진행한 내용을 다른 개발자들과 공유하기 위해 git과 같은 SCM에 올린다.

⇒ 흔히 dev 브랜치

1. Dev 브랜치의 내용을 개발 환경에 배포하기 전에 테스트와 Lint등 코드 포맷팅을 한다.
2. 배포하기 위한 빌드 과정을 거친다.
3. 코드를 배포한다.
4. 테스트를 진행한다.
5. 위 모든 과정을 DEV, QA, PROD 환경에서 모두 하고 각각에 맞는 환경에 배포.

**여러 배포 환경의 관리**

여러 배포환경의 관리에서 핵심은

인프라를 모듈화 하여 어떤것이 변수인지 잘 설정하고 이를 잘 설계하는 것.

ex) APP_ENV 처럼 현재 배포하고자 하는 것이 무슨 환경인지 설정하고 앱 내에서 사용하는 다양한 변수들을 APP_ENV에 맞게 잘 가져다 쓰는것이 핵심.

서비스 내부의 변수뿐만 아니라 클라우드 리소스를 많이 활용해서 개발하는 요즘에는 클라우드 리소스 내에서 인프라별 키관리가 매우 중요해서 aws system manager의 parameter store와 같은 키 관리 서비스를 쓰는 것을 추천.

각각의 배포 환경에 따른 젠킨스가 있고, 애들이 필요한 정보를 어디서 가져오느냐

### 실습

1. 웹사이트 코드 작성
2. 웹사이트 코드를 린트, 웹팩 빌드 해서 AWS s3 bucket에 html 파일을 업로드
3. Node.js 백엔드 코드를 typescript로 작성
4. 위 코드를 javascript compile 하고, 테스트 코드를 돌려서 도커 이미지를 만들어 ECR에 올린다.
5. 업로드한 ECR 이미지로 ECS 서비스를 재시작한다.(rolling deploy-무중단 배포) → continuous deploy.

어느 정도 규모 있는 서비스에서는 DEV용 계정 ,QA계정, PROD 계정 따로 써서 network interface도 달라서 아예 물리적으로 분리를 시켜서 개발. → 젠킨스 여러개 사용.

![Untitled](CI%20CD%20pipeline%20+%20Jenkins%20c817a934fb084cbc9b0301b2cd8ccd1b/Untitled.png)