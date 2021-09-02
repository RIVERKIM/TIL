# Git 기초

생성일: 2021년 9월 1일 오후 10:34

![https://blog.kakaocdn.net/dn/bmrxsd/btqQZxU0lWM/Nr1jwIuSQZi0tr9BsfqC4K/img.png](https://blog.kakaocdn.net/dn/bmrxsd/btqQZxU0lWM/Nr1jwIuSQZi0tr9BsfqC4K/img.png)

- Working Directory: source 작업하는 곳 여기서 git add를 호출하면 Staging Area로 이동.  기본적으로 git add —all로 모든 파일을 올리는게 좋다.
- Staging area: 이곳에 올리지 않은 파일은 git 대상이 아니며 여기서 commit을 호출할 경우    소스가 로컬 저장소에 올라간다. 여기에 올라간 내용을 확인하는 명령어 git status

- 로그 보기: git log —graph —online
- 초기화: git reset —[hard:완전 삭제, soft, mixed] → 사용 금지
- 되돌리기: git revert: 이전 이력은 그대로 두고, 되돌릴 커밋의 코드만 원복시킨다.

VC GitLens 사용

rollback했을 때 gitlens file history를 확인하면 과거와 지금이 어디가 변경되었는지 알 수 있다. (또는 github history) 

Conflct: 한개의 파일을 양쪽에서 고쳤을 때 발생.

git init → .git folder생성(로컬 저장소 -master)

git remote add <name> <git-url> (원격 저장소 설정)

git push → 로컬 저장소 내용을 서버에 올림.

git pull → 서버의 내용을 로컬저장소로 이동.

### branch

각 branch 각각은 stage와 local repository를 갖는다.

명령어:

 git branch <branch-name> # branch 만들기

git branch # branch 전체 보기 → 로컬 //-r 붙이면 서버 // -a 전체

git checkout <branch-name> # 전환

git push origin <branch-name>

branch 목적: 소스를 서비스에 노출시키지 않는 것이 목적. 마스터가 서비스한다. 

**#Master로 merge하기**

git checkout master

git merge <branch-name>

#충돌 발생시 git status로 확인

서로 다른 파일 수정시 충돌 x 

서로 같은 파일 같은 부분 수정시 충돌 발생 → 직접 수정 필요

대규모 프로젝트에서는 master merge를 함부로 할 수 없으므로 

master branch의 변화를 지속적으로 내 branch로 가져와서 수정 필요.

 

**#branch 삭제**

git branch -d <branch-name> #local 삭제

git push —delete origin <branch-name> #서버에 branch 삭제

Branch 생성 및 변경전에 먼저 commit을 해야 안전하다.

항상 merge 하기전 git diff <branch-name> <파일명>으로 어떤점이 다른지 확인.

#**branch-name 에 있는 source를 현재 branch에 patch**

git checkout -p <branch-name> <file-name>

github에 있는 마스터 소스를 통합 테스트 진행 후 실제 서비스에 배포.

따라서 특정 파일을 branch에서 patch해서 가져올 경우 그 파일만 git add <파일명> 으로 add해야 한다.

#**서버의 특정 브랜치 가져오기**

git checkout -t origin/<branch-name>

git pull은 다른 사람이 PR을 통해서 코드를 업데이트했거나, 아니면 Github를 통해서 commit했을 때 내용을 클라이언트로 내려받는 것.(자동으로 변경사항 merge한다.)

git pull <원격 저장소 이름> <branch-name>

#**다른 사람이랑 공동으로 작업하기**

github settings → manage access 초대

git clone <git-remote-url> or git pull