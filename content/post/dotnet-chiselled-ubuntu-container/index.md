---
title: Chiselled Ubuntu Container를 활용한 .Net앱 컨테이너화 
authors:
- youngbin-han # 저자 프로필 페이지 경로 입력
date: 2023-11-29T18:00:00+09:00
categories:
- Hands On # 새로운 기술 소개 또는 Hands On 이외 기술 관련 글
tags:
- Ubuntu
- Linux
- Container
- DotNet
feature_image: 'opening.jpg' # 포스트 커버 이미지 경로 (포스트에 포함된 이미지 중 하나 지정. 필드 제거하면 기본 이미지가 나옵니다.)
---

오늘날 많은 조직에서 웹 애플리케이션을 컨테이너 이미지로 만들어 Kubernetes와 같은 컨테이너를 호스팅 할 수 있는 환경에 배포하고 있는데요. 이러한 환경에서 탄력적인 배포를 위해 컨테이너 이미지를 최대한 경량화 하여 만들고 있기도 합니다. [다만 이 글을 보시면 아시겠지만, 컨테이너 이미지 경량화를 처음 해 본다면 쉽지많은 않을 것이고. 고려할 사할도 어느정도 있습니다.](/2022/11/08/container-imagesize-diet/) 최근에는 이를 위한 다양한 솔루션이나 베이스 컨테이너 이미지 또한 많이 나오고 있습니다. 특히 요즘은 Alpine Linux와 같이 처음부터 경량 컨테이너를 위한 OS나 Google의 Distroless처럼 경량화된 컨테이너 이미지 이면서도 프로그래밍 언어 툴체인이나 런타임을 넣어 두어서 쉽게 사용 가능한 컨테이너 이미지도 있습니다.

리눅스 배포판중 많이 쓰이는 Ubuntu도 최근 경량 컨테이너 이미지를 출시 했는데요, 바로 Chiselled Ubuntu Container 이미지 입니다. Ubuntu를 개발한 Canonical과 Microsoft과 최근 여러 부분에서 협업을 많이 하다 보니, .Net을 위한 Chiselled Container 이미지도 함께 출시 되었습니다. 때마침 서비스 개발팀의 많은 서비스 백엔드가 .Net 기반이기도 하여서 실제 서비스에 적용 하기도 하였는데요. 이번 포스팅을 통해 Chiselled Container 에 대해 알아보고, 다른 경량 컨테이너 이미지와 무엇이 다른지 그리고 서비스 개발팀에서는 어떻게 적용 했는지에 대해 다뤄 보고자 합니다.