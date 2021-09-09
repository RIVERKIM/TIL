# Docker 개념

생성일: 2021년 9월 9일 오후 6:52

### Docker Terms

- Docker image and container
- Docker Engine (Docker Daemon, core)
- Docker Client → 도커 밖
- Docker Host os → 도커 밖에서 host 해주는 os
- Docker Machine(Runtime Environment)
- Docker Compose - 여러개의 도커를 하나의 컨테이너 처럼 사용하게
- Docker Registry, Hub, Swarm

### Docker Container vs VM

![https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltb6200bc085503718/5e1f209a63d1b6503160c6d5/containers-vs-virtual-machines.jpg](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltb6200bc085503718/5e1f209a63d1b6503160c6d5/containers-vs-virtual-machines.jpg)

vm 이 Container 보다 훨씬 더 강력하게 격리된다. vm은 가상화된 하드웨어 위에 os가 올라가는 형태로 거의 완벽하게 host와 분리된다고 봐도 무방하다.

반면 container는 os 가상화이다. os를 가상화해서 올리고 커널을 host와 공유.

Container 장점:

1. 작다.

vm은 hypervisor가 hardware를 가상화 한다. 그리고 그 위에 Guest os가 올라가는 형태. 반면 container는 docker-engine위에 application 실행에 필요한 바이너리만 올라간다. 그외 부분은 호스트의 커널을 공유. 

만들어진 이미지가 작다는 것은 network를 덜 사용할 수 있다는 것을 의미.

cloud에 배포할 경우 용량이 더 적기때문에 더 적은 과금

1. 빠르다.

vm은 io가 발생하는 통로가 container보다 많다. vm은 host os에서 거의 완전히 분리된 형태로 운영이 되지만,  vm이 처리한 io는 결국 host os의 커널이 다시 받아 자신의 드라이버에 맞게 처리해 주어야한다. 여기서 병목현상이 발생. 반면 container는 커널을 공유하기 때문에 들어온 io가 쉽게 처리돼서 나갈 수 있게 된다. 즉 vm 이 container보다 더 많은 커널 처리가 들어가야하니 성능이 container가 더 좋다.

1. 라이프 사이클

image를 이용한 배포의 용이.  개발 환경에서 운영환경 image를 만들어 registry에 배포하고 원래 돌던 것을 교체할 떄 새로운 이미지만 올리면 변경 가능.

오케스트레이션이 제공 되기에 여러대의 host에 여러대의 container를 동시에 배포하면서 container를 관리할 수 있다. scale out도 쉽다.

단점: 

1. 보안

container가 host를 공유한다는 개념이기에 container하나가 뚫리면 바로 host 커널이 위험함.

1. 멀티 os

커널을 공유한다는 개념 때문에 host os와 전혀 다른 os를 container로 올릴 수 없음. 즉 linux 머신에서 window서버를 container로 운영할 수 없음.

### Docker Principle

- Namespace

PID namespace - container 마다 다른 pid

Network namespace - container마다 다른 ip

UID, MOUNT, UTS, IPC namespace

### Docker 명령어

```java
//container 실행
docker container run <option> <docker-image> <command>
-d option -> background 실행
-p port1:port2 -> port1 를 docker의 port2와 연결..
//ctrl p + ctrl q 하면 실행 상태에서 빠져나옴 아니면 exit해서 종료.
-it -> 대화형 서버 실행. //docker container run -it --name centos /bin/bash
//image 보기
docker images, docker image ls
// 실행 중인 container 보기
docker ps
-q // id만
-a // 전체
//docker 디스크 사용량
docker system df
// image 다운로드
docker pull <docker-image>
//image 보기
docker image inspect <image>
//image 삭제
docker image rm <image>
//전체 삭제
docker image rm $(docker container ls -aq)
//사용하지 않는 image 삭제
docker image prune
//docker image 검색
docker search <image 이름> ex) docker search nginx
//docker 중지 시작
docker container [stop|start] <container-id>

//하나의 명령 실행
docker container exec -it <container-id> <명령어>

*//port 확인
docker container port <container-id>

docker container rename <전 이름> <바꿀 이름>

//이미지 올리기
docker push <docker registry>
//docker login
docker login

ex) // 우분투 실행 후 들어가기
docker container start <container id>
docker attach <container-id>*

```

### Oracle Virtual Box port Forwarding

docker가 virtual box위에서 동작시 port를 지정해도 브라우저는 윈도우에서 실행되기때문에 포트 매칭이 안된다. 따라서 virtual box와 window사이를 port forwarding으로 매칭해줘야한다.