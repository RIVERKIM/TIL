# CI/CD 실습

생성일: 2021년 9월 2일 오후 11:19

### 예시 배포 환경

1. 웹사이트 코드 작성.
2. 웹사이트 코드를 린트, 웹팩 빌드 해서 AWS S3 bucket에 html 파일을 업로드 한다.
3. Node.js 백엔드 코드를 typescript 작성한다.
4. 위 코드를 javascript compile 하고, 테스트 코드를 돌려서 도커 이미지를 만들어 ECR에 올린다.
5. 업로드한 ECR 이미지로 ECS 서비스를 재시작한다.(rolling deploy) ⇒ continuous deploy

### 설정

1. AWS Ec2 instance 생성
2. 키 생성
3. 키 .ssh 파일로 이동

~/.ssh/config

```jsx
Host <이름>
	User ec2-user
	HostName <ec2-퍼블릭 ipv4>
	IdentifyFile ~/.ssh/<key-이름>.pem

```

1. ec2 instance → 보안 → 보안 그룹 → 인바운드 룰 편집 → 포트 범위(8080, 80)  0.0.0.0/0(사용자) = 8080포트와 80포트를 모든 사용자에게 열어주겠다.
2. Putty.exe, puttygen.exe 설치
3. puttygen.exe 실행 → conversation-import key → pem 키 가져오기 → save private key → pkk 파일 저장.
4.  putty.exe 실행 → HostName에 (퍼블릭 DNS) 인스턴스의 퍼블릭 DNS 이름을 사용하여 연결하려면 my-instance-user-name@my-instance-public-dns-name를 입력합니다.
5. ssh → auth → browse 해서 ppk 파일 등록 후 open

