# Spring project 배포 설정

### Executable Jar 생성

```java
./gradlew bootJar
```

- bootJar task를 실행하면 ./build/libs 아래에 실행가능한 jar가 생성된다.

> 참고: build 과정에서 lombok에러가 발생하면 jdk가 16과 lombok호환성 문제이니 lombok을 1.18.20으로 바꾸고 build하면 된다.
> 

### Executable Jar 서버 업로드

- scp 명령을 이용하여 ec2 서버에 jar 파일을 전송.
- scp(secure copy)는 ssh와 같은 프로토콜을 사용하여 원격지로 안전하게 파일을 복사하는 명령어 이다.
- scp -i 서버 인증서.pem 프로젝트 경로 ec2-user@서버ip:~/업로드 경로

```java
$ scp -i AwsKeyPair.pem /Users/happydaddy/git/SpringRestApi/build/libs/api-0.0.1-SNAPSHOT.jar ec2-user@10.20.30.40:~/api
```

### Java 설치

- yum으로 설치 가능한 jdk는 8까지이다.
- 이후부터는 aws corretto사용
- 17 주소 [https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html)

```java
//파일 다운로드
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-17-x64-linux-jdk.rpm -o java17.rpm
//다운 받은 파일 설치
sudo yum localinstall java17.rpm 

// 환경 변수 설정

which java // 자바 경로
readlink -f /usr/bin/java  // 위에서 나온 경로의 실제 위치 확인

vi /etc/profile // 환경 변수 설정을 위해 파일 편집
//맨 아래에 설정.
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.amzn2.0.2.x86_64
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

source /etc/profile // profile 변경 사항 적용.

echo $JAVA_HOME // 환경 변수 확인
```

- 설치된 자바 버전 확인: yum list installed | grep “java”
- 불필요한 버전 삭제: sudo yum remove java-1.8.0-openjdk-headless.x86_64

### Nginx 설정 수정 및 실행

- /etc/nginx 폴더에 있음.

```java
$ sudo vim nginx.conf
// server 설정 위에 upstream 추가
upstream tomcat {
        ip_hash;
        server 127.0.0.1:8083;
}
// server안에 location 내용 수정
location / {
        proxy_set_header    HOST $http_host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
        proxy_set_header    X-NginX-Proxy true;
        proxy_pass http://tomcat;
        proxy_redirect  off;
        charset utf-8;
}
$ sudo nginx -t
$ sudo service nginx reload
```

Tomcat 실행

```java
nohup java -jar -Dserver.port=8083 -Dspring.profiles.active=alpha -XX:MaxMetaspaceSize=128m -XX:+UseG1GC -Xss1024k -Xms256m -Xmx384m -Dfile.encoding=UTF-8 api-test-0.0.1-SNAPSHOT.jar &
```

> 참고: nohup는 백그라운드 프로세스로 작업할 때 사용하는 명령어.
nohup [명령] &
> 

application.yml (alpha 부분만)

```java
logging:
  level:
    root: warn
    spring.apitest: info
  path: /home/ec2-user/api/log
  file:
    max-history: 7

spring:
  config:
    activate:
      on-profile: alpha
  datasource:
    url: jdbc:mysql://18.191.158.21:36091/garam?useUnicode=true&autoReconnect=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: garamkim
    password: garam
  jpa:
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: true
  url:
    base: http://ec2-18-191-158-21.us-east-2.compute.amazonaws.com
```

- sql과 연동시 주소는 jdbc:mysql://ec2주소/database?~
- 시간존이 KST이기떄문에 끝에  &serverTimezone=UTC 꼭 붙여주어야 한다.
- driver-class-nam: com.mysql.cj.jdbc.Driver 고정.

### Gracefully Shutdown

- 업데이트 등의 이유로 서버를 재시작해야 하는 경우가 있는데, 이 때 서버를 종료시켜버리면 서버에서 처리하던 작업이 종료되기 때문에 주의가 필요.
- linux에서 kill 명령에 -TERM 혹은 -15 옵션을 주면 서버를 바로 종료시키지 않고, 서버에게 종료 요청을 보낸다.
- 해당 서버는 TERMINATE 요청을 받으면 현재 처리중인 작업을 모두 마무리하고 서버를 종료시키게 된다.
- TIMEOUT안에 종료되지 않으면 강제 종료 시킴.

```java
@Slf4j
public class GracefulShutdown implements TomcatConnectorCustomizer, ApplicationListener<ContextClosedEvent> {
    private static final int TIMEOUT = 30;

    private volatile Connector connector;

    @Override
    public void customize(Connector connector) {
        this.connector = connector;
    }

    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        this.connector.pause();
        Executor executor = this.connector.getProtocolHandler().getExecutor();
        if (executor instanceof ThreadPoolExecutor) {
            try {
                ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
                threadPoolExecutor.shutdown();

                if (!threadPoolExecutor.awaitTermination(TIMEOUT, TimeUnit.SECONDS)) {
                    log.warn("Tomcat thread pool did not shut down gracefully within " + TIMEOUT +
                            " seconds. Processing with forceful shutdown");

                    threadPoolExecutor.shutdownNow();

                    if (!threadPoolExecutor.awaitTermination(TIMEOUT, TimeUnit.SECONDS)) {
                        log.error("Tomcat thread pool did not terminate");
                    }
                }else {
                    log.info("Tomcat thread pool has been gracefully shutdown");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
```

### Boot 설정에 GracefulShutdown 추가

```java
@SpringBootApplication
public class SpringRestApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringRestApiApplication.class, args);
    }

// 다른 설정 생략

    @Bean
    public GracefulShutdown gracefulShutdown() {
        return new GracefulShutdown();
    }

    @Bean
    public ConfigurableServletWebServerFactory webServerFactory(final GracefulShutdown gracefulShutdown) {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.addConnectorCustomizers(gracefulShutdown);
        return factory;
    }
}
```