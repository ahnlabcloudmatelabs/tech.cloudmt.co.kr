---
title: 도커와 컨테이너의 이해 (1/3) - 컨테이너 사용법
authors:
  - dohyeon-lee
date: 2022-06-29T11:29:56.542Z
feature_image: 프레젠테이션1.jpg
categories:
  - Hands On
tags:
  - docker
---
### 목차

1. 도커 이미지? 도커 컨테이너?

   1. 도커 이미지
   2. 도커 컨테이너
2. 기본적인 도커 명령어

   1. 도커 컨테이너

- - -

본 포스트는 한국 HPE 교육 센터에서 진행한 컨테이너 기반 애플리케이션의 인프라스트럭처 아키텍처 구축 실무 과정을 수강 후 사내 공유를 위해 쓰인 글입니다.

안녕하세요. 첫 번째 글을 작성하게 된 클라우드메이트 TE팀 이도현입니다.

점점 더 많은 기업에서 ECS, EKS 등 다양한 컨테이너 서비스를 검토 및 도입하는 추세입니다. 컨테이너 서비스 관련된 업무를 마주할 때마다 기반 지식이 단단하지 않아 관련 교육이 필요했었습니다. 도커 이미지와 도커 컨테이너의 개념을 간단하게 설명하고, 기본적인 도커 명령어에 대해 포스팅하겠습니다.

→ 컨테이너 개념 설명이 포함되어 있는 글도 보시면 좋을 것 같습니다.\
<https://cloudmt.co.kr/?p=3927>



# 도커 이미지? 도커 컨테이너?

## 도커 이미지(Docker Image)

도커 이미지는 **컨테이너를 만드는 데 사용**되는 **읽기 전용(Read-only)** 템플릿입니다.

컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 도커파일을 만든 후 **Dockerfile을 빌드** 하여 이미지를 만듭니다.



## 도커 컨테이너(Docker Container)

도커 **이미지를 실행한 상태**입니다. 이미지로 컨테이너를 생성하면 이미지의 목적에 맞는 파일이 들어있는 파일 시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성됩니다. 이것을 도커 컨테이너라고 합니다. 도커 컨테이너는 읽기 전용인 이미지에 변경된 사항을 저장하는 컨테이너 계층(Layer)에 저장합니다.



![](image_container.png)



도커 이미지를 도넛 레시피에 비유한다면, 도커 컨테이너는 해당 레시피를 이용해 만든 도넛으로 비유할 수 있습니다. \
하나의 도넛 레시피에서 여러 가지 맛의 도넛을 만들 수 있는 것과 같이, 하나의 도커 이미지로 여러 개의 도커 컨테이너를 만들 수 있습니다. \
또한, 기존의 도넛 레시피를 수정하게 되어도, 이미 기존 레시피로 만들어진 도넛에는 영향이 없듯이, \
이처럼 도커 이미지를 변경해도 이미 실행 중인 도커 컨테이너에는 영향을 주지 않습니다.



# 기본적인 도커 명령어

도커 명령어의 구조는 아래와 같습니다.



* **docker \[대상] \[액션]**

  → \[대상] : `container`(생략 가능), `image`, `volume`, `network` 등

  → \[액션] : `ls`, `inspect`, `start`, `run` 등



## 도커 컨테이너

![](dockercontainerstatus.png)



* **docker (container) create**

  → 컨테이너를 생성하고 자동으로 시작하지는 않음

```shell
root@DH:~# docker create --name testos centos
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
875366cc4662d6ccdc21dfbaa654ed3eee74bb54d6a2ce34333a62924f7e0272
```

centos 이미지를 사용해서 컨테이너를 생성해 주는 명령어입니다.\
**`--name`** 옵션을 추가하여 컨테이너명은 testos로 설정하였습니다.\
    → name 옵션을 쓰지 않으면 임의의 name이 부여됩니다.

이미 생성된 컨테이너의 컨테이너 명을 바꾸고 싶으면 `docker rename [현재 이름] [바꿀 이름]` 명령어를 사용합니다. \
컨테이너 명을 변경하여도 컨테이너 ID는 변경되지 않습니다.



로컬 리포지토리에 이미지가 없으면 기본으로 docker hub에서 이미지를 pull 합니다. 한번 pull 한 이미지는 재사용이 가능합니다.



컨테이너를 생성할 때 옵션을 써줄 수도 있습니다.

```shell
docker create -it --name testos2 centos
```

해당 옵션들의 설명은 아래와 같습니다.