> [참고](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/putty.html), [참고](https://luji.tistory.com/6), [instance-user-name](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/managing-users.html)

### EC2 서버 설정.

젠킨스 설치하기 

```jsx
# red hat 젠킨스 패키지 추가.
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo systemctl daemon-reload

#install java, docker, git, jenkins
sudo yum install -y java-1.8.0-openjdk jenkins git docker

#자바 버전 8로 설정
alternatives --config java
sudo service jenkins start
/*
# -> 문제 발생 genkins 다운이 안됨.
#해결방안 아래 명령어 후 jenkins 다시 다운
amazon-linux-extras install epel -y
yum update -y
[출처](https://stackoverflow.com/questions/68806741/how-to-fix-yum-update-of-jenkins) 
*/
```

이제 젠킨스는 우리 EC2 public ip 주소:8080에 호스팅 된다.

처음 접속시 Unlock Jenkins 화면만 보이는데 그 아래 파일 경로를 복사해서 EC2 터미널에 치면 값을 얻을 수 있다. 

이 값으로 로그인

```jsx
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
//263ceab83bfa4777a9a0c99ff34e4f32
```

추천해주는 플러그인 전부 설치

원하는 계정명 비밀번호 설정 → 꼭기억 (계정명: kgr4163)

### Genkins git 연동

젠킨스 관리 → Manage Credential → jenkins(store) → global credentials

먼저 github new Repository → settings → developer settings → personal access token 발급

Global credentials → kind(username with password) → username(github username) → password(personal access token)

id는 우리가 부여하는 것 (jenkinsForGit)

#젠킨스도 결국 aws를 사용하는 유저이기 때문에 권한을 줘야함.

AWS service → IAM 대시보드 → 사용자 → 사용자 추가 → 사용자이름 + 프로그래밍 방식 access →

권한(실제 production이면 jenkins 정책을 만들지만 실습이기에 admin(기존 정책) 준다.)

credential 추가

secret text 설정으로 아까 받았던 aws jenkins 유저 access key 랑 secret key 등록 (여기서 이름은 젠키스 파일에서 credentials("이름")으로 불러올 예정.)

```jsx
environment {
        //key를 이용해서 aws 명령어 사용하기 위해
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId') //설정한 cridentials 이름
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2' // default 서울.
        HOME = '.'
    }
```

젠키스 유저를 등록안하고 ec2에 instance profile attach하는 곳에 IAM role을 attach 시켜주면 따로 credentials 등록 안해도 됨.

### aws s3 추가 + Frontend 배포

aws s3 service → 버킷 추가 →이름 아무거나 (이 이름이 나중에 s3://<이름>으로 쓰임) → public 설정 (website로 사용할 것이기에 설정 → 사용후 제거!! 돈나감;;;)

**메일 설정**

jenkins 관리 → 환경 설정 → Email로 알려줌 → smtp 서버 (gmail → smtp.gmail.com) → use SMTP Authentication 설정 → 사용자 명 , 비밀번호에 gmail 입력.

위의 메일 부분에서 connect timeout 문제 발생. google authenfication문제이기에 일단 건너뜀. 

```jsx
//aws s3에 파일을 올림.
        stage('Deploy Frontend') {
            steps {
                echo "Delopying Frontend"
                //프론트엔드 디렉토리의 정적파일들을 s3에 올림, 이 전에 반드시 EC2 instance profile 등록을 해야함.
                dir ('./website') {
                    sh '''
                    aws s3 sync ./ s3://testforgaram
                    '''
                }
            }

            post {
                success {
                    echo 'Successfully Cloned Repository'

                    mail to: 'kgr4163@korea.ac.kr',
                         subject: 'Deploy Frontend Success',
                         body: "Successfully deployed frontend!!"
                }

                failure {
                    echo 'I failed :('

                    mail to: 'kgr4163@korea.ac.kr',
                         subject: 'Failed pipeline',
                         body: "Something is wrong with deploy frontend"
                }
            }
        }
```

### 백엔드 배포

```jsx
stage('Lint Backend') {
            //Docker plugin and Docker pipeline 두개를 깔아야 가능.
            agent {// jenkins 는 node가 없기때문에 docker 에 노드 받아서 실행하라고 명령.
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./server') {
                    sh '''
                    npm install &&
                    npm run lint
                    '''
                }
            }
        }

        stage('Test Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }
            steps {
                echo 'Test Backend'
            
                dir ('./server')
                    sh '''
                    npm install &&
                    npm run test
                    '''
            }
        }

        //우리가 backend는 docker로 배포하기 때문에
        //jenkins에 docker를 설치해 줬다는 가정하에.
        stage('Build Backend') {
            agent any
            steps {
                echo 'Build Backend'
                //build 관련 파라미터는 application레벨에서 관리하고
                //여기서는 PROD 인지 DEV인지 알려주는 것만.
                dir ('./server') { 
                    sh """
                    docker build . -t server --bulid-arg env=${PROD}
                    """
                }
            }

            post {
                failure {
                    //빌드 하다가 실패하면 pipeline 종료.
                    error 'This pipeline stops here'
                }
            }
        }

        stage('Deploy Backend') {
            //실환경에는 ecs 혹은 쿠버네티스 환경 업데이트
            agent any
            
            steps {
                echo 'Build Backend'

                dir('./server') { //두번째 실행부터는 docker rm -f $(docker ps -aq)와 같은 
                //명령어 써서 돌고 있는 컨테이너들 꺼야함. 
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                }
            }

            post {
                success {
                    mail to: "kgr4163@korea.ac.kr",
                         subject: "Deploy Success",
                         body: "Successfully deployed!"
                }
            }
        }
    }
```

### 젠키스 운용

젠킨스 운용전 plugin 설치 필요 (docker, git)

젠킨스관리 → 플러그인 매니저 → (docker, docker pipeline 다운.)

aws 터미널에

```jsx
sudo yum install docker
// docker 사용하도록 설정.
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins

sudo service docker start
```

docker 설치 후 /var/run/docker.sock의 permission denied 나오는 경우는 "sudo chmod 666 /var/run/docker.sock"

젠킨스에서 item 만들기

jenkins → create a new job → 이름 + pipe라인 클릭 후 ok → pipeline에 이전에 작성했던 jenkins file 내용 끌어와서 넣음. → github project 에 repository url 입력. (끝에 git은 빼고)

pollSCM 클릭.

만들었던 파일들 git push 후 build now

### 환경 변수 설정.

젠킨스 관리 → 환경 설정 → global properties(environment variables)

잘 돌아가면 서버에 curl 보내기 + s3에 잘 있는지

```jsx
curl --ipv4 -v "<url>"
```

### 최종 Jenkinsfile

```jsx
pipeline {
    //어떤 노예를 쓸건가
    agent any

    // 몇 분 주기로 trigger되는 가
    triggers {
        pollSCM('*/3 * * * *')
    }
    // 이 파이프라인에서 쓸 환경변수들
    environment {
        //key를 이용해서 aws 명령어 사용하기 위해
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId') //설정한 cridentials 이름
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2' // default 서울.
        HOME = '.'
    }
    //pipeline 각 단계
    stages {
        // git레포지토리를 다운.
        stage('Prepare') {
            agent any

            steps {
                //echo "Lets start Long Journey! Env: ${Env}"
                echo 'Clonning Repostiory'

                git url: 'https://github.com/RIVERKIM/CI-CD-with-jenkins-practice.git',
                    branch: 'master',
                    credentialsId: 'jenkinsForGit'

            }

            post {
                success {
                    echo 'Successfully Cloned Repository'
                }
                
                always {
                    echo "i tried..."
                }
                //post 다 끝났을 때 
                cleanup {
                    echo "after all other post condition"
                }
            }

        }

        // stage('Only for production') {
        //     when {
        //         branch 'production'
        //         environment name: 'APP_ENV', value: 'prod'
        //         anyOf {
        //             environment name: 'DEPLOY_TO', value: 'production'
        //             environment name: 'DEPLOY_TO', value: 'staging'
        //         }
        //     }
        // }
        //aws s3에 파일을 올림.
        stage('Deploy Frontend') {
            steps {
                echo "Delopying Frontend"
                //프론트엔드 디렉토리의 정적파일들을 s3에 올림, 이 전에 반드시 EC2 instance profile 등록을 해야함.
                dir ('/var/lib/jenkins/workspace/myProject@2/OneDrive/Desktop/projectPractice/Jenkins-practice/website') {
                    sh '''
                    aws s3 sync ./ s3://testforgaram
                    '''
                }
            }

            post {
                success {
                    echo 'Successfully Cloned Repository'

                    // mail to: 'garamkim1357@gmail.com',
                    //      subject: 'Deploy Frontend Success',
                    //      body: "Successfully deployed frontend!!"
                }

                failure {
                    echo 'I failed :('

                    // mail to: 'garamkim1357@gmail.com',
                    //      subject: 'Failed pipeline',
                    //      body: "Something is wrong with deploy frontend"
                }
            }
        }

        stage('Lint Backend') {
            //Docker plugin and Docker pipeline 두개를 깔아야 가능.
            agent {// jenkins 는 node가 없기때문에 docker 에 노드 받아서 실행하라고 명령.
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./OneDrive/Desktop/projectPractice/Jenkins-practice/server') {
                    sh '''
                    npm install &&
                    npm run lint
                    '''
                }
            }
        }

        stage('Test Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }
            steps {
                echo 'Test Backend'
            
                dir ('./OneDrive/Desktop/projectPractice/Jenkins-practice/server') {
                    sh '''
                    npm install &&
                    npm run test
                    '''
            }
        }
        }

        //우리가 backend는 docker로 배포하기 때문에
        //jenkins에 docker를 설치해 줬다는 가정하에.
        stage('Build Backend') {
            agent any
            steps {
                echo 'Build Backend'
                //build 관련 파라미터는 application레벨에서 관리하고
                //여기서는 PROD 인지 DEV인지 알려주는 것만.
                dir ('./OneDrive/Desktop/projectPractice/Jenkins-practice/server') { 
                    sh """
                    docker build . -t server 
                    """
                    //--bulid-arg env=${PROD}
                }
            }

            post {
                failure {
                    //빌드 하다가 실패하면 pipeline 종료.
                    error 'This pipeline stops here'
                }
            }
        }

        stage('Deploy Backend') {
            //실환경에는 ecs 혹은 쿠버네티스 환경 업데이트
            agent any
            
            steps {
                echo 'Build Backend'

                dir('./OneDrive/Desktop/projectPractice/Jenkins-practice/server') { //두번째 실행부터는 docker rm -f $(docker ps -aq)와 같은 
                //명령어 써서 돌고 있는 컨테이너들 꺼야함. 
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                }
            }

            post {
                success {
                    echo "Build backend success!"
                    // mail to: "kgr4163@korea.ac.kr",
                    //      subject: "Deploy Success",
                    //      body: "Successfully deployed!"
                }
            }
        }
    
}
}
```