# AWS EC2

생성일: 2021년 9월 8일 오후 10:00

### Amazon Elastic Compute Cloud

Amazon web services 클라우드에서 확장 가능한 컴퓨팅 용량을 제공한다.

이 서비스를 통해 아마존의 데이터 센터의 서버용 컴퓨터들의 자원을 원격으로 임대하여 사용할 수 있다.

### 장점:

- 용량을 늘리거나 줄일 수 있다.
- 사용한만큼 돈을 지불하므로 저렴하다.
- 사용자가 인스턴스를 완전히 제어할 수 있다.
- 보안 및 네트워크 구성, 스토리지 관리 효율적이다.

cf) AMI(Amazon Machine Image): 서버에 필요한 운영체제와 여러 소프트웨어들이 적절히 구성된 템플릿.

### EC2 구조

**AWS VPC(virtual private network)**

AWS Cloud 내부에서 구성되는 사용자의 AWS 계정 전용 가상 네트워크로 이곳에서 AWS 리소스를 시작할 수 있다. 

AWS VPC는 AWS 클라우드에서 다른 가상 네트워크와 논리적으로 분리되어 있다. IP 주소범위와 VPC 범위를 설정하고 서브넷을 추가하고 보안 그룹을 연결한 다음 라우팅 테이블을 구성한다.

**Public Subnet & Private Subnet**

public subnet: Internet Gateway, ELB , 그리고 public IP/Elastic Ip를 가진 인스턴스를 내부에 가지고 있는다. 특히 public subnet내에 있는 nat instance를 통하여 private subnet내에 있는 instances이 인터넷이 가능하게 한다.

Private Subnet: 기본적으로 외부와 차단되어 있다. Private Subnet내의 인스턴스들은 private ip만을 가지고 있으며 internet inbound/outbound가 불가능 하고 오직 다른 서브넷과의 연결만이 가능.

![https://blog.2dal.com/wp-content/uploads/2017/09/AWS-homelan-1.png](https://blog.2dal.com/wp-content/uploads/2017/09/AWS-homelan-1.png)

### EC2 Instance 접속하기

cf) 윈도우의 경우 putty.exe, puttygen.exe로 접속.

1. SSH 클라이언트를 연다.
2. private key가 있는 파일을 찾는다. <key-name>.pem 파일

```java
// 키를 공개적으로 볼 수 없도록 한다.
chmod 400 <key-name>.pem
//퍼블릭 DNS를 이용하여 인스턴스 연결:
ssh -i "<key-name>.pem" ec2-user@<public-DNS>
```

### SSH config 파일로 더 편하게 접속하기

하나의 컴퓨터에 복수의 SSH 키를 사용할 경우 SSH 접속에 사용한 SSH 명령이 복잡해지기에 SSH config파일을 이용하여 이 문제를 해겨ㄹ.

```java
//ssh 기본 명령어
ssh <사용자ID>@<서버명>
//ssh 키(~/.ssh/id_rsa)가 아닌 다른 ssh 키 파일을 사용해야 한다면 -i 옵션 사용
ssh <사용자ID>@<서버명> -i ~/.ssh/<key-pair>.pem
```

SSH 설정 파일 구성

```java
Host firsthost
    SSH_OPTION_1 custom_value
    SSH_OPTION_2 custom_value
    SSH_OPTION_3 custom_value
Host secondhost
    ANOTHER_OPTION custom_value
```

- Host: ssh 명령에 사용하는 이름.
- Hostname: Host에 지정된 이름이 매핑되는 실제 호스트 명.
- User: 네트워크 커넥션에 사용되는 계정 명
- Port: 원격 ssh 데몬이 사용하는 포트, 기본 값=22
- IdentityFile: Host별로 사용할 키 위치 지정.

결론

```java
//ssh -i "<key-name>.pem" ec2-user@<public-DNS>
ssh aws-practice
//config
Host aws-practice
    HostName <public ip>
    User ec2-user
    IdentityFile <pem 파일 위치>
```

### 인바운드 규칙 편집

ec2 보안 그룹 → 인바운드 룰  → 편집

인바운드 규칙은 인스턴스에 도달하도록 허용된 수신 트래픽을 제어합니다.