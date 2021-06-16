# Git 서버

이 글을 읽는 독자라면 이미 하루 업무의 대부분을 Git으로 처리할 수 있을 거라고 생각한다. 이제는 다른 사람과 협업하는 방법을 고민해보자. 다른 사람과 협업하려면 리모트 저장소가 필요하다. 물론 혼자서 저장소를 만들고 거기에 Push 하고 Pull 할 수도 있지만 이렇게 하는 것은 아무 의미가 없다. 이런 방식으로는 다른 사람이 무슨 일을 하고 있는지 알려면 항상 지켜보고 있어야 간신히 알 수 있을 터이다. 당신이 오프라인일 때도 동료가 저장소를 사용할 수 있게 하려면 언제나 이용할 수 있는 저장소가 필요하다. 즉, 공동으로 사용할 수 있는 저장소를 만들고 모두 이 저장소에 접근하여 Push, Pull 할 수 있어야 한다.

Git 서버를 운영하는 건 매우 간단하다. 우선 사용할 전송 프로토콜부터 정한다. 이 장의 앞부분에서는 어떤 프로토콜이 있는지 그리고 각 장단점은 무엇인지 살펴본다. 그다음엔 각 프로토콜을 사용하는 방법과 그 프로토콜을 사용할 수 있도록 서버를 구성하는 방법을 살펴본다. 마지막으로 **다른 사람의 서버에 내 코드를 맡기긴 싫고 고생스럽게 서버를 설치하고 관리하고 싶지도 않을 때 고를 수 있는 선택지가 어떤 것들이 있는지 살펴본다.**

**서버를 직접 설치해서 운영할 생각이 없으면** 이 장의 마지막 절만 읽어도 된다. 마지막 절에서는 Git 호스팅 서비스에 계정을 만들고 사용하는 방법에 대해 설명한다. 그리고 다음 장에서는 분산 환경에서 소스를 관리하는 다양한 패턴에 대해 논의할 것이다.

리모트 저장소는 일반적으로 워킹 디렉토리가 없는 *Bare 저장소* 이다. 이 저장소는 협업용이기 때문에 체크아웃이 필요 없다. 그냥 Git 데이터만 있으면 된다. 다시 말해서 Bare 저장소는 일반 프로젝트에서 `.git` 디렉토리만 있는 저장소다.



## 4.1 프로토콜

Git은 **Local, HTTP, SSH, Git 이렇게 네 가지의 프로토콜을 사용**할 수 있다. 이 절에서는 각각 어떤 경우에 유용한지 살펴본다.



### 로컬 프로토콜

가장 기본적인 것이 *로컬 프로토콜* 이다. 리모트 저장소가 단순히 같은 시스템의 다른 디렉토리에 있을 때 사용한다. 팀원들이 전부 한 시스템에 로그인하여 개발하거나 아니면 NFS같은 것으로 파일시스템을 공유하고 있을 때 사용한다. 이런 상황은 문제가 될 수 있다. 모든 저장소가 한 시스템에 있기 때문에 한순간에 모두 잃을 수 있다.

공유 파일시스템을 마운트했을 때는 로컬 저장소를 사용하는 것처럼 Clone 하고 Push 하고 Pull 하면 된다. 일단 저장소를 Clone 하거나 프로젝트에 리모트 저장소로 추가한다. 추가할 때 URL 자리에 저장소의 경로를 사용한다. 예를 들어 아래와 같이 로컬 저장소를 Clone 한다.

```console
$ git clone /srv/git/project.git
```

아래처럼도 가능하다:

```console
$ git clone file:///srv/git/project.git
```