* **docker ps**

  → 실행(Up) 중인 컨테이너들의 목록을 확인

  → `docker container ls`와 같음

```shell
root@DH:~# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
adb6732a399d   centos    "/bin/bash"   51 seconds ago   Up 50 seconds             testos2
```



출력된 결과의 각 항목은 다음과 같이 설명할 수 있습니다.

> **\[CONTAINER ID] :**
>
> * 컨테이너에 할당되는 고유한 컨테이너 ID
> * 전체 ID에서 12자리만 출력
>
> **\[IMAGE] :**
>
> * 컨테이너를 생성할 때 사용된 이미지
>
> **\[COMMAND] :**
>
> * 컨테이너가 시작될 때 실행될 명령어
> * `docker run` 이나 `docker create` 명령어의 맨 끝에 새로운 명령어를 입력해서 컨테이너를 생성할 때 대체 가능
>
> **\[CREATED] :**
>
> * 컨테이너 생성 후 경과 시간
>
> **\[STATUS] :**
>
> * 컨테이너의 상태
> * 실행 중(Up), 종료(Exited), 일시 중지(Pause)
>
> **\[PORTS] :**
>
> * 컨테이너가 개방한 포트와 호스트에 연결된 포트
>
> **\[NAMES] :**
>
> * 컨테이너 이름



`-a` (all) 옵션을 함께 써주면 실행 중이지 않은 컨테이너를 포함하여 전체 컨테이너 목록을 출력합니다.

```shell
root@DH:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
adb6732a399d   centos    "/bin/bash"   55 seconds ago   Up 54 seconds             testos2
875366cc4662   centos    "/bin/bash"   5 hours ago      Created                   testos
```



`--no-trunc` 옵션을 함께 써주면 컨테이너 ID 전체를 보여줍니다.

```shell
root@DH:~# docker ps --no-trunc
CONTAINER ID                                                       IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
adb6732a399de1d9d4d2d8e2b74d6a4c6829652c8e950c77daebe32afdc25430   centos    "/bin/bash"   5 minutes ago   Up 5 minutes             testos2
```



* **docker start**

  → 컨테이너를 시작(실행)

  → ***생성해둔 컨테이너***를 시작할 수 있음

```shell
root@DH:~# docker start -ai testos
[root@151f3b70b5a4 /]#
```

컨테이너(testos)를 시작하면서 `-ai` 옵션을 사용해 해당 컨테이너 내부로 접근하여 표준 입력을 받을 수 있도록 하였습니다.

해당 컨테이너에 접근한 상태로 exit 명령을 사용하면, `/bin/bash`가 종료되면서 컨테이너도 함께 종료됩니다.

때문에 순차적으로 `Ctrl + P`, `Ctrl + Q`를 눌러 컨테이너 실행 상태를 유지한 채로 빠져나온 후, 컨테이너가 실행 중(Up)인지 확인할 수 있습니다.

```shell
root@DH:~# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
151f3b70b5a4   centos    "/bin/bash"   44 seconds ago   Up 16 seconds             testos
```



* **docker stop**

  → 실행 중인 컨테이너를 종료

```shell
root@DH:~# docker stop testos
testos
```

컨테이너가 종료(Exited)된 것을 확인할 수 있습니다.

```shell
root@DH:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
151f3b70b5a4   centos    "/bin/bash"   8 minutes ago   Exited (0) 19 seconds ago             testos
```



* **docker run**

  → 컨테이너를 시작하고 COMMAND를 실행

  → 로컬에 이미지가 있다면 해당 이미지로 실행하고, 없으면 도커허브에서 다운로드 후 실행

  → ***create + start***

```shell
root@DH:~# docker run -dit --name test centos
5d56fc765e3780fb06f5f3d5a66935e1a087087d7b9ab69e979e830773603a81

root@DH:~# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
5d56fc765e37   centos    "/bin/bash"   1 second ago   Up 1 second             test
```

`-d` 옵션을 사용해 사용자가 직접 컨테이너 안으로 접근하지 않고, 컨테이너의 COMMAND를 백그라운드로 실행할 수 있습니다.



컨테이너를 시작할 때, 명령어의 맨 뒤에 임의로 COMMAND를 정의할 수 있습니다.

```shell
root@DH:~# docker run -it --name date centos /bin/date
Tue Jun 28 08:56:41 UTC 2022

root@DH:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                          PORTS     NAMES
24a654120847   centos    "/bin/date"   About a minute ago   Exited (0) About a minute ago             date
```