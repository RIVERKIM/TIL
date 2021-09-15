# ECS 사용법

생성일: 2021년 9월 15일 오전 12:38

### IAM User 생성

> [가이드](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)

### 보안그룹 생성

보안그룹이란 EC2 인스턴스의 포트포워딩 규칙 그룹 이다.

EC2 서비스 콘솔에서 HTTP(80), HTTPS(443) 인입 허용.

### EC2 Launch Type ECS 클러스터 생성

ECS → 클러스터 → 클러스터 생성 → EC2 Linux + 네트워킹 → 이름 선택하고 default 설정으로 생성.

### ECS 클러스터의 EC2 인스턴스의 보안 그룹 변경

보안 그룹 → EC2ContainerService ~~ → 인바운드 규칙 편집 → 22포트 ssh 추가

### 작업 정의 생성

ECS →작업 정의 → 새 작업 정의 생성 → EC2 → 작업 정의 (json 파일 입력)

```java
//작업 정의
{
    "containerDefinitions": [
        {
            "entryPoint": [
                "sh",
                "-c"
            ],
            "portMappings": [
                {
                    "hostPort": 80,
                    "protocol": "tcp",
                    "containerPort": 80
                }
            ],
            "command": [
                "/bin/sh -c \"echo '<html> <head> <title>Amazon ECS Sample App</title> <style>body {margin-top: 40px; background-color: #333;} </style> </head><body> <div style=color:white;text-align:center> <h1>Amazon ECS Sample App</h1> <h2>Congratulations!</h2> <p>Your application is now running on a container in Amazon ECS.</p> </div></body></html>' >  /usr/local/apache2/htdocs/index.html && httpd-foreground\""
            ],
            "cpu": 10,
            "memory": 300,
            "image": "httpd:2.4",
            "name": "simple-app"
        }
    ],
    "family": "console-sample-app-static"
}
```

작업 정의는 아직 인스턴스화 된 작업 객체가 아니다.  따라서 이 작업 정의를 작업으로 인스턴스화하여 관리해주는 서비스 객체를 배포해야 한다.

### 서비스 배포

서비스란 특정 작업 인스턴스가 일정 갯수를 항상 유지할 수 있도록 관리해주는 객체이다.

따라서 하나의 인스턴스에 장애가 발생해 다운되더라도, 자동으로 새로운 인스턴스를 생성해준다. 

ECS → 클러스터 → 서비스 → 서비스 생성, 전부 default

### 서비스 보기

ECS → Cluster → 서비스 → Tasks → IPv4 주소로 접속.

### 서비스 정리

ECS → Cluster → 서비스삭제 → 클러스터 삭제.