Git은 파일 경로를 직접 쓸 때와 `file://` 로 시작하는 URL을 사용할 때를 약간 다르게 처리한다. 디렉토리 경로를 그대로 사용하면 Git은 필요한 파일을 직접 복사하거나 하드 링크를 사용한다. 하지만 `file://` 로 시작하면 Git은 네트워크를 통해서 데이터를 전송할 때처럼 프로세스를 별도로 생성하여 처리한다. 이 프로세스로 데이터를 전송하는 것은 효율이 좀 떨어지지만 그래도 `file://` 를 사용하는 이유가 있다. 이것은 외부 Refs나 개체들이 포함된 저장소의 복사본을 깨끗한 상태로 남겨두고자 함이다. 보통은 다른 버전 관리 시스템들에서 임포트한 후에 사용한다([Git의 내부](https://git-scm.com/book/ko/v2/ch00/ch10-git-internals)에서 자세히 다룬다). 여기서는 속도가 빠른 디렉토리 경로를 사용한다.

이미 가진 Git 프로젝트에는 아래와 같이 로컬 저장소를 추가한다.

```console
$ git remote add local_proj /srv/git/project.git
```

그러면 네트워크에 있는 리모트 저장소처럼 `local_proj` 이름으로 리모트처럼 Push 하거나 Pull 할 수 있다.



#### 장점

파일 기반 저장소의 장점은 간단하다는 것이다. 기존에 있던 네트워크나 파일의 권한을 그대로 사용하기 때문에 설정하기 쉽다. 이미 팀 전체가 접근할 수 있는 파일시스템을 가지고 있다면 저장소를 아주 쉽게 구성할 수 있다. 다른 디렉토리를 공유할 때처럼 모든 동료가 읽고 쓸 수 있는 공유 디렉토리에 Bare 저장소를 만들면 된다. 다음 절인 [서버에 Git 설치하기](https://git-scm.com/book/ko/v2/ch00/_getting_git_on_a_server)에서 Bare 저장소를 만드는 방법을 살펴볼 것이다.

또한, 동료가 작업하는 저장소에서 한 일을 바로 가져오기에도 좋다. 만약 함께 프로젝트를 하는 동료가 자신이 한 일을 당신이 확인해 줬으면 한다. 이럴 때 `git pull /home/john/project` 처럼 명령어를 실행시켜서 매우 쉽게 동료의 코드를 가져올 수 있다. 그 동료가 서버에 Push 하고 당신이 다시 Pull 할 필요 없다.



#### 단점

다양한 상황에서 접근할 수 있도록 디렉토리를 공유하는 것 자체가 일반적으로 어렵다. 집에 있을 때 Push 해야 하면 리모트 저장소가 있는 디스크를 마운트해야 하는데 이것은 다른 프로토콜을 이용하는 방법보다 느리고 어렵다.

게다가 파일시스템을 마운트해서 사용하는 중이라면 별로 빠르지도 않다. 로컬 저장소는 데이터를 빠르게 읽을 수 있을 때만 빠르다. NFS에 있는 저장소에 Git을 사용하는 것은 보통 같은 서버에 SSH로 접근하는 것보다 느리다.

마지막으로 이 프로토콜은 저장소에 우발적인 사고가 발생하지 않도록 보호해주지 않는다. 모든 사용자는 쉘에서 ``리모트'' 디렉토리에 무슨 짓이든지 할 수 있다. 누군가 저장소에 침범해서 Git 내부 파일을 삭제하고 변경하지 못하도록 하는 장치가 없다.



### HTTP 프로토콜

Git은 HTTP로 통신할 때, 서로 다른 두 방법으로 HTTP를 사용할 수 있다. 1.6.6 이전 버전에서는 읽기만 가능한 단순한 방법밖에 사용할 수 없었다. 1.6.6 버전부터는 똑똑한 프로토콜을 사용할 수 있다. 이 프로토콜은 Git 데이터를 전송할 때 SSH처럼 서로 협상한다. 새로운 HTTP 프로토콜은 사용이 쉽고 기능도 좋아서 많은 사람들이 사용하고 있다. 이 프로토콜을 보통 *스마트* HTTP 프로토콜이라 하고 예전의 HTTP 프로토콜을 *멍청한* HTTP 프로토콜이라고 한다. 먼저 스마트 HTTP 프로토콜을 설명한다.



#### 스마트 HTTP

스마트 HTTP 프로토콜은 SSH나 Git 프로토콜처럼 통신한다. 다만 HTTP나 HTTPS 포트를 이용해 통신하고 다양한 HTTP 인증 방식을 사용한다는 것이 다르다. SSH는 키를 발급하고 관리해야 하는 번거로움이 있지만, HTTP는 사용자이름과 암호만으로 인증할 수 있기 때문에 더 편리하게 사용할 수 있다.

아마 지금은 Git에서 가장 많이 사용하는 프로토콜일 것이다. `git://` 프로토콜처럼 익명으로 사용할 수도 있고, SSH처럼 인증을 거쳐 Push 할 수도 있기 때문이다. 이 두 가지 동작을 다른 URL로 나눌 필요없이 하나의 URL로 통합해서 사용할 수 있다. 그냥 인증기능을 켜놓은 저장소에 Push를 하면 서버는 사용자이름과 암호를 물어본다. 그리고 Fetch나 Pull 같은 읽기 작업에서도 같은 URL을 사용한다.

실제로 GitHub 같은 서비스에서 제공하는 저장소는 Clone을 할 때나 Push를 할 때 같은 URL을 사용한다. (예, https://github.com/schacon/simplegit)



#### 멍청한 HTTP

Git 서버가 스마트 HTTP 요청에 응답하지 않으면 Git 클라이언트는 차선책으로 *멍청한* HTTP 프로토콜을 시도한다. 이 멍청한 프로토콜은 원격 저장소를 그냥 파일 건네주는 웹 서버로 취급한다. HTTP와 HTTPS 프로토콜은 아름다울 정도로 설정이 간단하다. HTTP 도큐먼트 루트 밑에 Bare 저장소를 두고 `post-update` 훅을 설정하는 것이 기본적으로 해야 하는 일의 전부다([Git Hooks](https://git-scm.com/book/ko/v2/ch00/_git_hooks) 에서 자세히 다룰 것이다). 저장소가 있는 웹 서버에 접근할 수 있다면 그 저장소를 Clone 할 수도 있다. 아래와 같이 HTTP를 통해서 저장소를 읽을 수 있게 한다.

```console
$ cd /var/www/htdocs/
$ git clone --bare /path/to/git_project gitproject.git
$ cd gitproject.git
$ mv hooks/post-update.sample hooks/post-update
$ chmod a+x hooks/post-update
```

다 됐다. `post-update` 훅은 Git에 포함되어 있으며 `git update-server-info` 라는 명령어를 실행시킨다. 이 명령어를 써야 HTTP로 Fetch와 Clone 명령이 제대로 동작한다. 누군가 SSH를 통해서 저장소에 Push 하면 `post-update` 훅이 실행된다. 그럼 다른 사용자들은 Push 된 파일을 아래와 같이 Clone 할 수 있다.

```console
$ git clone https://example.com/gitproject.git
```

여기서는 Apache 서버를 사용해서 기본 루트 디렉토리인 `/var/www/htdocs` 를 사용하지만 다른 웹 서버를 사용해도 된다. 단순히 Bare 저장소를 HTTP 문서 루트에 넣으면 된다. Git 데이터는 일반적인 정적 파일처럼 취급된다([Git의 내부](https://git-scm.com/book/ko/v2/ch00/ch10-git-internals) 에서 정확히 어떻게 처리하는지 다룬다).

보통은 스마트 HTTP 프로토콜만 이용하거나 멍청한 HTTP 프로토콜만 사용한다. 이 둘을 한꺼번에 사용하는 경우는 드물다.



#### 장점

스마트 HTTP 프로토콜의 장점만 보자

읽기와 쓰기에 하나의 URL만 사용한다. 그리고 사용자에게 익숙한 사용자이름과 암호 방식의 인증을 사용한다. 사용자이름과 암호 방식의 인증이 SSH에 비해 사용하기 간단하다. SSH는 사용자가 알아서 키를 만들고 공개키를 서버에 올린 후에야 비로소 인증을 받을 수 있다. SSH에 대해 잘 모르거나 익숙지 않은 사용자를 생각하면 이런 사용성은 엄청난 장점이다. 게다가 SSH만큼이나 빠르고 효율적이기 까지 하다.

HTTPS를 이용해서 전송하는 데이터를 암호화하는 것도, 클라이언트에게 서명된 SSL 인증서를 요구하는 것도 가능하다.

HTTPS는 매우 보편적인 프로토콜이기 때문에 거의 모든 회사 방화벽에서 통과하도록 돼있다는 장점도 있다.



#### 단점

HTTPS를 사용하도록 설정하는 것이 SSH로 설정하는 것보다 까다로운 서버가 있다. 그것 말고는 스마트 HTTP 프로토콜이 다른 프로토콜보다 못한 단점은 별로 없다.

Push 할 때 HTTP 인증을 사용하면 SSH 인증키를 사용하는 것보다 좀 더 복잡하다. 그래도 인증 캐싱 툴을 사용하면 좀 낫다. OSX에는 키체인(Keychain Access)이, Windows에는 인증서 관리자(Credential Manager)가 있다. HTTP 암호 캐싱 설정에 대한 더 자세한 사항은 [Credential 저장소](https://git-scm.com/book/ko/v2/ch00/_credential_caching) 를 참고하길 바란다.



### SSH 프로토콜

Git의 대표 프로토콜은 SSH이다. SSH를 이용하면 아무런 외부 도구 없이 Git 서버를 구축할 수 있다. 대부분 서버는 SSH로 접근할 수 있도록 설정돼 있다. 뭐, 설정돼 있지 않더라도 쉽게 설정할 수 있다. 그리고 SSH는 인증 기능이 있고 어디에서든 사용할 수 있으며 사용하기도 쉽다.

SSH를 통해 Git 저장소를 Clone 하려면 `ssh://` 로 시작하는 URL을 사용한다:

```console
$ git clone ssh://[user@]server/project.git
```

아래와 같은 SCP 형태의 구문으로 줄여 쓸 수도 있다.

```console
$ git clone [user@]server:project.git
```

사용자 계정을 생략할 수도 있는데 계정을 생략하면 Git은 현재 로그인한 사용자의 계정을 사용한다.



#### 장점

SSH 장점은 정말 많다. 첫째, SSH는 상대적으로 설정하기 쉽다. SSH 데몬은 정말 흔하다. 많은 네트워크 관리자들은 SSH 데몬을 다루어 본 경험이 있고 대부분의 OS 배포판에는 SSH 데몬과 관리도구들이 모두 들어 있다. 둘째, SSH를 통해 접근하면 보안에 안전하다. 모든 데이터는 암호화되어 인증된 상태로 전송된다. 마지막으로 SSH는 HTTPS, Local 프로토콜과 마찬가지로 전송 시 데이터를 가능한 압축하기 때문에 효율적이다.



#### 단점

SSH는 익명으로 접근할 수 없다. 심지어 읽기 전용인 경우에도 익명으로 시스템에 접근할 수 없다. 회사에서만 사용할 거라면 SSH가 가장 적합한 프로토콜일 것이지만 오픈소스 프로젝트는 SSH만으로는 부족하다. 만약 SSH를 사용하는 프로젝트에 익명으로 접근할 수 있게 하려면, Push 할 때는 SSH로 하고 다른 사람들이 Fetch 할 때는 다른 프로토콜을 사용하도록 설정해야 한다.



### Git 프로토콜

Git 프로토콜은 Git에 포함된 데몬을 사용하는 것이다. 포트는 9418이며 SSH 프로토콜과 비슷한 서비스를 제공하지만, 인증 메커니즘이 없다. 저장소에 git-export-daemon-ok 파일을 만들면 Git 프로토콜로 서비스할 수 있지만, 보안은 없다. 이 파일이 없는 저장소는 서비스되지 않는다. 이 저장소는 누구나 Clone 할 수 있거나 아무도 Clone 할 수 없거나 둘 중의 하나만 선택할 수 있다. 그래서 이 프로토콜로는 Push 하게 할 수 없다. 엄밀히 말하자면 Push 할 수 있도록 설정할 수 있지만, 인증하도록 할 수 없다. 그러니까 당신이 Push 할 수 있으면 이 프로젝트의 URL을 아는 사람은 누구나 Push 할 수 있다. 그냥 이런 것도 있지만 잘 안 쓴다고 알고 있으면 된다.



#### 장점

Git 프로토콜은 전송 속도가 가장 빠르다고 할 수 있다. 전송량이 많은 공개 프로젝트나 별도의 인증이 필요 없고 읽기만 허용하는 프로젝트를 서비스할 때 유용하다. 암호화와 인증을 빼면 SSH 프로토콜과 전송 메커니즘이 별반 다르지 않다.



#### 단점

Git 프로토콜의 단점은 인증 메커니즘이 없다는 것이다. Git 프로토콜만으로 접근할 수 있는 프로젝트는 바람직하지 못하다. 일반적으로 SSH나 HTTPS 프로토콜과 함께 사용한다. 소수의 개발자만 Push 할 수 있고 대다수 사람은 `git://` 을 사용하여 읽을 수만 있게 하는 것이다. 어쩌면 가장 설치하기 어려운 방법일 수도 있다. 별도의 데몬이 필요하고 프로젝트에 맞게 설정해야 한다. 자원을 아낄 수 있도록 xinetd 같은 것도 설정해야 하고 방화벽을 통과할 수 있도록 9418 포트도 열어야 한다. 이 포트는 일반적으로 회사들이 허용하는 표준 포트가 아니다. 규모가 큰 회사의 방화벽이라면 당연히 이 포트를 막아 놓는다.



## 4.2 서버에 Git 설치하기

서버에 Git을 설치해서 공개하는 방법을 알아보자.

| 노트 | 여기에서는 Linux에 설치하는 방법에 대해서만 간단히 설명할 것이다. 물론 Mac과 Windows에도 설치할 수 있다. 실제로 서버에 Git을 설치하고 설정하려면 온갖 보안 조치를 설정하고 OS 도구들을 써야 한다. 그 모든 것을 이 글에서 다루진 않지만 무엇에 신경 써야 하는지는 알 수 있을 것이다. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

어떤 서버를 설치하더라도 일단 저장소를 Bare 저장소로 만들어야 한다. 다시 말하지만, Bare 저장소는 워킹 디렉토리가 없는 저장소이다. `--bare` 옵션을 주고 Clone 하면 새로운 Bare 저장소가 만들어진다. Bare 저장소 디렉토리는 관례에 따라 `.git` 글자가 이름에 붙는다.

```console
$ git clone --bare my_project my_project.git
Cloning into bare repository 'my_project.git'...
done.
```

이제 `my_project.git` 디렉토리에는 복사한 Git 디렉토리 데이터만 들어 있다.

아래와 같이 실행한 것과 비슷하다:

```console
$ cp -Rf my_project/.git my_project.git
```

물론 설정상의 미세한 차이가 있지만, 저장소의 내용만 고려한다면 같다고 볼 수 있다. 워킹 디렉토리가 없는 Git 저장소인 데다가 별도의 디렉토리도 하나 만들었다는 점에서는 같다.



### 서버에 Bare 저장소 넣기

Bare 저장소는 이제 만들었으니까 서버에 넣고 프로토콜을 설정한다. `git.example.com` 라는 이름의 서버를 하나 준비하자. 그리고 그 서버에 SSH로 접속할 수 있게 만들고 Git 저장소를 `/srv/git` 디렉토리에 저장할 것이다. 서버에 `/srv/git` 디렉토리가 있다고 가정하고 아래와 같이 Bare 저장소를 복사한다.

```console
$ scp -r my_project.git user@git.example.com:/srv/git
```

이제 다른 사용자들은 SSH로 서버에 접근해서 저장소를 Clone 할 수 있다. 사용자는 `/srv/git` 디렉토리에 읽기 권한이 있어야 한다.

```console
$ git clone user@git.example.com:/srv/git/my_project.git
```

이 서버에 SSH로 접근할 수 있는 사용자가 `/srv/git/my_project.git` 디렉토리에 쓰기 권한까지 가지고 있으면 바로 Push 할 수 있다.

`git init` 명령에 `--shared` 옵션을 추가하면 Git은 자동으로 그룹 쓰기 권한을 추가한다.

```console
$ ssh user@git.example.com
$ cd /srv/git/my_project.git
$ git init --bare --shared
```

Git 저장소를 만드는 것이 얼마나 쉬운지 살펴보았다. Bare 저장소를 만들어 SSH로 접근할 수 있는 서버에 올리면 동료와 함께 일할 준비가 끝난다.

그러니까 Git 서버를 구축하는데 사람이 할 일은 정말 별로 없다. SSH로 접속할 수 있도록 서버에 계정을 만들고 Bare 저장소를 사람들이 읽고 쓸 수 있는 곳에 넣어 두기만 하면 된다. 이제 준비됐다. 더 필요한 것은 없다.

다음 절에서는 좀 더 정교하게 설정하는 법을 살펴볼 것이다. 사용자에게 계정을 만들어 주는 법, 저장소를 읽고 쓸 수 있게 하는 법, 웹 UI를 설정하는 법 등은 여기에서 설명하지 않는다. 동료와 함께 개발할 때 꼭 필요한 것은 SSH 서버와 Bare 저장소뿐이라는 것만은 꼭 기억하자.



### 초 간단 뚝딱

만약 창업을 준비하고 있거나 회사에서 Git을 막 도입하려고 할 때는 개발자의 수가 많지 않아서 설정할 게 별로 없다. 사용자를 관리하는 것이 Git 서버를 설정할 때 가장 골치 아픈 것 중 하나다. 사람이 많으면 어떤 사용자는 읽기만 가능하게 하고 어떤 사용자는 읽고 쓰기 둘 다 가능하게 해야 한다. 이렇게 설정하는 것은 조금 더 까다롭다.



#### SSH 접속

만약 모든 개발자가 SSH로 접속할 수 있는 서버가 있으면 너무 쉽게 저장소를 만들 수 있다. 앞서 말했듯이 정말 할 일이 별로 없다. 그리고 저장소의 권한을 꼼꼼하게 관리해야 하면 운영체제의 파일시스템 권한관리를 이용할 수 있다.

동료가 저장소에 쓰기 접근을 해야 하는 데 아직 SSH로 접속할 수 있는 서버가 없으면 하나 마련해야 한다. 아마 당신에게 서버가 하나 있다면 그 서버에는 이미 SSH 서버가 설치되어 있고 지금도 SSH로 접속하고 있을 것이다.

팀원들이 접속할 수 있도록 하는 방법은 몇 가지가 있다. 첫째로 모두에게 계정을 만들어 주는 방법이 있다. 이 방법이 제일 단순하지만 다소 귀찮다. 팀원마다 `adduser` 를 실행시키고 임시 암호를 부여해야 하기 때문에 보통 이 방법을 쓰고 싶어 하지 않는다.

둘째로 서버마다 'git’이라는 계정을 하나씩 만드는 방법이 있다. 쓰기 권한이 필요한 사용자의 SSH 공개키를 모두 모아서 'git' 계정의 `~/.ssh/authorized_keys` 파일에 모든 키를 입력한다. 그러면 모두 'git' 계정으로 그 서버에 접속할 수 있다. 이 'git' 계정은 커밋 데이터에는 아무런 영향을 끼치지 않는다. 다시 말해서 접속하는 데 사용한 SSH 계정과 커밋에 저장되는 사용자는 아무 상관없다.

SSH 서버 인증을 LDAP 서버를 이용할 수도 있다. 이미 사용하고 있는 중앙집중식 인증 소스가 있으면 해당 인증을 이용하여 SSH 서버에 인증하도록 할 수도 있다. SSH 인증 메커니즘 중 아무거나 하나 이용할 수 있으면 사용자는 그 서버에 접근할 수 있다.



## 4.3 SSH 공개키 만들기

많은 Git 서버들은 SSH 공개키로 인증한다. 공개키를 사용하려면 일단 공개키를 만들어야 한다. 공개키를 만드는 방법은 모든 운영체제가 비슷하다. 먼저 키가 있는지부터 확인하자. 사용자의 SSH 키들은 기본적으로 사용자의 `~/.ssh` 디렉토리에 저장한다. 그래서 만약 디렉토리의 파일을 살펴보면 이미 공개키가 있는지 확인할 수 있다.

```console
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

id_dsa나 id_rsa라는 파일 이름이 보일 것이고 이에 같은 파일명의 `.pub` 라는 확장자가 붙은 파일이 하나 더 있을 것이다. 그중 `.pub` 파일이 공개키이고 다른 파일은 개인키다. 만약 이 파일들이 없거나 `.ssh` 디렉토리도 없으면 `ssh-keygen` 이라는 프로그램으로 키를 생성해야 한다. `ssh-keygen` 프로그램은 Linux나 Mac의 SSH 패키지에 포함돼 있고 Windows는 'Git for Windows' 안에 들어 있다.

```console
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

`.ssh/id_rsa` 키를 저장하고 싶은 디렉토리를 입력하고 암호를 두 번 입력한다. 이때 암호를 비워두면 키를 사용할 때 암호를 묻지 않는다.

다음은 사용자가 자신의 공개키를 Git 서버 관리자에게 보내야 한다. 사용자는 `.pub` 파일의 내용을 복사하여 이메일을 보내기만 하면 된다. 공개키는 아래와 같이 생겼다.

```console
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@mylaptop.local
```

다른 운영 체제에서 SSH 키를 만드는 방법이 궁금하면 https://help.github.com/articles/generating-ssh-keys에 있는 GitHub 설명서를 찾아보는 게 좋다.



## 4.4 서버 설정하기

서버에서 설정하는 일을 살펴보자. 일단 Ubuntu같은 표준 Linux 배포판을 사용한다고 가정한다. 사용자들은 아마도 `authorized_keys` 파일로 인증할 것이다.

| 노트 | `ssh-copy-id` 명령을 사용하면 여기에서 설명하는 SSH 공개키를 복사하고 설치하는 내용을 자동화 한 도구이다. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

먼저 `git` 계정을 만들고 사용자 홈 디렉토리에 `.ssh` 디렉토리를 만든다:

```console
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

`authorized_keys` 파일에 SSH 공개키를 추가해야 사용자가 접근할 수 있다. 추가하기 전에 이미 알고 있는 사람의 공개키를 받아서 가지고 있다고 가정한다. 공개키가 어떻게 생겼는지 다시 한번 확인하자.

```console
$ cat /tmp/id_rsa.john.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
dAv8JggJICUvax2T9va5 gsg-keypair
```

`.ssh` 디렉토리에 있는 `authorized_keys` 파일에 추가한다:

```console
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys
```

`--bare` 옵션을 주고 `git init` 를 실행해서 워킹 디렉토리가 없는 빈 저장소를 하나 만든다.

```console
$ cd /srv/git
$ mkdir project.git
$ cd project.git
$ git init --bare
Initialized empty Git repository in /srv/git/project.git/
```

이제 John, Josie, Jessica는 이 저장소를 리모트 저장소로 등록하고 나서 브랜치를 Push 할 수 있다. 프로젝트마다 적어도 한 명은 서버에 접속해서 Bare 저장소를 만들어야 한다. `git` 계정과 저장소를 만든 서버의 호스트 이름이 `gitserver` 라고 하자. 만약 이 서버가 내부망에 있고 `gitserver` 가 그 서버를 가리키도록 DNS에 설정하면 아래와 같은 명령을 사용할 수 있다(`myproject` 프로젝트가 이미 있다고 가정한다).

```console
# on John's computer
$ cd myproject
$ git init
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin git@gitserver:/srv/git/project.git
$ git push origin master
```

이제 이 프로젝트를 Clone 하고 나서 수정하고 Push 할 수 있다.

```console
$ git clone git@gitserver:/srv/git/project.git
$ cd project
$ vim README
$ git commit -am 'fix for the README file'
$ git push origin master
```

이렇게 개발자들이 읽고 쓸 수 있는 Git 서버를 쉽게 만들 수 있다.

이 개발자들은 서버에 `git` 계정으로 로그인할 수 있다. 이를 막으려면 `passwd` 파일에서 로그인 셸을 바꿔야한다.

단순히 로그인 셸을 `git-shell` 로 바꾸기만 하면 `git` 계정으로는 `git` 만 사용할 수 있다. 이 로그인 셸은 서버의 다른 부분은 건들 수 없도록 돼있다. `git-shell` 을 사용자의 로그인 셸로 지정해야 한다. `/etc/shells` 에 `git-shell` 를 추가한다. 아래를 보자.

```console
$ cat /etc/shells   # 이미 `git-shell` 이 등록돼 있는지 확인
$ which git-shell   # git-shell 실행파일이 설치돼 있는지 확인
$ sudo vim /etc/shells  # 바로 위 명령으로 확인한 git-shell 실행파일의 절대경로를 추가
```

`chsh <계정이름> -s <shell>` 명령어를 이용해서 특정 계정의 셸을 바꿀 수 있다.

```console
$ sudo chsh git  # git-shell 경로를 입력, 보통 /usr/bin/git-shell 임
```

이제 *git* 계정으로 Push 와 Pull 할 수 있지만 서버의 셸은 가질 수 없다. 로그인하려고 하면 아래와 같이 로그인 불가능 메시지만 보게 될 것이다.

```console
$ ssh git@gitserver
fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access.
Connection to gitserver closed.
```

이제 Git은 제대로 동작하면서 개발자들이 셸을 얻지 못하게 되었다. 위의 출력에서 볼 수 있듯이 `git` 계정의 홈 디렉토리에 `git-shell-commands` 디렉토리를 만들어 `git-shell` 의 동작을 조금 바꿀 수 있다. 예를 들면 서버에서 사용할 수 있는 Git 명령어를 제한할 수 있다. 또 명령어를 실행했을 때 나오는 메시지도 변경 할 수 있다. `git help shell` 명령어를 실행하면 Git 셸을 꾸미는 데에 필요한 정보를 얻을 수 있다.



## 4.5 Git 데몬

여기선 ``Git'' 프로토콜로 동작하는 데몬 설정 방법을 알아본다. 이 방법은 인증 기능이 없는 Git 저장소를 만들 수 있는 가장 빠른 방법이다. 다시 한 번 강조하지만, 인증 기능이 없다. 전 세계 누구든지 데이터에 접근할 수 있다는 뜻이다.

만약 서버가 외부에 그냥 노출돼 있다면 우선 방화벽으로 보호하고 프로젝트를 외부에서 접근할 수 있게 만들어야 한다. 그리고 이미 서버를 방화벽으로 보호하고 있어도 사람이나 컴퓨터(CI 서버나 빌드 서버)가 읽기 접근을 할 수 있도록 SSH 키를 일일이 추가하고 싶지 않을 것이다.

어쨌든 Git 프로토콜은 상대적으로 설치하기 쉽다. 그냥 데몬을 실행하면 된다.

```console
$ git daemon --reuseaddr --base-path=/srv/git/ /srv/git/
```

`--reuseaddr` 는 서버가 기존의 연결이 타임아웃될 때까지 기다리지 말고 바로 재시작하게 하는 옵션이다. `--base-path` 옵션을 사용하면 사람들이 프로젝트를 Clone 할 때 전체 경로를 사용하지 않아도 된다. 그리고 마지막에 있는 경로는 노출할 저장소의 위치를 Git 데몬에 알려주는 것이다. 마지막으로 방화벽을 사용하고 있으면 9418 포트를 열어서 지금 작업하는 서버의 숨통을 틔워주어야 한다.

운영체제에 따라 Git 데몬을 실행시키는 방법은 다르다.

대개의 리눅스 배포판은 `systemd`를 가장 보편적으로 사용하며 이를 이용하는 방법이 가장 일반적이다. 아래의 내용으로 `/etc/systemd/system/git-daemon.service` 파일을 작성한다.

```console
[Unit]
Description=Start Git Daemon

[Service]
ExecStart=/usr/bin/git daemon --reuseaddr --base-path=/srv/git/ /srv/git/

Restart=always
RestartSec=500ms

StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=git-daemon

User=git
Group=git

[Install]
WantedBy=multi-user.target
```

여기서 주의해서 봐야 할 부분은 `git` 이라는 사용자와 그룹을 사용하여 Git 데몬이 실행된다는 점이다. 운영하는 환경에 따라 이 부분을 이미 존재하는 사용자나 그룹을 지정해서 사용할 수도 있다. 위의 예제에서는 Git 실행 파일의 위치가 `/usr/bin/git` 으로 설정되어 있으나 다른곳에 위치해있다면 변경해주어야 한다.

마지막으로 `systemctl enable git-daemon` 명령을 실행해서 시스템이 부팅될 때 자동으로 서비스가 시작되고, 시스템이 종료될 때 자동으로 서비스도 종료 되도록 설정한다. `systemctl start git-daemon`, `systemctl stop git-daemon` 두 명령으로도 설정할 수 있다.

우분투 LTS 14.04 까지는 Upstart 구성을 사용한다. 따라서 14.04 이하의 버전이라면 Upstart 스크립트를 사용한다. 우선 아래와 같이 파일을 만든다.

```console
/etc/init/local-git-daemon.conf
```

아래의 내용을 입력한다.

```console
start on startup
stop on shutdown
exec /usr/bin/git daemon \
    --user=git --group=git \
    --reuseaddr \
    --base-path=/srv/git/ \
    /srv/git/
respawn
```

보안을 위해서 저장소를 읽을 수만 있는 사용자로 데몬을 실행시킬 것을 강력하게 권고한다. `git-ro` 라는 계정을 새로 만들고 그 계정으로 데몬을 실행시키는 것이 좋다. 하지만 여기에서는 쉽게 설명하려고 `git-shell`을 실행하는 동일한 사용자인 `git` 계정을 사용한다.

서버가 재시작할 때 Git 데몬이 자동으로 실행되고 데몬이 죽어도 자동으로 재시작될 것이다. 서버는 놔두고 Git 데몬만 재시작할 수 있다.

```console
$ initctl start local-git-daemon
```

다른 시스템에서는 `sysvinit` 시스템의 `xinetd` 스크립트를 사용하거나 자신만의 방법으로 해야 한다.

아무나 읽을 수 있다는 것을 Git 서버에 알려주어야 한다. 저장소에 `git-daemon-export-ok` 파일을 만들면 된다.

```console
$ cd /path/to/project.git
$ touch git-daemon-export-ok
```

이 파일이 있으면 Git 데몬은 인증 없이 프로젝트를 노출하는 것으로 판단한다.



## 4.6 스마트 HTTP

지금까지 인증 기능을 갖춘 SSH와 인증 기능이 없는 git 프로토콜을 배웠다. 이제는 이 두 기능을 한꺼번에 가진 프로토콜을 알아보자. 서버에서 `git-http-backend` 명령어를 이용해 일단 기본적인 스마트 HTTP를 지원하는 Git 서버를 실행한다. Git 클라이언트에서 `git fetch` 나 `git push` 를 실행하면 서버로 HTTP 요청을 보낸다. 서버는 그 요청을 보고 경로와 헤더를 읽어 클라이언트가 HTTP로 통신하려 하는지 감지한다. 이는 1.6.6 버전 이상의 클라이언트에서 동작한다. 서버는 클라이언트가 스마트 HTTP 프로토콜을 지원한다고 판단되면 스마트 HTTP 프로토콜을 사용하고 아니면 멍청한 프로토콜을 계속 사용한다. 덕분에 하위 호환성이 잘 유지된다.

이제 설정해보자. CGI 서버로 Apache를 사용한다. Apache가 없다면 Linux에서는 아래와 같이 Apache를 설치할 수 있다.

```console
$ sudo apt-get install apache2 apache2-utils
$ a2enmod cgi alias env
```

이 명령어 한 방이면 `mod_cgi`, `mod_alias`, `mod_env` 도 사용할 수 있다. 다 앞으로 사용할 모듈들이다.

`/srv/git` 디렉토리의 Unix 사용자 그룹도 `www-data` 로 설정해야 한다. 그래야 웹 서버가 저장소를 읽고 쓸 수 있다. Apache 인스턴스는 CGI 스크립트를 이 사용자로 실행시킨다(기본 설정이다).

```console
$ chgrp -R www-data /srv/git
```

그리고 Apache 설정 파일을 수정한다. 그러면 `git http-backend` 를 실행했을 때 모든 요청을 `/git` 경로로 받을 수 있다.

```console
SetEnv GIT_PROJECT_ROOT /srv/git
SetEnv GIT_HTTP_EXPORT_ALL
ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
```

`GIT_HTTP_EXPORT_ALL` 환경 변수를 설정하지 않으면 `git-daemon-export-ok` 파일이 있는 저장소에는 아무나 다 접근할 수 있게 된다. 그냥 Git 데몬의 동작과 똑같다.

마지막으로 Apache가 `git-http-backend` 에 요청하는 것을 허용하고 쓰기 접근 시 인증하게 한다.

```console
<Files "git-http-backend">
    AuthType Basic
    AuthName "Git Access"
    AuthUserFile /srv/git/.htpasswd
    Require expr !(%{QUERY_STRING} -strmatch '*service=git-receive-pack*' || %{REQUEST_URI} =~ m#/git-receive-pack$#)
    Require valid-user
</Files>
```

`.htpasswd` 파일에는 접근을 허가하려는 사용자의 암호가 들어가 있어야 한다. 아래는 ``schacon'' 이란 사용자를 추가하는 방법이다.

```console
$ htpasswd -c /srv/git/.htpasswd schacon
```

Apache에는 사용자 인증 방법이 많다. 그중 하나를 골라 사용해야 하는데 위에 설명한 방법이 가장 간단한 방법의 하나다. 그리고 이렇게 사용자 인증 설정을 할 때는 보안을 위해 SSL로 접속해 작업하는 것이 좋다.

웹 서버는 Apache 말고도 다른 서버를 사용할 수도 있고, 인증 방식도 다르므로 Apache 설정에 대해서 길게 이야기하지 않는다. 대신 이것만 알아두었으면 한다. HTTP를 이용한 모든 통신에서는 `git http-backend` 와 Git을 함께 사용한다는 것이다. Git 그 자체로는 인증 기능을 가지고 있지 않다. 하지만 웹 서버의 인증 레이어와 손쉽게 연동할 수 있다. CGI를 실행할 수 있는 웹 서버라면 어떤 서버든지 붙일 수 있다. 가장 좋아하는 서버를 사용하길 바란다.

| 노트 | Apache 웹 서버에서 인증 설정에 대해 더 자세히 알아보려면 Apache 문서를 참고하길 바란다.(http://httpd.apache.org/docs/current/howto/auth.html) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



## 4.7 GitWeb

프로젝트 저장소를 단순히 읽거나 쓰는 것에 대한 설정은 다뤘다. 이제는 웹 기반 인터페이스를 설정해 보자. Git은 웹에서 저장소를 조회할 수 있는 GitWeb이라는 CGI 스크립트를 제공한다.

![Git 웹용 UI](https://git-scm.com/book/en/v2/images/git-instaweb.png)

그림 49. Git 웹용 UI, GitWeb

Git은 GitWeb을 쉽게 사용해 볼 수 있도록 서버를 즉시 띄우는 명령을 제공한다. 시스템에 `lighttpd` 나 `webrick` 같은 경량 웹 서버가 설치돼 있어야 이 명령을 사용할 수 있다. Linux에서는 `lighttpd` 를 많이 사용한다. `lighttpd` 가 설치돼 있으면 프로젝트 디렉토리에서 그냥 `git instaweb` 을 실행하면 바로 실행될 것이다. Mac의 Leopard 버전은 Ruby가 미리 설치돼 있기 때문에 `webrick` 이 더 나은 선택이다. 사용할 웹 서버가 `lighttpd` 가 아니라면 아래와 같이 `--httpd` 옵션을 사용해야 한다.

```console
$ git instaweb --httpd=webrick
[2009-02-21 10:02:21] INFO  WEBrick 1.3.1
[2009-02-21 10:02:21] INFO  ruby 1.8.6 (2008-03-03) [universal-darwin9.0]
```

이렇게 하면 1234 포트로 HTTPD 서버를 시작하고 이 페이지를 여는 웹 브라우저를 자동으로 실행시킨다. 사용자에게는 꽤 편리하다. 필요한 일을 모두 마치고 나서 같은 명령어에 `--stop` 옵션을 추가하여 서버를 중지한다:

```console
$ git instaweb --httpd=webrick --stop
```

자신의 프로젝트에서 언제나 웹 인터페이스를 운영하려면 먼저 웹 서버에 이 CGI 스크립트를 설치해야 한다. 몇몇 리눅스 배포판에서는 `apt` 나 `dnf` 으로 설치할 수 있게 `gitweb` 패키지를 제공한다. 그러니 패키지로 설치할 수 있는지 확인해 보는 것이 좋다. 여기에서는 GitWeb을 수동으로 설치하는 방법을 간단히 살펴보자. 먼저 GitWeb이 포함된 Git 소스 코드를 구한 다음 아래의 CGI 스크립트를 빌드한다.

```console
$ git clone git://git.kernel.org/pub/scm/git/git.git
$ cd git/
$ make GITWEB_PROJECTROOT="/srv/git" prefix=/usr gitweb
    SUBDIR gitweb
    SUBDIR ../
make[2]: `GIT-VERSION-FILE' is up to date.
    GEN gitweb.cgi
    GEN static/gitweb.js
$ sudo cp -Rf gitweb /var/www/
```

빌드할 때 `GITWEB_PROJECTROOT` 변수로 Git 저장소의 위치를 알려준다. 이제 Apache가 이 스크립트를 사용하도록 VirtualHost 항목에 설정한다.

```console
<VirtualHost *:80>
    ServerName gitserver
    DocumentRoot /var/www/gitweb
    <Directory /var/www/gitweb>
        Options +ExecCGI +FollowSymLinks +SymLinksIfOwnerMatch
        AllowOverride All
        order allow,deny
        Allow from all
        AddHandler cgi-script cgi
        DirectoryIndex gitweb.cgi
    </Directory>
</VirtualHost>
```

다시 말해서 GitWeb은 CGI나 Perl을 지원하는 웹 서버라면 아무거나 사용할 수 있다. 이제 `http://gitserver/` 에 접속하여 온라인으로 저장소를 확인할 수 있다.



## 4.8 GitLab

간단하게 쓰기엔 GitWeb이 꽤 좋다. 그런데 좀 더 기능이 많은 Git 서버를 쓰려면 다른 서버를 찾아 설치해야 한다. GitLab은 널리 사용하는 서버 중 하나이다. 여기서 예제를 통해 설치하고 사용하는 것을 배워보자. GitLab은 기능이 많은 만큼 설정도 복잡하고 유지보수를 위해 해야 할 것도 많다.

### 설치

GitLab은 데이터베이스와 따로 연동해야하는 웹 애플리케이션이라 다른 Git 서버들보다 설치하기에 복잡하지만, 문서화가 잘 되어있으므로 이를 참고한다.

설치 방법은 여러 가지가 있다. 가상 머신 이미지나 원클릭 인스톨러를 내려받아 빨리 설치하고 환경에 맞게 후다닥 설정해서 사용할 수 있다. https://bitnami.com/stack/gitlab에서 내려받을 수 있다. Bitnami의 로그인 화면은 아래와 같다(alt-→ 를 눌러서 들어간다). 로그인 화면에 설치된 GitLab의 IP와 기본 사용자이름, 암호가 써있다.

![Bitnami GitLab 가상 머신의 로그인 화면](https://git-scm.com/book/en/v2/images/bitnami.png)

그림 50. Bitnami GitLab 가상 머신의 로그인 화면

더 많은 것이 알고 싶다면 GitLab 커뮤니티 에디션의 readme 파일을 읽어보면 된다. https://gitlab.com/gitlab-org/gitlab-ce/tree/master에서 내려받을 수 있다. Chef의 레시피나 Digital Ocean(역주 - 호스팅 서비스)의 가상 머신, RPM, DEB 패키지 등에 관한 설치 방법들이 있다. ``비공식적인'' 설명서도 있다. 흔치 않은 운영체제나 데이터베이스와의 연동하는 법, 스크립트로 완전히 수동으로 설치하는 법 등 많은 주제를 다룬다.

### 관리자

GitLab의 관리자 도구는 웹 페이지로 되어있다. 웹 브라우저로 GitLab이 설치된 곳의 주소에 들어가면 그냥 보인다. 그리고 관리자로 로그인하자. 기본 사용자이름은 `admin@local.host`, 암호는 `5iveL!fe` 이다(이건 로그인 후에 바꿀 수 있다). 로그인하고 나서 메뉴 오른쪽 위에 있는 ``Admin area'' 를 클릭한다.

![GitLab 메뉴의 ``Admin area'' 버튼](https://git-scm.com/book/en/v2/images/gitlab-menu.png)

그림 51. GitLab 메뉴의 ``Admin area'' 버튼

#### 사용자

GitLab의 사용자 계정은 한 사람당 하나씩 만든다. 사용자 계정의 내용은 복잡하지 않다. 로그인 데이터에 추가로 개인 정보가 들어있다. 각 사용자마다 **네임스페이스**가 있다. 네임스페이스는 프로젝트를 묶는 단위이다. **jane** 사용자가 **project**라는 프로젝트를 진행 중이라면 프로젝트의 URL은 http://server/jane/project가 될 것이다.

![GitLab 사용자의 관리 화면](https://git-scm.com/book/en/v2/images/gitlab-users.png)

그림 52. GitLab 사용자의 관리 화면

사용자를 삭제하는 방법은 두 가지다. 일시적으로 GitLab에 로그인하지 못하게 하는 ``정지(Blocking)'' 가 있다. 정지한 사용자 데이터와 네임스페이스 안의 프로젝트 데이터는 삭제되지 않고 그대로 남는다. 커밋의 이메일 주소에 대한 링크도 여전히 사용자 프로파일 페이지로 연결된다.

하지만 사용자를 ``삭제(Destroying)'' 하면 그 사용자와 관련된 모든 데이터가 삭제된다. 삭제한 사용자의 모든 프로젝트와 데이터가 삭제되고 해당 사용자가 소유한 그룹도 삭제된다. 영구히 삭제돼 되돌릴 수 없으므로 조심해야 한다.

#### 그룹

GitLab 그룹은 프로젝트와 누가 어떤 프로젝트에 어떻게 접근할지에 대한 권한 데이터의 모음이다. 그룹에도 사용자처럼 프로젝트 네임스페이스가 있다. +training+라는 그룹이 +materials+라는 프로젝트를 가지고 있으면 URL은 http://server/training/materials가 된다.

![GitLab의 그룹 관리 화면](https://git-scm.com/book/en/v2/images/gitlab-groups.png)

그림 53. GitLab의 그룹 관리 화면

그룹은 많은 사용자가 모인 곳이다. 그룹의 사용자의 권한은 그룹의 프로젝트에 대한 권한과 그룹 자체에 대한 권한이 따로 있다. 권한은 `Guest''(이슈 등록과 채팅만 할 수 있다.)부터 `Owner''(그룹과 멤버, 프로젝트에 대한 모든 제어가 가능하다.)까지 지정할 수 있다. 여기에서 어떤 권한이 있는지 나열하기엔 너무 많다. GitLab의 관리 화면에서 각 권한에 대한 링크를 참고하길 바란다.

#### 프로젝트

GitLab의 프로젝트는 간단히 이야기하면 하나의 Git 저장소다. 모든 프로젝트는 한 사용자나 한 그룹에 속하게 된다. 사용자에 딸린 프로젝트는 사용자가 관리자로서 그 프로젝트를 완전히 제어한다. 그룹에 딸린 프로젝트는 해당 그룹의 사용자 권한 레벨에 따라 다르다.

프로젝트마다 공개 수준을 지정할 수 있어서 사람마다 프로젝트 페이지와 저장소가 보이거나 안 보이게 할 수 있다. 프로젝트가 *Private* 이면 프로젝트 소유자가 허락한 사람들만 프로젝트에 접근할 수 있다. *Internal* 은 로그인한 사용자에게만 보인다. 그리고 *Public* 프로젝트는 모든 사람이 볼 수 있다. 이런 공개 수준은 `git fetch` 같은 접근이나 웹 UI 접근에 다 적용된다.

#### 훅

GitLab은 훅도 지원하는데 프로젝트 훅이나 시스템 훅을 사용할 수 있다. 훅은 어떤 이벤트가 발생하면 해당 이벤트 정보가 담긴 JSON 데이터를 HTTP POST로 보낸다. Git 저장소나 GitLab과 연동해서 CI나 채팅, 개발 도구 등으로 자동화하기에 좋다.

### 기본 사용법

먼저 새로운 프로젝트를 만들어보자. 툴바의 `+'' 아이콘을 클릭한다. 프로젝트의 이름, 프로젝트 네임스페이스, 공개 수준을 입력한다. 지금 입력한 것은 대부분 나중에 다시 바꿀 수 있다. `Create Project'' 를 클릭하면 끝난다.

프로젝트가 만들어졌으면 로컬 Git 저장소랑 연결하자. HTTPS나 SSH 프로토콜을 이용해 프로젝트를 Git 리모트로 등록한다. 저장소 URL은 프로젝트 홈페이지 위 쪽에 있다. 아래와 같이 명령어를 이용해 로컬 저장소에 `gitlab` 이라는 이름으로 리모트 저장소를 등록한다.

```console
$ git remote add gitlab https://server/namespace/project.git
```

로컬 저장소가 없으면 그냥 아래 명령어를 실행한다.

```console
$ git clone https://server/namespace/project.git
```

웹 UI는 꽤 유용하다. 저장소에 대한 각종 정보를 보여준다. 프로젝트 홈페이지에서는 최근 활동을 보여주고 제일 위의 링크를 클릭하면 프로젝트의 파일과 커밋 로그가 나온다.

### 함께 일하기

함께 일할 사람에게 그냥 Git 저장소의 Push 권한을 주는 걸로 간단하게 협업을 시작할 수 있다. 프로젝트 설정 페이지에서 `Members'' 섹션에 같이 일할 사용자를 추가한다. 그리고 그 사용자가 Push 할 수 있도록 설정한다(다른 접근 수준에 대해서는 그룹에서 볼 수 있다). `Developer'' 이상의 권한을 주면 그 사용자는 우리 저장소에 Push 하거나 브랜치를 만들 수 있다.

Merge 요청을 하도록 해서 통제권을 유지한 채로 협업하는 방법도 있다. 프로젝트에 접근할 수 있는 모든 사용자가 프로젝트에 기여할 수 있다. 사용자는 마음껏 브랜치를 만들고 커밋, Push 하고 나서 이 브랜치를 `master` 나 다른 브랜치에 Merge 해달라고 요청한다. Push 권한이 없는 사용자는 저장소를 `fork'' 한 다음에 `fork'' 한 *자신의 저장소* 에 Push 한다. 그리고는 원래 저장소에 내 저장소에 있는 브랜치를 Merge 해달라고 요청하면 된다. 소유자는 이걸로 자신의 저장소에 대한 모든 통제 권한을 가진다. 어떤 데이터가 들어올 수 있는지 언제 들어오는지 소유자가 결정할 수 있다.

Merge 요청과 이슈는 대화의 기본 단위이다. 각 Merge 요청에서는 일반적인 토론뿐만 아니라 라인 단위로까지 대화가 이루어진다. 물론 코드 리뷰가 간단히 끝날 수도 있다. 요청과 이슈는 모두 사용자에게 할당되거나 마일스톤의 과제로 편입된다.

이 섹션에서는 GitLab의 Git과 맞닿은 부분만 설명했지만 이게 전부가 아니다. GitLab은 굉장히 성숙했다. 이 외에도 프로젝트 위키나 토론용 ``walls'', 시스템 관리 도구 등 협업용 기능이 많다. GitLab의 장점은 일단 서버가 돌아가면 SSH로 서버에 접속할 일이 별로 없다는 것이다. 대부분 관리는 웹 브라우저로 가능하다.



## 4.9 또 다른 선택지, 호스팅

Git 서버를 직접 운영하기가 부담스러울 수 있다. 그런 사람들을 위해 Git 호스팅 서비스가 몇 가지 있다. 외부 Git 호스팅 서비스를 사용하면 좋은 점이 있다. 설정이 쉽고 서버 관리 비용을 아낄 수 있다. 내부적으로 Git 서버를 운영하더라도 소스 코드를 공개하기 위해 호스팅 서비스를 이용해야 할 수 있다. 이런 식으로 손쉽게 오픈 소스 커뮤니티를 구성한다.

현재 많은 호스팅 서비스가 있다. 각각 장단점이 있으므로 잘 선택해서 사용하면 된다. https://git.wiki.kernel.org/index.php/GitHosting에 최신 Git 호스팅 서비스 리스트가 있으니 참고하자.

GitHub에 대해서는 [GitHub](https://git-scm.com/book/ko/v2/ch00/ch06-github)에서 자세히 설명하려 한다. GitHub은 가장 큰 Git 호스팅 서비스이다. Git 서버를 직접 운영하지 않으려면 수십 개의 호스팅 서비스 중에서 하나를 골라야 한다.



## 4.10 요약

Git 서버를 운영하거나 사람들과 협업을 하는 방법 몇 가지를 살펴 보았다.

자신의 서버에서 Git 서버를 운영하면 제어 범위가 넓어지고 방화벽 등을 운영할 수 있다. 하지만 설정하고 유지보수하는 데에 시간이 많이 든다. **호스팅 서비스를 이용하면 설정과 유지보수가 쉬워진다. 대신 코드를 외부에 두게 된다. 자신의 회사나 조직에서 이를 허용하는지 사용하기 전에 확인**해야 한다.

필요에 따라 둘 중 하나를 선택하든지, 아니면 두 방법을 적절히 섞어서 사용하는 것이 좋다.