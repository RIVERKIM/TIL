# Git Flow

생성일: 2021년 9월 1일 오후 10:34

Git Flow: 실무에서 깃을 어떻게 사용할지에 대한 표준화 된 방식.

### Git Flow Concepts

- master - 기본 - 항상 서비스 가능한 상태의 소스 코드
- develop - 기본 - 개발의 주축이 되는 branch
- feature/ - 기능을 개발할 때
- release/ - 배포할 때
- hotfix/ - 긴급 버그 수정

![https://techblog.woowahan.com/wp-content/uploads/img/2017-10-30/git-flow_overall_graph.png](https://techblog.woowahan.com/wp-content/uploads/img/2017-10-30/git-flow_overall_graph.png)

기능 개발시 develop branch 에서 feature branch 따서 작업

작업 완료 시  develop branch와 merge하고 관련 feature branch 삭제.

develop branch에서 release branch를 따서 배포를 위한 작업 수행 -개발 서버에 배포

release branch에서 버그가 있다면 수정하고 다시 develop으로 merge 되고 release는 master로 merge. → 이 소스는 마스터와 merge 됐기에 이제 배포가 가능한 상태이므로 tag를 작성 후 이 태크로 배포.

따라서 master와 develop 브랜치는 항상 최신의 코드를 갖게 된다.

치명적인 버그 발생시 master branch에서 hotfix 브랜치를 따서 버그 수정 후 다시 master에 merge

### Install Git Flow

방법1: Install gitflow VSCode extension

방법2: [주소](https://github.com/nvie/gitflow)

1.  binaries 로 다운.
2. download dll files

    libiconv-1.9.2-1-bin\bin\libiconv2.dll

    libintl-0.14.4-bin\bin\libintl3.dll

    util-linux-ng-2.14.1-bin\bin\getopt.exe

3. copy to msysgit's bin - C:\Program Files\Git\bin

```java
$ git clone --recursive git://github.com/nvie/gitflow.git
$ cd gitflow

$ git config --global url."https://github".insteadOf git://github

$ ln -s "/C/Program Files (x86)/Git/bin/git-flow" git-flow
```

### Git Flow Commands

```java
git flow init //git flow 시작 -> master 와 develop branch 생성
git push origin --all // 그 두 branch를 서버에 push
git flow feature start <feature-branch> //feature branch 생성.
git flow feature list // feature branch 목록 보여줌.
git add .
git commit -m
// push & pull-request(to review)
git flow feature finish <feature-branch>
git flow release start <tag> //release와 hotfix는 tag단위로 배포.
git flow release finish <tag>
git tag
git push origin --all --follow-tags
git flow hotfix start <tag>
git flow hotfix finish <tag>
```

![Untitled](Git%20Flow%20450491a39d8745c5aad999d2d01c4e4f/Untitled.png)

Merge 할 때 항상 PR을 통해 코드 리뷰

원격 branch정보 삭제

```java
git push <remote> --delete <이름>
```