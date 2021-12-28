# Nginx를 이용한 무중단 배포

### 무중단 배포(Blue /Green Deployment)

- 기존에는 배포시 Was가 완전히 로딩 될 때까지 서비스가 불가능한 이슈가 있다.

### Spring Actuator 추가

- springboot에서는 actuator를 제공하여 서비스의 상태 정보를 실시간으로 모니터링 할 수 있도록 제공하고 있다.

```java
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

- [localhost:8080/actuator/health에](http://localhost:8080/actuator/health에) 접근하면 현재 서버 상태를 json으로 볼 수 있다.
- actuator가 제공하는 endpoint에 접근할 수 있도록 /actuator/health를 접근 허용해야 한다.

```java
.antMatchers(HttpMethod.GET, "/helloworld/**","/actuator/health").permitAll()
```

> 참고: GET /actuator 로 요청을 보내면 정보를 알 수 있는 url이 주어진다.
> 

### 배포 shell script 작성

```java
touch deploy.sh
chmod +x deploy.sh
```

- 타겟 서버에 배포 디렉토리 생성(/app/dist)
- scp 명령어로 로컬 build 디렉터리에서 jar 파일을 타겟서버 배포 디렉터리로 전송
- 전송한 jar 파일에 대한 심볼릭 링크 생성. 배포시 버전에 따른 jar 파일명을 하나의 이름으로만 관리함으로써 jar 실행시 최신 버전의 jar 파일 이름을 알아내야할 필요가 없게 해주는 용도.
- 현재 실행중인 서버 조회. 8083이 띄어져 있으면 새로 업데이트된 jar 파일로 8084 서버를 한대 더 실행.
- 신규 서버 스타트가 완료되면(health 체크를 5초마다 실행) 기존에 실행되고 있던 서버는 종료.
- 서버리스트에 여러개의 서버가 나열되어 있다면 위의 작업이 서버수만큼 반복.

**deploy.sh**

```java
#!/bin/bash
PROFILE=alpha
PROJECT=api-test
PROJECT_HOME=
JAR_PATH=/home/ec2-user/api-test-0.0.1-SNAPSHOT.jar
DEPLOY_PATH=/home/ec2-user/app
JAVA_OPTS="-XX:MaxMetaspaceSize=128m -XX:+UseG1GC -Xss1024k -Xms256m -Xmx384m -Dfile.encoding=UTF-8"
PORT=8083
SERVER=18.191.158.21

echo Deploy start

echo Target server - $server

mkdir -p $DEPLOY_PATH/dist

runPid=$(pgrep -f $PROJECT)

if [ -z $runPid ]; then
        echo "No servers are running"
fi

runPortCount=$(ps -ef | grep $PROJECT | grep -v grep | grep $PORT | wc -l)
if [ $runPortCount -gt 0 ]; then
        PORT=8084
fi
echo "Server $PORT Starting..."

nohup java -jar -Dserver.port=$PORT -Dspring.profiles.active=$PROFILE $JAVA_OPTS api-test-0.0.1-SNAPSHOT.jar &

echo "Health check $PORT"

for retry in {1..10}
do
        health=$(curl -s http://localhost:$PORT/actuator/health)
        checkCount=$(echo $health | grep 'UP' | wc -l)
        echo "Check count - $checkCount"
        if [ $checkCount -ge 1 ]; then
                echo "Server $PORT Started Normally"

                if [ $runPid -gt 0 ]; then
                        echo "Server $runPid Stopping..."
                        kill -TERM $runPid
                        sleep 5
                        echo "Server $runPid Stopped"
                fi
                break;
        fi
        sleep 5
done

echo Deploy end
```

### 무중단 서비스를 위한 개선

- Java 서블릿 컨테이너 기반의 프레임워크는 서버 구동 속도가 느리다는 단점을 가지고 있다.
- 이 시간 동안에 오는 서비스의 응답을 처리할 수 없으므로 추가적인 처리가 필요.
- 여기서는 tomcat 인스턴스를 nginx의 설정에서 순간적으로 교체하는 방법으로 무중단을 구현.

/etc/nginx/conf.d/service_addr.inc

```java
set $service_addr http://127.0.0.1:8083;
```

- 매번 바뀌는 port 환경 변수 추가

/etc/nginx/nginx.conf

```java
include /etc/nginx/conf.d/service_addr.inc;

location / {
     proxy_set_header    HOST $http_host;
     proxy_set_header    X-Real-IP $remote_addr;
     proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header    X-Forwarded-Proto $scheme;
     proxy_set_header    X-NginX-Proxy true;
     proxy_pass $service_addr;
     proxy_redirect  off;
     charset utf-8;
}
```

- service_addr.inc에서 설정한 환경 변수로 처리되도록 설정.

deploy**.sh**

```java
for retry in {1..10}
do
        health=$(curl -s http://localhost:$PORT/actuator/health)
        checkCount=$(echo $health | grep 'UP' | wc -l)
        echo "Check count - $checkCount"
        if [ $checkCount -ge 1 ]; then
                echo "Server $PORT Started Normally"

                if [ $runPid -gt 0 ]; then
                        echo "Server $runPid Stopping..."
                        kill -TERM $runPid
                        sleep 5
                        echo "Server $runPid Stopped"

                        echo "Nginx Port change"
                        echo "set \$service_addr http://127.0.0.1:${PORT};" | sudo tee /etc/nginx/conf.d/service_addr.inc
                        echo "Nginx reload"
                        sudo service nginx reload
                fi
                break;
        fi
        sleep 5
done
```

- port 변경 + nginx 재구동 설정 추가.