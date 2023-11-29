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

# Chiselled Container란?
Distroless 컨셉의 컨테이너로, 우분투 기반 환경에서 애플리케이션 실행에 필요한 것만 남겨두고 다른 불필요한 패키지나 구성요소는 모두 제거한 형태의 컨테이너 입니다. 예를 들어 .Net 앱을 Chiselled Container 로 컨테이너화 한다면, .Net 런타임과 빌드해서 나온 애플리케이션 어셈블리 파일과 애플리케이션이 요구하는 라이브러리를 제외한 모든 것을 제거한 컨테이너라고 보면 되겠습니다. 제거 되는 것에는 앱 실행에 굳이 필요하지 않은 `bash`, `cd`, `ls` 같은 명령도 제거가 되고. 컨테이너 이미지 완성 후에는 더 사용하지 않는 패키지 관리자(`apt`, `dpkg`)등도 모두 제거 대상에 포함이 됩니다. 컨테이너에 포함된 패키지라 해도 패키지에서 앱 실행에 굳이 필요하지 않은 부분(매뉴얼, 설정 스크립트, 헤더 파일 등)도 모두 제거가 됩니다. 이렇게 앱 실행에 필요한 것만 남기고, 다른 불필요한 것들은 모두 정리가 되기 때문에, 기존의 일반적인 우분투 혹은 다른 배포판 이미지 기반으로 만들 때에 비하여 훨씬 작은 크기의 컨테이너 이미지를 만들 수 있습니다.

이렇게 불필요한 것을 제거하는 작업을 직접 한다면 아마 꽤나 복잡하고 수고스러운 과정이 될텐데요. Chiselled Container의 경우, [Chisel](https://github.com/canonical/chisel) 이라는 CLI 도구를 활용해서 경량화된 컨테이너 이미지를 만들 때, 포함 시키고자 하는 데비안 패키지의 필요한 부분만 잘라다가 넣을 수 있습니다. Chisel CLI은 컨테이너에 패키지를 추가할 때, 사전에 각 데비안 패키지를 대상으로 만들어진 Slice 정의 파일을 참고하여 설치 작업을 하는데, 각각의 Slice 정의 파일에는 패키지의 어떤 파일을 설치할 지 혹은 설치하지 않을지에 대한 사항이 정의되어 있습니다. 

예를 들어, `chisel` 명령으로 Ubuntu 22.04의 [`ca-certificates`](https://packages.ubuntu.com/jammy/ca-certificates)패키지를 `myrootfs/` 경로의 루트 파일시스템에 설치 한다고 가정하면, 아래와 같은 명령을 실행하게 됩니다.
```bash
chisel cut --release ubuntu-22.04 --root myrootfs/ ca-certificates
```

그러면 Chisel CLI에서 [`chisel-release`](https://github.com/canonical/chisel-releases) 아카이브에 있는 [`ca-certificates.yaml`](https://github.com/canonical/chisel-releases/blob/ubuntu-22.04/slices/ca-certificates.yaml)를 참고하여 해당 파일의 `contents` 속성 아래에 정의된 경로에 해당하는 파일만 설치를 하게 됩니다. 해당 `ca-certificates.yaml` 파일 내용은 아래와 같이 생겼는데, [원래의 데비안 패키지에 포함된 파일 목록](https://packages.ubuntu.com/jammy/all/ca-certificates/filelist)과 비교하면. 인증서 파일 이외 불필요한 문서 파일 등은 모두 제외 되어 있음을 알 수 있습니다.

```yaml
package: ca-certificates

slices:
  data:
    contents:
      /etc/ssl/certs/ca-certificates.crt: {text: FIXME, mutable: true}
      /usr/share/ca-certificates/mozilla/: {until: mutate}
      /usr/share/ca-certificates/mozilla/*: {until: mutate}
    mutate: |
      certs_dir = "/usr/share/ca-certificates/mozilla/"
      certs = [content.read(certs_dir + path) for path in content.list(certs_dir)]
      content.write("/etc/ssl/certs/ca-certificates.crt", "".join(certs))
```
이렇게 Chisel CLI로 패키지 추가 작업을 진행하면, 결론적으로 아래 그림처럼 원하는 데비안 패키지에서 딱 필요한 부분만 "끌로 깎아(Chisel)" 내어다가 컨테이너 이미지를 만드는데 사용하는 형태가 되게 됩니다.

![Slice of Ubuntu](./slice-of-ubuntu.png)
> 그림 출처: https://github.com/canonical/chisel/blob/main/docs/_static/slice-of-ubuntu.png