---
title: 실무에서 개발자가 알아야 하는 도커/쿠버네티스 범위(기본)
authors:
  - juunini
date: 2024-01-16T09:20:11+09:00
categories:
  - Tech
tags:
  - 도커
  - docker
  - 쿠버네티스
  - kubernetes
  - k8s
  - dockerfile
  - developers
feature_image: "images/cover.webp"
---

개발자의 본업은 개발이지만, 현실에선 인프라 관리로 인해 많은 고통을 받습니다.  

개발에 집중할 수 있도록 도커/쿠버네티스 사용에 필요한 기본 지식을 알려드리겠습니다.

# Docker

도커는 이것만 알면 됩니다.

- docker run -dit \\  
    --name \<컨테이너 이름\> \\  
    -p \<host port\>:\<container port\> \\  
    -e \<env_key=env_value\> \\  
    \<이미지 이름\> \\  
    \<CMD\(optional\)\>
- docker ps \(-a\)
- docker start \<컨테이너 이름 또는 ID\>
- docker exec -it \<컨테이너 이름 또는 ID\> \<CMD\>
- docker stop \<컨테이너 이름 또는 ID\>
- docker rm \(-f\) \<컨테이너 이름 또는 ID\>
- docker logs \<컨테이너 이름 또는 ID\>
- docker pull \<이미지 이름\>
- docker images
- docker rmi \<이미지 이름\>
- docker build .
- docker tag \<이미지 이름\> \<태그\>
- docker push \<이미지 이름 또는 태그\>

더 알면 좋기야 하지만  
이것만 알아도 됩니다.

# Kubernetes

컨테이너로 개발을 한다고 해서 쿠버네티스를 꼭 알아야 하는 것은 아닙니다.  
쿠버네티스 보다 우선해야 하는 것은 Dockerfile 작성법입니다.

# Dockerfile

개발자에게 필요한 Dockerfile의 최우선 사항은 다음 다섯 가지입니다.  

- FROM \<이미지 이름\>
- RUN \<쉘 명령어\>
- ADD \<host path\>:\<container path\>
- CMD \<쉘 명령어\>

그리고 **alpine**

아래는 NodeJS의 컨테이너 이미지에서  
bullseye(debian)와 alpine의 용량 비교입니다.  

![](./images/image1.webp)

| Container Image | Size |
| -: | :- |
| Debian bullseye | 52MB |
| Alpine 3.19 | 3MB |

Dockerfile을 작성할 때  
공식 문서를 참고해도 좋지만, 아래 링크를 참고해도 좋습니다.

https://github.com/docker/awesome-compose

위 링크는 docker compose에 대한 awesome 문서이지만  
작성하신 애플리케이션의 언어나 프레임워크를 검색하면 대부분 나오고  
눌러서 들어가보면 docker compose만 있는게 아니라 Dockerfile도 포함되어 있습니다.  

# 왜?

Dockerfile 작성법을 익혀서  
애플리케이션을 컨테이너로 만들 수 있으면  
어느 클라우드를 사용하든 컨테이너화 된 애플리케이션으로  
배포가 가능합니다.  
(예: lambda같은 서버리스나 ECS 같은 서비스)

개발자는 여기까지만 할 줄 알아도 굉장히 훌륭합니다.  
그리고 k8s로 넘어가도 Dockerfile이 아주 중요합니다.  
(근데 여기까지도 몰라도 잘 먹고 살 수 있...)

> k8s는 필요하지 않은 상황에선 비용이 굉장히 비쌉니다.
> 오케스트레이션이 필요한 상황이 아니라면
> 기존 방식대로 서버를 배포하시는게 더 좋습니다.

# 왜 여기까지만 알아도 되는가?

기존의 아키텍쳐가 [SRE(사이트 신뢰성 엔지니어링)](https://www.redhat.com/ko/topics/devops/what-is-sre)를 보장할 수 없는 상태가 된다면  
개발자는 기능 개발, 버그 수정만으로도 시간이 부족해 지속적인 고통을 받게 됩니다.

많은 분들이 인프라 비용을 줄이기 위해 노력하지만,  
그로 인해 개발자가 개발에 집중하지 못하고 소모하는 시간은 생각하지 못하고 있습니다.  
개발자들이 개발에만 집중할 수 있는 환경이 주어졌을 때와 아닌 경우를 비교해보면  
전자의 경우가 후자의 경우보다 효율적으로 비용을 쓰는 경우가 일반적입니다.

개발자가 개발에 집중하지 못하고  
쿠버네티스 셋팅 때문에 '워크로드 관리는 어떻게 할 것인가?',   
'CSP에서 제공하는 PaaS 쿠버네티스를 사용할 것인가? 혹은 오픈소스를 활용할 것인가?'  
등으로 고민하는 것 또한 엄청난 비용입니다.

많은 회사들이 인력 부족으로 늘 시간에 쫓겨 고통을 받습니다.  
위와 같은 엔지니어링은 순식간에 가능한 것이 아니라, 담당자를 지정하고 지속적으로 관리(구축, 운영, 개선)해야 합니다.  
 
개발자들이 인프라 비용 50만원, 100만원을 줄여보고자 고통을 받지만,  
그로 인해 개발자가 한 달 중 절반 혹은 그 이상을 인프라 관리하느라 개발하지 못하는 인건비가 그를 상회하는 것은 간과합니다.  
개발자들이 개발에만 집중할 수 있는 환경이 가장 비용 효율적입니다.
 
담당자(플랫폼 엔지니어링)가 없거나 부족하여 힘드시다면 클라우드메이트에 연락주세요.
