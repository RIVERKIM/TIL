# git 실무

생성일: 2021년 9월 1일 오후 10:34

### Local Repository

Working Dir → staging → .git

#각각  add, commit으로 이동

#반대로 돌아오는 것은  reset <file-name>, reset —soft

### 여러개의 github 계정 연동.

동일한 로컬환경에서 다른 계정으로 커밋 기록을 남기고 싶을 때 깃허브에서 제공하는 ssh-key 라는 인증수단 사용.

**ssh-key 등록**

```jsx
$: ssh-keygen -t rsa -C "<email>" -f "key_name(id_rsa_userA)"
```

ssh 키는 C:\Users\garam\.ssh 등록

**키 등록**

```jsx
eval $(ssh-agent)

ssh-add <key-name>
```

config 작성.

~/.ssh/config파일

```jsx
Host github.com-main
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_main

Host github.com-sub
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_sub
```

github 각 계정 로그인 → settings → SSH and GPG keys → SSh news → 만들었던 sshkey pub 를 사용해서 연동.

위의 작성했던 config 파일의 Host이름을 사용해서 계정 연동

```jsx
ssh -T github.com-main
```

원하는 저장소에 가서 clone → ssh 주소 얻으면

- remote 설정이 되어있는 경우:

git remote set-url <로컬 저장소> git@github.com-sub:TempeICON/myprojectuserA.git

- 처음 하는 경우

git remote add <name> git@github.com-sub:TempeICON/myprojectuserA.git

### Git with VS code

- Git Extension 다운.
- Markdown all in one

명령 팔레트에서 다양한 git 명령 사용가능

add gitignore

add remote 등

source control에서 commit 가능.

commits 창에서 push 가능.

#editor 변경

```jsx
git config --global core.editor "vscode -w"
```

git 설정 확인

git config —list

 

### Git Branch

1. master 브랜치는 항상 Deploy할 수 있는 상태로 두자.
2. 작업을 할 때는 항상 Topic 브랜치를 만들어서 하자
3. 브랜치 이름은 누가봐도 알 수 있도록 작명.
4. 정기적으로 자주 push
5. Merge가 끝나면 해당 Topic 브랜치는 바로 삭제.

**Fast-forward**

브랜치를 master와 병합할 떄 master 브랜치의 상태가 이전부터 변경되어 있지 않아서 바로 병합할 수 있는 상태.

[https://t1.daumcdn.net/cfile/tistory/2132A64F54F3155D2E](https://t1.daumcdn.net/cfile/tistory/2132A64F54F3155D2E)

default로 —ff 옵션이며 단순히 태그만 옮김.

[https://t1.daumcdn.net/cfile/tistory/232FEE4F54F3155E34](https://t1.daumcdn.net/cfile/tistory/232FEE4F54F3155E34)

—no-ff 옵션인 경우 (git merge —no-ff<branch-name> )

fast forward 관계인 경우에도 반드시 병합 커밋을 만듬.

[https://t1.daumcdn.net/cfile/tistory/2641324F54F3155F1F](https://t1.daumcdn.net/cfile/tistory/2641324F54F3155F1F)

**commit 메시지 변경**

git commit --amend

### Rebase

1. 하나의 Commit은 하나의 의미만 갖자
2. rebase

ex) git rebase -i HEAD~2 → 제일 위에서 두번째 커밋까지 합치겠다.

→ 이후에 명령어 입력

fixup → 바로 앞(previous)로 합치기

drop → 해당 commit 삭제

squash → 바로 앞(previous)을 최신으로 합치기

등 여러 명령어 사용해서 합치기

reset → push 하기 전 없애기 

```jsx
git reset —hard HEAD~ 최신 것 지우기
git reset --soft HEAD~-> commit 에서 staging으로
git reset HEAD -> 최신 staging -> WorkDir
```

revert → push 한 후 없애기

### Tag

git tag <이름>

git push <저장소> <tag> 저장소에 태그 올리기

태그는 배포되는 단위

만약 v0.0.1 과 v0.0.2 태그를 올렸고, 현재 상태가 v0.0.2이라면

```jsx
git reset --hard v0.0.1 // 소스 상태를 v0.0.1로 변경.
```

태그 없애기

git tag -d <tag-name>

서버에서 없애기

git push <저장소>  -d <tag-name>