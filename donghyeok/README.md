# 2.4 Git의 기초 - 되돌리기

git commit --amend : 완료한 커밋을 수정해야할 때 쓸 수 있음.  예를들어 어떤 파일을 빼먹고 커밋했을 때, 혹은 커밋한 메시지를 수정하고 싶을 때 등 사용할 수 있다.

아래는 예시이다.

```bash
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

'initial commit'을 할 때 forgotten_file을 빼먹어서 forgotten_file을 stage area에 올리고 --amend 옵션을 통해 'initial commit'을 덮어씌웠다.



### 파일 상태를 Unstage로 변경하기

stage area(git add된 file들)에 있는 파일을 unstage할 수 있다.

![image-20210613225213709](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613225213709.png)

위 그림처럼 명령어는 ==git reset HEAD <file>== 이다.

근데 위 그림에서 보면 git restore --staged  <file>로도 unstage할 수 있나보다. 한번 해보겠다.

![image-20210613225444504](https://raw.githubusercontent.com/donghyeok-shin/image_server/main/img/image-20210613225444504.png)

결과는 같다.



