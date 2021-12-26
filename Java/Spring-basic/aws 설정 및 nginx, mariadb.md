# aws 설정 및 nginx, mariadb

### 서버 타임존 변경

- aws는 기본적으로 os 설치시 타임존이 UTC로 맞춰져 있으므로 TimeZone을 변경해줘야 한다.

```java
[ec2-user@ip-172-31-26-186 ~]$ date
Sun Dec 26 04:50:20 UTC 2021
[ec2-user@ip-172-31-26-186 ~]$ sudo rm /etc/localtime
[ec2-user@ip-172-31-26-186 ~]$ sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
[ec2-user@ip-172-31-26-186 ~]$ date
Sun Dec 26 13:51:06 KST 2021
```

- ln - s ⇒ symbolic link 생성

### Nginx 설치

```java
sudo amazon-linux-extras install nginx1
sudo service nginx start
```

- nginx 파일 위치
    - 설정: /etc/nginx/nginx.conf
    - 로그: /var/log/nginx
- 인바운드 규칙 수정:  80번 포트 열기

### Mariadb 설치

```java
sudo yum install mariadb-server
sudo service mariadb start
```

**Port 변경/ 언어셋 변경**

```java
$ sudo vim /etc/my.cnf
[mysqld]
port = 36091
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci
$ sudo service mariadb restart
```

- utf8mb4로 설정하는 이유는 IOS에서 이모티콘/ 이모지등을 표현할 때 문자당 4byte를 사용하는데 기본 설정을 사용하면 3byte만 받을 수 있어 utf8mb4로 변경.

### root 패스워드 변경

```java
// root 패스워드 설정
$ mysqladmin -u root -p password 'xxRootPwd^&*'
Enter password: 그냥 엔터
$ mysql -u root -p
Enter password:설정한 비밀번호 입력
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
```

- root 패스워드는 처음엔 설정이 안되어 있으므로 설정을 해줘야 한다.
- 

### 운용영 database / id 생성

- root를 그냥 사용할 경우 모든 권한을 가지고 있고 외부에서 접속 가능하게 설정할 경우 위험하기 떄문에 운영 DB와 운영 ID를 따로 만들어서 권한을 격리시켜 사용하는 것이 바람직하다.

```java
create database garam
//사용자를 추가하면서 패스워드까지 설정.
create user garamkim@localhost identified by "garam"
//garam 데이터 베이스의 권한을 garamkim 유저에게 준다.
grant all privileges on garam.* to garamkim@localhost identified by "garam"
//privileges 등록
flush privileges

exit;
//접속
mysql -u garamkim -p 
```

- mariadb  port를 inbound에 추가.

### 운영 계정 외부 접속 권한 설정

- 운영 dB에 대해 운영 유저가 내 서버 ip에서만 비밀번호를 넣어 접근가능하도록 설정.

```java
// root 로그인
MariaDB [(none)]> grant all privileges on daddyprogrammer.* to happydaddy@'103.xxx.200.%' identified by 'xxxdaddypwd^&*';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

**mysql workbench 접속 확인**

- 새로운 커넥션 생성, 서버 ip = ec2 ip, port = mysql port, username = 운영용 id, password = 운영용 id password로 접속이 되는지 확인.
- 잘 돌아가고 있는지 확인

```java
//mysql 만 보기
netstat -tap | grep mysql
//돌아가고 있는 프로그램 전체 다 보기
netstat -ln

```