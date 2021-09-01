# Git 기초

생성일: 2021년 9월 1일 오후 10:34

![https://blog.kakaocdn.net/dn/bmrxsd/btqQZxU0lWM/Nr1jwIuSQZi0tr9BsfqC4K/img.png](https://blog.kakaocdn.net/dn/bmrxsd/btqQZxU0lWM/Nr1jwIuSQZi0tr9BsfqC4K/img.png)

- Working Directory: source 작업하는 곳 여기서 git add를 호출하면 Staging Area로 이동.  기본적으로 git add —all로 모든 파일을 올리는게 좋다.
- Staging area: 이곳에 올리지 않은 파일은 git 대상이 아니며 여기서 commit을 호출할 경우    소스가 로컬 저장소에 올라간다.

- 로그 보기: git log —graph —online
- 초기화: git reset —[hard:완전 삭제, soft, mixed] → 사용 금지
- 되돌리기: git revert: 이전 이력은 그대로 두고, 되돌릴 커밋의 코드만 원복시킨다.

VC GitLens 사용

rollback했을 때 gitlens file history를 확인하면 과거와 지금이 어디가 변경되었는지 알 수 있다. (또는 github history) 

Conflct: 한개의 파일을 양쪽에서 고쳤을 때 발생.