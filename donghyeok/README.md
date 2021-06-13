### 2.4 Git의 기초 - 되돌리기

git commit --amend : 완료한 커밋을 수정해야할 때 쓸 수 있음.  예를들어 어떤 파일을 빼먹고 커밋했을 때, 혹은 커밋한 메시지를 수정하고 싶을 때 등 사용할 수 있다.

아래는 예시이다.

```bash
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

'initial commit'을 할 때 forgotten_file을 빼먹어서 forgotten_file을 stage area에 올리고 --amend 옵션을 통해 'initial commit'을 덮어씌웠다.



#### 파일 상태를 Unstage로 변경하기

stage area(git add된 file들)에 있는 파일을 unstage할 수 있다.

![image-20210613225213709](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613225213709.png)

위 그림처럼 명령어는 ==git reset HEAD <file>== 이다.

근데 위 그림에서 보면 ==git restore --staged  <file>==로도 unstage할 수 있나보다. 한번 해보겠다.

![image-20210613225444504](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613225444504.png)

결과는 같다.



#### Modified 파일 되돌리기

Modified상태의 file을 최신 commit의 file상태로 되돌리는 명령어는 ==git checkout -- <file>==이다. 아래는 실행 결과이다.

![image-20210613230533041](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613230533041.png)

근데 위 그림에서 알 수 있듯이 ==git restore <file>==로도 수정사항을 버릴 수 있나보다. **파일 상태를 Unstage로 변경하기** 쳅터와 다른 부분은 --staged 옵션이 없다는 것이다.

![image-20210613230843000](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613230843000.png)



### 2.5 리모트 저장소

원격 저장소를 의미하며 저장소는 여러개가 있을 수 있다. 원격 저장소라고 하더라도 ==로컬에 존재할 수 있다==. 리모트가 딱히 네트워크 혹은 인터넷 멀리의 저장소를 의미하는 것은 아니다.



#### 리모트 저장소 확인

==git remote==로 확인할 수 있음. 저장소를 clone했을 때 리모트 저장소는 자동으로 등록됨.

-v 옵션을 이용하면 저장소 단축이름과 url을 같이 볼 수 있음. 아래는 git remote -v 결과

```bash
git remote -v
origin  https://github.com/evan-hwang/git-study.git (fetch)
origin  https://github.com/evan-hwang/git-study.git (push)
```

### 

#### 리모트 저장소 추가/삭제/이름 변경

기존 워킹 디렉토리에 새 리모트 저장소를 추가할 수 있음.

==git remote add <단축이름> <url>== 명령을 이용함.

해당 명령어를 테스트해보기 위해서 먼저 프로젝트를 clone하였고, 아래처럼 clone_in_local이라는 단축이름으로 추가하였다.

![image-20210613234115071](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613234115071.png)



==git remote remove <단축이름>== 명령으로 저장소 삭제하기

![image-20210613235444676](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613235444676.png)



git remote rename <단축이름> <바꾸려는 이름> 으로 이름 변경 가능.



#### 리모트 저장소 pull/fetch

clone을 하면 자동으로 리모트 저장소가 ==origin==이라는 이름으로 추가되며, master 브랜치를 추적함.

git fetch origin명령을 실행하면, clone 혹은 마지막으로 fetch/pull한 이후의 수정된 것을 원격 저장소에서 가져옴. ==fetch는 수정사항을 가져오기는 하나, 자동으로 merge되지 않기== 때문에 로컬 작업을 마무리하고 수동 merge해야함.

==pull을 이용하면 자동 merge를 하는것이 fetch와의 차이점==.



#### 리모트 저장소 push

수정사항을 공유하고 싶을 때 upstream 저장소에 push가능.

명령어는 ==git push <리모트 저장소 이름> <브랜치 이름>==



#### 리모트 저장소 살펴보기

명령어는 git remote show <리모트 저장소 이름>

이 명령어는 git pull 명령을 실행할 때 master브랜치와 merge할 브랜치가 무엇인지 보여준다고 함.



### 2.6 태그

#### 태그 조회하기

==git tag== 명령으로 이미 만들어진 태그 조회 가능하며 와일드카드(*)를 사용하려면 -l, --list 옵션을 사용해야함.

```bash
$ git tag -l "v1.8.5*"
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
v1.8.5.1
v1.8.5.2
v1.8.5.3
v1.8.5.4
v1.8.5.5
```



#### 태그 붙이기

태그는 Lightweight, Annotated가 있음.

- Lightweight : 단순히 특정 커밋에 대한 포인터로 커밋 체크섬만 저장함.

  git tag <태그명>으로 명령 실행하며, git show를 실행하면 커밋정보만 볼 수 있음.

- Annotated : git db에 tag를 만든 사람 이름, 이메일, 날짜, 메시지 등을 저장함. 이러한 정보를 유지할 필요가 없거나, 임시  생성 태그인 경우 Lightweight를 사용. -a 옵션을 추가하면 됨. -m을 입력하지 않으면 편집기를 실행시킴.

  ```console
  $ git tag -a v1.4 -m "my version 1.4"
  $ git tag
  v0.1
  v1.3
  v1.4
  ```

  git show를 이용하여 해당 태그 정보 및 커밋 정보를 확인할 수 있음.



#### 이전 커밋 태그

```console
$ git tag -a v1.2 9fceb02
```

위와 같이 해당 커밋의 체크섬을 이용해서 태그할 수 있음.



#### 태그 공유

git push는 tag를 자동으로 서버에 공유하지 않음. 태그를 별도로 push해야함.

==git push origin <태그 이름>== 명령 실행으로 태그를 push함.

한번에 여러개의 tag를 push하고자한다면 --tags 옵션 사용

```console
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
```



#### 태그 checkout

==git checkout <태그 이름>==이며 태그를 checkout했을 때는 "detached HEAD" 상태가 됨. 브랜치 동작 checkout 동작과 다름.

버그 픽스 등 코드 수정은 브랜치를 만들어서 작업하는 것이 좋음.
