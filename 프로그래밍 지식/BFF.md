# BFF

생성일: 2021년 8월 30일 오후 6:27

## BFF(Backend For Frontend)

기존의 Monolitic 방식은 backend가 어떤 종류의(mobile, browser) frontend인지 관계 없이 모든 요청을 받아서 처리하는 형식이였다.

![Untitled](BFF%200339cc02a1224f01b4fc4c896b44b5c7/Untitled.png)

하지만 위와 같은 구조는 다양한 End point를 처리하기 위한 통합된 API를 써야하기에 public api 영역이 단일 장애 포인트가 된다. 이와 같은 경우를 API Gateway라고 한다.

위와 같은 문제를 해결하기 위해 각각의 End point를 위한 backend가 중간 매개 서버로 등장한다.

BFF에서 backend는 frontend의 modality(인터랙션 과정에서 사용되는 의사소통 채널)에 의해 구분된다.

따라서 각 modality에 따른 요구 사항을 별도로 구현하도록 할 수 있다.

![Untitled](BFF%200339cc02a1224f01b4fc4c896b44b5c7/Untitled%201.png)

### 장점

- Monolitic API 보다 작고 계산 복잡성이 적기에 호율적이다.
- 각 front end에 최적화 된 solution 제공 가능.

### 단점

- Fanout: 어느 하나 BFF가 문제가 발생하면 해당 interface 전체 BFF가 중단될 수 있음.

    ![Untitled](BFF%200339cc02a1224f01b4fc4c896b44b5c7/Untitled%202.png)

- Fuse: 각 서비스가 여러 BFF에 응답하여 과부하가 발생할 수 있다.

![Untitled](BFF%200339cc02a1224f01b4fc4c896b44b5c7/Untitled%203.png)

> [참고 사이트](https://akfpartners.com/growth-blog/backend-for-frontend)

erererer