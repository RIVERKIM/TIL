# ECS 개념

생성일: 2021년 9월 8일 오후 10:02

### AWS ECS(Elastic Container Service)

AWS ECS는 Cluster에서 Docker Container를 손쉽게 실행, 중지 및 관리할 수 있게 해주는 컨테이너 관리 서비스로, 확장성과 속도가 뛰어나다.

ECS는 두 가지의 시작 유형을 가지고 있다. 관리하는 서버를 사용하지 않는 인프라에서 애플리케이션을 호스팅 할 수 있는 Fargete 시작 유형(serverless hosting), 더 세부적인 제어를 위해 관리하는 서버(aws ec2 instance)를 두는 EC2 시작 유형(EC2 instance hosting)이 있다.

### ECS 사용 시 이점

- 간단한 API 호출을 사용하여 컨테이너 기반 애플리케이션을 시작 및 중지할 수 있다.
- 중앙 집중식 서비스를 사용하여 클러스터 상태를 확인할 수 있다.
- 다수의 친숙한 EC2 기능에 액세스할 수 있다.
- 일관된 배포 및 구축 환경을 생성하고, 배치 및 ETL(Extract-Transform-Load) 워크로드를 관리 및 크기 조정하고, 마이크로 서비스 모델에 정교한 애플리케이션 아키텍처를 구축할 수 있다.

### 작업 명세(Task Definition)과 작업(Task)

작업 명세는 컨테이너로 배포할 어플리케이션에 대한 다양한 파라미터 집합이다. 어플리케이션에 사용할 컨테이너 이미지, 포트 정보, 데이터 볼륨 정보 등을 저장. 작업 명세를 instance화 한 객체를 작업(task)라 한다.

### 클러스터

EC2 리소스의 논리적 그룹으로, 작업(Task)가 실행되는 공간이다.

ECS의 EC2 Launch type의 경우, ECS Container Agent가 실행되는 EC2 인스턴스 집합을 의미한다.

### 컨테이너 에이전트(Container Agent)

컨테이너 에이전트는 ECS 클러스터를 구성하는 인프라 자원 마다 실행 된다.

인프라 자원의 리소스 사용률을 수집하여 ECS에 전달하고,

ECS로 부터의 작업(Task)의 시작/중지 요청을 받아 Agent가 실행중인 인프라에서 수행합니다.

[https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb5IRlB%2FbtqFuV0dDyU%2F8sexjXcw6lKwUVukyqBlqK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb5IRlB%2FbtqFuV0dDyU%2F8sexjXcw6lKwUVukyqBlqK%2Fimg.png)

### 클러스터 템플릿(Launch Type)

- 네트워킹 전용(Fargate Launch Type)

AWS가 관리하는 서버리스 인프라 클러스터 위에 task를 배포.

- EC2 Linux(or window) + 네트워킹(EC2 Launch Type)

ECS Container Agent를 실행하는 EC2 인스턴스들로 구성된 클러스에 task를 배포합니다. EC2 인스턴스의 운영체제를 선택 가능.