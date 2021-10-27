# NAT 서버 생성

### Public Subnet에 EC2인스턴스로 NAT 서버 구축

Private Subnet내의 웹서버들이 외부로 통신하기 위해서 보통 NAT서버를 Public Subnet안에 구축하여 NAT를 통해 외부로 통신을 한다.

**EC2 인스턴스 생성**

1. EC2 → 인스턴스 시작하기
2. Amazon Community → nat들어간 AMI 선택 → t2-micro 선택
3. 인스턴스 세부 구성 정보 → network (tulip-vpc), subnet (public1 2a), 우발적 종료로부터 보호 체크
4. 저장공간 → 루트 볼륨만, Volume Type (General purpose SSD)
5. 태그 → key: Tulip1 , value: Nat  전부 체크
6. 보안 그룹
    1. AWS의 Security Group에 대한 기본 정책은 Inbound Deny All / Outbound Any Open 입니다 (Outbound Traffic에 대한 정책설정은 보안그룹에서 할수 없고 Network ACL기능을 사용해야 합니다. 그러므로 허용할 Protocol, Port, IP를 지정하여 Inbound traffic에 대한 정책을 허용해 줍니다.
    2. 모든 트래픽, 모두 0-65535, 사용자 지정, 172.16.0.0/24
    3. 사용자 지정 UDP, UDP, 1194, 위치 무관, 0.0.0.0/0, ::0
7. 인스턴스 만들기

**탄력적 IP 생성 후 EC2 인스턴스에 할당**

인스턴스가 만들어졌지만, Public IP와 Elastic IP가 지정돼있지 않았따. 외부 통신을 위해 Public IP 지정

1. 탄력적 IP → 생성 → 작업 → 할당

**NAT 인스턴스 주소를 Private Subnet을 위한 Route에 연결 시켜 주기**

1. VPC → Route table → private route → Routes → 편집
2. 0.0.0.0/0, Nat 인스턴스 설정 후 저장
3. EC2 인스턴스 → 작업 → 네트워킹 → 소스 대상 확인 /변경 → 비활성화