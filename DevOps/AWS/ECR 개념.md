# ECR

생성일: 2021년 9월 8일 오후 10:00

### ECR(EC2 Container Registry)

개발자가 Docker Container Image를 쉽게 저장, 관리 및 배포할 수 있게 해주는 완전관리형 Docker container Registry이다.

Amazon ECR는 AWS IAM를 사용하여 리소스 기반 권한으로 프라이빗 컨테이너 이미지 리포지토리를 지원한다. 이렇게 하면 지정된 사용자 또는 Amazon EC2 인스턴스가 컨테이너 리포지토리 및 이미지에 엑세스 할 수 있다.

**개발 ~ 배포**

![https://res.cloudinary.com/devdevil/image/upload/v1599449911/illunex_blog/2020-08-27-img01.jpg](https://res.cloudinary.com/devdevil/image/upload/v1599449911/illunex_blog/2020-08-27-img01.jpg)

- 코드를 작성
- 저장소에 저장
- 서버를 실행

여기서 저장소 역할이 ECR의 역할, Docker Private Repository를 구축하고 관리하는 수고를 AWS에 맡기는 것.

이렇게 하면 AWS EC2 인스턴스가 컨테이너 리포지토리 및 이미지에 엑세스 가능.

Container 이미지를 s3에 저장하기 때문에 고가용성이 유지되고, AWS IAM 인증을 통해 이미지 Pull/Push 에 대한 권한 관리가 가능.

### ECR IAM 권한 추가

ECR 서비스를 이용하려면 IAM 사용자에게 ECR 접근 권한을 주어야 한다. 그래서 IAM사용자에게 AmazonEC2ContainerRegistryFullAccess 권한을 준다.

### ECR Repository 생성

ECR → Repository 생성 → default 로 생성

cf) Amazon linux2를 실행하는 Ec2인스턴스 에서는 amazon-linux-extras 명령어로 소프트웨어 패키지 설치.

```java
//extras topic 목록 확인
amazon-linux-extras list
//extras topic 설치 가능토록 (비)활성화
amazon-linux-extras enable or disable <name>
//extras topic 설치
amazon-linux-extras install <name>
```

1. EC2 Docker 설치

```java
sudo yum update -y
sudo amazon-linux-extras install -y docker
sudo service docker start
yum list installed | grep docker
```

1. Push 명령에 있는 것들을 하나씩 입력

```java
//인증 토큰을 검색하고 레지스트리에 대해 Docker 클라이언트를 인증합니다.
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 608823332223.dkr.ecr.ap-northeast-2.amazonaws.com
```

> Got Permission denied 에러 발생

```java
sudo groupadd docker
sudo usermod -aG docker ${USER}
newgrp docker
```

1. Dockerfile 만들기

```java
touch Dockerfile

//Dockerfile
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello World!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```

1. Build the docker image from Dockerfile

```java
docker build -t garam .
//빌드 잘 됐는지 확인
docker images --filter reference=garam
```

1. 빌드가 완료되면 이미지에 태그를 지정하여 이 리포지토리에 푸시할 수 있습니다

```java
docker tag garam:latest 608823332223.dkr.ecr.ap-northeast-2.amazonaws.com/garam:latest
```

1. 다음 명령을 실행하여 이 이미지를 새로 생성한 AWS 리포지토리로 푸시합니다.

```java
docker push 608823332223.dkr.ecr.ap-northeast-2.amazonaws.com/garam:latest
```

EC2 인스턴스에서 docker image 가져올 때

```java
docker pull <ECR-image url>
```
