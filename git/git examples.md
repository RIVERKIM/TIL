# git 예제들

생성일: 2021년 9월 8일 오후 3:21

1. **실수로 remote repository에 불필요한 파일/폴더를 push 했을 때**

.gitignore에 추가하고 삭제 후 push해도 원격에서는 지워지지 않음.

1) 로컬과 원격 모두 지우고 싶을 때

```java
git rm -r <파일> 
```

2) 원격만 지우고 싶을 때

```java
git rm -r --cached <파일>
```

1. **실수로 Topic 브랜치가 아닌 master branch에서 작업.**

1) push 는 안했을 때

```java
git branch tmp //임시 브랜치 사용.
git reset --hard HEAD~ //마스터 복구
git merge --no-ff tmp //토픽브랜치 변경 후
git branch -d tmp
```

2) push 한 경우

```java
git branch tmp
git reset --hard HEAD~
git push -f origin master //그냥 git push origin master 충돌 발생
git merge --no-ff tmp//토픽브랜치 변경 후
git branch -d tmp
```

1. 작업 중이던 Topic 브랜치를 실수로 삭제

git reset으로 브랜치는 복구되지 않는다.

브랜치를 그대로 복구하고자 하는 경우.

```java
git reflog //branch 관련 로그
git checkout -b topic <commit hash>

```

1. 버그 수정(hotfix)해서 Tag 0.2.1 배포 후

1) 예기치 못한 장애 발생

빨리 0.2.0으로 Rollback 하고 다시 배포.

```java
git branch tmp // 0.2.1 브랜치 내용 보존 -> git flow 경우 develop에 남아 있음.
git reset --hard 0.2.0
git push origin +master // git push -f origin master
//이 때 hotfix branch를 따면 다른 브랜치에 내용이 있더라도 master를 따라감.

```

2) 0.2.1의 소스를 이용해서 0.2.2를 작업하자.

```java
git flow hotfix 0.2.2
git merge --no-ff develop
//고치고 commit 후
git flow hotfix finish 0.2.2 //hotfix master에 merge
git push origin --all --follow-tags // 배포
```