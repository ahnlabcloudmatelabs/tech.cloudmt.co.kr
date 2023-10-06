---
feature_image: images/cover.png
authors:
- jeonghwan kim
date: "2023-08-31T00:00:00Z"
categories:
- hands on

tags:
- Docker
- Container Image
- dockerfile
title: 컨테이너 작성시 환경변수 활용하기 1/2
---

# 문서 작성의 이유

이전 글은 [컨테이너 이미지 용량 줄이기](https://tech.cloudmt.co.kr/2022/11/08/container-imagesize-diet/) 였고, 이번에는 컨테이너를 작성할 때 환경변수가 왜 중요한지 전달하고자 한다. 다만 분량 때문에 2개로 나누어서 전달 하고자 한다.

	⁃	개발 , 테스트 , 운용 환경 모두 각각의 컨테이너가 생성되어야 하는지?
	⁃	컨테이너 기본 실행환경은 어떤 환경으로 실행되어야 하는지?
	⁃	컨테이너 실행 환경은 어떻게 설정하는가?

이 글은 위 질문에 대한 답을 다음과 같이 정의 하고 설명한다. 더 좋은 방법이 있을 수 있지만 질문에 대한 접근 방법으로 나쁘지 않다고 생각한다.

	⁃	모두 하나의 컨테이너를 활용하여 운영해야 한다.
	⁃	컨테이너 기본 실행환경은 개발환경을 기본으로 실행되어야 한다.
	⁃	여러 환경 변수 설정법을 알아본다.

# 문제상황 파악

## 가장 간단한 컨테이너 작성

내용에 대한 설명을 하기전에 우선은 가장 간단한 컨테이너를 작성해보자. 그냥 단순히 "나는 컨테이너고, 실행환경은 무엇이다." 라는 내용을 출력하는 bash 스크립트를 만들어 보았다. 개발자가 아니여도 쉽게 이해할 수 있도록 일부러 최대한 간결하게 확인할 수 있는 방법을 택하였다.

```
$ cat run.sh
#!/bin/sh

while [ true ]; do
  echo “I am container. env: test”
  sleep 3
done

$ cat Dockerfile
FROM alpine

COPY run.sh /root/run.sh
RUN chmod +x /root/run.sh

CMD /root/run.sh

$ docker build -t blog .

$ docker run -it --rm blog
I am container. env: test
I am container. env: test
```

## 환경별 컨테이너의 필요성?

위의 출력 화면의 변경을 위해 (= 실제 어플리케이션의 동작 차이점) `개발, 테스트, 운용` 환경 모두 다른 컨테이너를 만들었다고 가정해보자. 환경별 컨테이너를 만들기 위해 소스코드 변경이 3번 필요하고 컨테이너 작성도 3번이 필요했을 것이다.

컨테이너 작성 과정에서 발생하는 문제에 대해서는 열심히 글을 적다가 모두 지우고 결론만 적어둔다. (참고로 4시간 넘게 작성했다가 너무 설득을 위한 설명이 들어가는 듯 싶어 삭제하였다.)

	⁃	각 환경별 코드 변경 이후 컨테이너를 만든다면 각 컨테이너별 동작을 별도로 확인하는 것이 맞다.
	⁃	컨테이너 생성의 시기가 달라진다면 내부 내용이 달라질 수 있는 경우도 있다.
	⁃	각 환경별 별도의 소스 변경을 작업한다면 소스 변경 중 사람의 실수가 발생할 수 있는 부분이 있다.

하여 코드를 다음과 같이 변경하여 코드를 3번 변경하고 컨테이너를 3번 만드는 것이 아니라 아래처럼 수정을 한다면 하나의 컨테이너로 모든 환경을 실행 할 수 있게 된다.

## 단일 컨테이너 작성

```
$ cat run.sh
#!/bin/sh

while [ true ]; do
  echo "I am container. env: ${ENV}"
  sleep 3
done

$ docker build -t blog .

$ docker run -it --rm -e ENV=test blog
I am container. env: test
I am container. env: test

$ docker run -it --rm -e ENV=stage blog
I am container. env: stage
I am container. env: stage

$ docker run -it --rm -e ENV=prod blog
I am container. env: prod
I am container. env: prod
```

이와 같이 우리는 하나의 컨테이너로 충분히 3가지 환경 모두 적용하여 사용할 수 있음을 확인하였다. 여기서는 너무 간단한 예시만 들었다고 생각할 수 있다.

	⁃	여러개의 변수가 필요한 경우는 불편할 것이다.
	⁃	예시로 활용한 어플리케이션은 너무 간단하다.

이것은 어디까지나 빠르고 쉬운 이해를 위해서 아주 간단한 예시를 들었을 뿐 실제 동작에 필요한 환경변수를 정의 하는 것은 여러가지 방법이 있을 수 있다. 그것에 대한 방법은 어플리케이션 작성시 고민이 필요한 부분이라고 생각한다.

다만 여기서 이야기 하고 싶은 부분은 어플리케이션 동작에 반드시 필요한 다음의 내용들은 환경변수로 정의 하는 것이 필요하다는 것이다. (모두 기술할 수 없을 것이고 어디까지나 예시이다.)

	⁃	Database 접속을 위한 환경 설정 내역
	⁃	공용 폴더 (이미지, 알고리즘 모델 등...) 접속 경로
	⁃	각종 서비스 요청 주소
	⁃	로그 파일 저장 위치 혹은 방법

# 환경변수 활용

그럼 이제 환경변수를 전달 할 수 있는 방법을 정리해 보자. 대략 다음과 같은 방식을 사용하게 된다. 모두 이번에 다룰 수는 없고 내용을 나눠서 설명하겠다.
	⁃	docker 실행 명령으로 전달하기
	⁃	docker file 에 명시하기
	⁃	docker compose 에 직접 입력하기
	⁃	docker compose 에서 환경 변수로 반영하기
	⁃	여러 .env 파일을 나눠서 사용하기

## docker 실행 명령으로 전달

첫번째인 docker 실행 명령으로 전달하기는 이미 위에서 사용방법을 보여주었다. 공식 문서는 다음의 내용을 참고하자. [docker run](https://docs.docker.com/engine/reference/commandline/run/#env)

## 컨테이너 실행 기본 값이 없을 때

그렇다면 만약 실수로 환경 변수를 지정하지 않고 실행한다면 어떻게 될 것인가? 환경변수가 없이 실행된다면 어플리케이션에 따라서는 바로 비정상 종료되는 경우가 많을 것이다. (오히려 비정상 종료가 되는 것이 더 안전하다. 어플리케이션 실행 도중 종료되는 것이 더 문제를 찾는데 어려움이 생길 수 있다.)

```
$ docker run -it --rm blog
I am container. env:
I am container. env:
```

## 컨테이너 실행 기본 값 지정

그렇다면 실행시 환경변수 설정을 하지 않더라도 실행 변수를 지정할 수 있는 방법은 없을까? 그 부분은 바로 Dockerfile 을 작성할 때 부터 기본값을 설정하는 것이다. [environment-replacement](https://docs.docker.com/engine/reference/builder/#environment-replacement)

```
$ cat Dockerfile
FROM alpine
ENV ENV=test

COPY run.sh /root/run.sh
RUN chmod +x /root/run.sh

CMD /root/run.sh

$ docker run -it --rm blog
I am container. env: test
I am container. env: test
```

이렇게 Dockerfile 을 작성할 때부터 반드시 필요한 환경 변수는 지정을 하여 Dockerfile 을 작성한다면 실행을 위한 기본값을 매번 지정하지 않아도 된다.

# 정리

우리는 왜 컨테이너 작성시 환경변수가 필요하고, 환경변수를 활용하였을 때 하나의 컨테이너로 운용할 수 있는지 확인해보았다. 아래의 내용은 곧 다음글에서 다루어보겠다.

	⁃	컨테이너 기본 실행환경은 반드시 어떤 환경으로 실행되어야 하는지?
	⁃	컨테이너 실행 환경은 전달 방법은 어떤 것들이 있는지? (남은 것들)

