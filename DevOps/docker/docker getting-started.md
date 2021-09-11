# docker getting-started

생성일: 2021년 9월 9일 오후 6:52

### 설정

```java
docker run -dp 80:80 docker/getting-started
```

Build app Container image

```java
//Dockerfile
FROM node:12-alpine // docker base image
RUN apk add -no-chche python g++ make 
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"] // the default command to run when starting a container from the image

//docker image build 
// -t flag tags our image
// . docker should llok for the dockerfile in the current directory
docker build -t getting-started .

```

### Docker hub에 올리기

1. [Sign up](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade) and share images using Docker Hub.
2. Sign in to [Docker Hub](https://hub.docker.com/).
3. Click the **Create Repository** button.
4. For the repo name, use `getting-started`. Make sure the Visibility is `Public`.

```java
docker login -u <username>

docker tag getting-started <username>/getting-started

docker push <username>/getting-started
```

### 새로운 instance에서 이미지 실행

1. Open your browser to [Play with Docker](https://labs.play-with-docker.com/).
2. Click **Login** and then select **docker** from the drop-down list.
3. Connect with your Docker Hub account.
4. Once you’re logged in, click on the **ADD NEW INSTANCE** option on the left side bar. If you don’t see it, make your browser a little wider. After a few seconds, a terminal window opens in your browser.

```java
docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```

### Container filesystem

container가 실행될 때, 컨테이너들은 서로 다른 파일시스템을 갖는다. 따라서 어떤 컨테이너에서의 변화는 같은 이미지를 사용하는 컨테이너라도  다른 컨테이너에서는 볼 수 없다.

```java
//data.txt 에 1-10000중 한 수 넣는 구문
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"

//확인 가능
docker exec <container-id> cat /data.txt 

//같은 이미지로 새로운 컨테이너를 만들었을 때 data.txt파일은 없음.
docker run -it ubuntu ls /
```

### Container Volumes

Container안에서 수행한 일들은 Container가 삭제될 때 모두 사라진다. 하지만 Volumes을 사용하면 이 값들을 유지할 수 있다.

Volumes: 특정 컨테이너의 파일시스템과 host machine을 연결하도록 한다.

Container의 디렉토리가 마운트된다면 이 디렉토리에서의 변화는 host machine에서 확인할 수 있다.

Volume을 만들고 이것을 데이터가 저장될 디렉토리에 mount하면 데이터를 유지할 수 있다.

Mount: 리눅스 시스템에서 사용하기를 원하는 특정장치를 시스템에 인식시키는 작업.

### Named Volume

Docker는 디스크의 물리적 위치를 기억하고 우리는 그 이름을 기억해서 사용하면 된다.

```java
//todo-db라는 volume 생성
docker volume create todo-db
//-v flag specify a volume mount todo-db라는 volume을 사용할 것이고
// 이것은 /etc/todos 에 저장될 것이다.
// 이제 이 container를 삭제 후 명령어를 다시 쳐도 내용 유지.
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

Volume 정보 보기

```java
docker volume inspect <volume-name>
//Mountpoint는 데이터가 저장되는 실제 디스크 위치.
```

### Bind mounts

With**bind mounts**, we control the exact mountpoint on the host. We can use this to persist data, but it’s often used to provide additional data into containers. When working on an application, we can use a bind mount to mount our source code into the container to let it see code changes, respond, and let us see the changes right away.

![Untitled](docker%20getting-started%2081e7b10afba1461a82f29f984018c559/Untitled.png)

```java
docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
//-w /app -> 명령어가 실행될 working directory
// -v "$(pwd):/app" -> bind mounts the current directory from the host in the container into the /app directory

// docker log 보기
docker logs -f <container-id>
```

bind mounts는 local 개발환경 세팅에서 매우 흔하다. 이것을 사용하면 build 도구나 개발 환경을 설치하지 않아도 docker run을 실행하면 개발 환경이 준비된다.

### Multi Container

일반적으로 각 컨테이너는 한개의 일만을 수행하도록 한다.

컨테이너는 기본적으로 독립적으로 실행되며, 같은 machine에서 돌아가는 프로세스에 대해 어떤 것도 알 수 없다. 

하지만 두개 이상의 컨테이너가 같은 network에 있다면 두 컨테이너는 서로 대화할 수 있다.

```java
//todo-app이라는 network 생성
docker network create todo-app
//mysql container 실행 
docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
//mysql 실행.
docker exec -it <container-id> mysql -u root -p
```

### Connect to mysql

nicolaka/netshoot container를 사용하여 network 확인하기

 

```java
docker run -it --network <network-name> nicolaka/netshoot

//look up the ip address for the host mysql
dig mysql

//Answer 영역을 보면 우리가 이전에 network-alias 로 지정한 mysql이 
//172.23.0.2 로 resolve 되는 것을 알 수 있다.
// 따라서 앱은 mysql 이라는 host이름으로 network 연결 가능.
/*
; <<>> DiG 9.14.1 <<>> mysql
 ;; global options: +cmd
 ;; Got answer:
 ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
 ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

 ;; QUESTION SECTION:
 ;mysql.				IN	A

 ;; ANSWER SECTION:
 mysql.			600	IN	A	172.23.0.2

 ;; Query time: 0 msec
 ;; SERVER: 127.0.0.11#53(127.0.0.11)
 ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
 ;; MSG SIZE  rcvd: 44*/

```

### nodejs mysql에 연결하기

```java
docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:12-alpine \
   sh -c "yarn install && yarn run dev"

//아이템 추가 후

docker exec -it <mysql-container-id> mysql -p todos

mysql> select * from todo_items;
```

### Docker Compose

multi-container application을 정의하고 공유하는 데 사용하는 도구

docker-compose를 사용하는데 가장 큰 이점은 application stack을 파일에 저장할 수 있다는 것이다. 

1. docker-compose.yml 파일 생성
2. Compose file에서는 Schema version을 명시하면서 시작한다. 대부분은 가장 최신버전을 사용하는 것이 좋다.
3. application에서 실행할 services를 정의한다.

```java
//nodejs app
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"

//이 구문을 docker-compose.yml로 바꾸면
version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
       - 3000:3000
     working_dir: /app
     volumes:
       - ./:/app
     environment:
       MYSQL_HOST: mysql
       MYSQL_USER: root
       MYSQL_PASSWORD: secret
       MYSQL_DB: todos
```

```java
//mysql
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7

//docker-compose.yml
version: "3.7"

services:
	app:
		//아까 정의한 위의 내용들
	mysql:
     image: mysql:5.7
     volumes:
       - todo-mysql-data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: secret
       MYSQL_DATABASE: todos

volumes: //compose에서는 volumes이 자동으로 만들어지지 않기에 
//volumes: 에 명시해야한다. 이름만 쓰면 default 옵션이 사용된다.
	todo-mysql-data:
```

### Application Stack 실행

```java
//start up the application stack
docker-compose up -d
//결과창, network가 자동으로 생성된다.
/*
Creating network "app_default" with the default driver
 Creating volume "app_todo-mysql-data" with default driver
 Creating app_app_1   ... done
 Creating app_mysql_1 ... done
*/

//로그 확인
docker-compose logs -f <service-name>

// Tear it down, container는 멈추고 network는 제거된다.
docker-compose down
```

### Image-Building best practices

- **Security Scanning**

이미지에서 보안 취약점을 찾는데 사용.

```java
docker scan <image>
```

- **Image layering**

Image layer보기 

```java
docker image history <image>
// 더 보려면 --no-trunc 옵션 추가.
```

- **Layer Caching**

컨테이너 이미지 build 시간을 줄이는 방법.

```java
//각 명령어는 한개의 layer를 만듬.
//기존 파일의 문제점 만약 application에서 변경이 일어날 경우
// 그 때마다 yarn install로 dependencies를 다시 다운.
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]

//방법1. 
FROM node:12-alpine
 WORKDIR /app
 COPY package.json yarn.lock ./
 RUN yarn install --production
 COPY . .
 CMD ["node", "src/index.js"]

//방법2 .dockerignore 파일생성. 선택적으로 image관련된 파일을 복사.
```

### Multi-stage build

```java
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```