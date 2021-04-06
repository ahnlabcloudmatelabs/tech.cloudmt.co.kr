---
title: COM 라이브러리를 참조하는 .Net 앱의 컨테이너화
authors:
- youngbin-han # 저자 프로필 페이지 경로 입력
date: 2021-04-06T09:51:53+09:00
categories:
# 아래 4가지 카테고리 중 하나만 선택하여 남겨두고, 나머지는 지우고 글을 계속 작성하세요.
# - Hands On # 실습 위주의 글
- Tech # 새로운 기술 소개 또는 Hands On 이외 기술 관련 글
# - Column # 엔지니어가 아니여도 쉽게 읽을 수 있는 글
# - Mate Story # 클라우드메이트 소식, 일하는 방식 소개 등
tags:
- Windows Container
- Container
- .Net
- MSBuild
- COM Library
feature_image: 'images/cover.png' # 포스트 커버 이미지 경로 (포스트에 포함된 이미지 중 하나 지정. 필드 제거하면 기본 이미지가 나옵니다.)
ShowToc: false # 글 개요 보여줄지 여부
TocOpen: false # 글 개요를 보여주는 경우, 펼처서 보여줄지 여부.
draft: false # 초안 작성 모드. true 설정시 커밋해도 나오지 않습니다.
---

최근 통합 클라우드 빌링 서비스인 mateBilling 에서 사용하는 SOAP API 기반 카드결제 API 서버를 RESTful API 서버로 다시 개발하는 작업을 하고 있습니다.
그러면서 컨테이너화 하는 작업도 같이 진행했는데, Windows Container 를 이용해서 컨테이너화를 시도했습니다. 그 경험에 대해서 글로 정리해 공유해 보고자 합니다.

# 고려사항
결제 API 앱은 .Net(C#) 으로 작성되어 있고, API 호출이 들어오면 PG사에서 제공한 결제 클라이언트 라이브러리를 참조하여 호출하고, 그 결과를 다시 전달해 주는 방식으로 구성되어 있습니다. 
PG사에서 제공한 결제 클라이언트 라이브러리가 `*.dll` 확장자의 COM 라이브러리 형식으로 되어있는데, 때문에 아래와 같은 사항을 고려해야 했습니다.

- 빌드 및 빌드된 앱 실행 전 `regsvr32.exe` 명령으로 COM 라이브러리를 시스템에 등록해야 합니다. (컨테이너화 하는 경우 Windows Container 에서만 가능)
- .Net CLI 는 COM 라이브러리 참조가 있는 앱을 빌드하지 못하기 때문에, MSBuild 를 사용하여 빌드해야 합니다.

## 컨테이너에서의 MSBuild 사용
먼저 .Net CLI 로 빌드하지 못하는 점이 고려할 점 입니다. COM 라이브러리를 참조하는 프로젝트를 아래와 같이 .Net CLI 의 `dotnet build` 로 빌드를 실행하면 아래와 같은 오류가 발생합니다.

```powershell
❯ dotnet build
.NET용 Microsoft (R) Build Engine 버전 16.9.0+57a23d249
Copyright (C) Microsoft Corporation. All rights reserved.

  복원할 프로젝트를 확인하는 중...
  복원할 모든 프로젝트가 최신 상태입니다.
C:\Program Files\dotnet\sdk\5.0.201\Microsoft.Common.CurrentVersion.targets(2805,5): error : MSB4803: "ResolveComReference" 작업은 MSBuild의 .NET Core 버전에서 지원되지 않습니다. MSBuild의 .NET Framework 버전을 사용하세요.  자세한 내용은 https://aka.ms/msbuild/MSB4803을 참조하세요. [C:\Users\YoungbinHan\Documents\WorkProjects\MatePayApiService\MatePayApiService\MatePayApiService.csproj]

빌드하지 못했습니다.

C:\Program Files\dotnet\sdk\5.0.201\Microsoft.Common.CurrentVersion.targets(2805,5): error : MSB4803: "ResolveComReference" 작업은 MSBuild의 .NET Core 버전에서 지원되지 않습니다. MSBuild의 .NET Framework 버전을 사용하세요.  자세한 내용은 https://aka.ms/msbuild/MSB4803을 참조하세요. [C:\Users\YoungbinHan\Documents\WorkProjects\MatePayApiService\MatePayApiService\MatePayApiService.csproj]
    경고 0개
    오류 1개

경과 시간: 00:00:02.28
```
> MSB4803: "ResolveComReference" 작업은 MSBuild의 .NET Core 버전에서 지원되지 않습니다. MSBuild의 .NET Framework 버전을 사용하세요.  자세한 내용은 https://aka.ms/msbuild/MSB4803을 참조하세요.

때문에 GUI 환경에서 수동으로 빌드 하는 경우, 간단히 해당 프로젝트를 Visual Studio (Visual Studio Code 가 아닙니다.)에서 열어 빌드를 진행하면 됩니다.
![](images/vsbuild1.png)

Dockerfile을 작성해서 컨테이너화 할 때도 같은 방식으로 해야 할 텐데, 컨테이너 이미지 빌드는 CLI 환경에서만 진행하기 때문에 방금처럼 Visual Studio 를 GUI 로 열어서 마우스로 `솔루션 빌드` 를 클릭하도록 Dockerfile 에서 지정 할 수는 없습니다. 대신 Visual Studio Build Tools 설치 프로그램을 명령 행으로 실행하여 빌드 도구를 설치하고, Developer Command Prompt for Visual Studio (또는 Developer PowerShell for Visual Studio) 에서 사용 가능한 `MSBuild.exe` 를 통해 빌드해야 합니다.

컨테이너 빌드 과정에서 이러한 환경이 필요한 경우, Microsoft 에서 제공하는 [컨테이너에 Visual Studio Build Tools 설치 가이드](https://docs.microsoft.com/ko-kr/visualstudio/install/build-tools-container?view=vs-2019) 문서를 참고하여, .Net 앱 빌드 전 필요한 도구를 먼저 설치하도록 Dockerfile을 작성할 수 있습니다.

## 베이스 컨테이너 이미지
두 번째 고려사항, 베이스 컨테이너 이미지 입니다. 이는 앞에서 언급한 `regsvr32.exe` 명령으로 COM 라이브러리를 등록하는 것과 관련이 있습니다. [Windows Container 의 경우 4가지의 베이스 이미지가 있습니다.](https://docs.microsoft.com/ko-kr/virtualization/windowscontainers/manage-containers/container-base-images)

- Windows : 데스크탑 Windows 에 포함된 대부분의 프로그램과 라이브러리가 포함된 베이스 이미지 입니다.
- Windows Server Core : Windows Server 에서 데스크탑 GUI 를 제거한 버전.
- Windows Nano Server : 컨테이너에 최적화한 Windows 이미지. 일부 API, PowerShell, WMI 등 도구를 제거하여 사이즈를 더 줄인 이미지.
- Windows IoT Core : IoT 장비(Reapberry Pi 등)에서의 사용에 최적화된 버전의 Windows.

일반적인 웹 앱을 컨테이너로 빌드하는 경우 Windows Server Core 나 Windows Nano Server 를 베이스 이미지로 많이 사용합니다. 모든것을 포함한 Windows 이미지는 사이즈가 너무 크기 때문에 적합하지 않고, IoT Core 또한 IoT 장비용 소프트웨어를 빌드할 때 적합하지 일반적인 웹 앱 빌드와 운영에는 적합하지 않습니다. 

그렇다면 Server Core, Nano Server 만 남는데 여기서 고려할 것은 등록하고자 하는 COM 라이브러리가 64bit용으로도 있는지, 32bit용만 있는지 여부 입니다. Nano Server의 경우 경량화를 위해 일부 도구나 API뿐만 아니라 32bit 지원도 제거되어 있어, 64bit만 지원하기 때문입니다. 그래서 SysWOW64 와 같은 32bit 호환성 지원을 위한 구성요소도 전부 제거되어 있습니다. 때문에 `C:\Windows\System32\regsvr32.exe` 명령을 통해 64bit COM 라이브러리는 등록이 가능하지만, 32bit COM 라이브러리 등록에 사용하는 `C:\Windows\SysWOW64\regsvr32.exe` 는 없기 때문에, 32bit COM 라이브러리는 등록할 수 없습니다.

> Nano Server의 중요한 차이점
>
> Nano Server는 컨테이너 및 마이크로서비스를 기반으로 하는 클라우드-네이티브 애플리케이션을 실행하기 위한 간단한 운영 체제로 또는 훨씬 더 작은 공간을 차지하는 민첩하고 비용 효과적인 데이터 센터 호스트로 최적화되어 있으므로 Nano Server와 Server Core 또는 데스크톱 환경 포함 Server 설치 간에는 중요한 차이점이 있습니다.
> - ...
> - **64비트** 애플리케이션, 도구 및 에이전트**만 지원됩니다.**
> - ...
>
> https://docs.microsoft.com/ko-kr/windows-server/get-started/getting-started-with-nano-server

![](images/nanosyswow64.png)

실제로 Windows Nano Server 컨테이너 이미지로 컨테이너를 만들어 내부에 접속해 확인 해 보면, SysWOW64 디렉토리 자체는 존재하긴 하지만, 대부분의 라이브러리와 실행 파일은 존재하지 않는 것을 확인할 수 있습니다.

![](images/nanoregsvr32.png)

당연하겠지만, `C:\Windows\System32\regsvr32.exe` 명령으로 32bit COM 라이브러리 등록도 불가능 합니다. PG사에서 제공받은 결제 클라이언트 라이브러리의 경우 64bit 버전도 제공받긴 했지만 정상적으로 작동하는 라이브러리가 아니였기 때문에 32bit 버전을 사용해야 했습니다. 때문에 컨테이너화에 적합한 Nano Server 를 사용하지 못하고 Server Core 이미지를 사용해야 했습니다.

# MSBuild 사용을 Visual Studio Build Tools 설치

컨테이너 환경에서 Visual Studio Build Tools 를 사용하려면, 해당 도구가 미리 포함된 공식 컨테이너 이미지가 따로 없기 때문에 직접 설치해야 합니다. 앞서 언급한 [컨테이너에 Visual Studio Build Tools 설치 가이드](https://docs.microsoft.com/ko-kr/visualstudio/install/build-tools-container?view=vs-2019)를 참고하여 Dockerfile 의 빌드 도구 설치 부분을 작성해 봅시다.

먼저 베이스 이미지를 지정합니다. 문서의 권장사항에 따라 .Net 이 미리 포함된 이미지를 사용합니다. 여기서 빌드할 프로젝트는 .Net 5 기반이고, 빌드 과정에서 32bit COM 라이브러리 등록도 필요하므로, Windows Server Core 기반의 .Net 5 SDK 컨테이너 이미지를 사용하겠습니다.

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0-windowsservercore-ltsc2019
```

그리고 `vs_buildtools.exe` 를 다운로드 받아 실행하여 Visual Studio Build Tools 를 설치하도록 작성합니다. 보통 Visual Studio 를 설치하면 중간에 재부팅을 하는 경우도 간혹 있는데, 컨테이너 빌드 중에는 이러한 상황이 발생하면 빌드가 중단되므로, 재부팅이 일어나지 않도록 하기 위해 `--norestart` 옵션을 넣어줍니다. 설치 도중 사용자 입력을 요구하는 경우에도 빌드가 중단되므로, 이러한 상황 또한 발생하지 않도록 `--quiet` 옵션도 넣어줍니다.

또한 설치 시, 컨테이너에 설치할 수 없는 버전의 Windows SDK 는 설치되지 않도록 `--remove` 옵션을 통해 설정해 줍니다.
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0-windowsservercore-ltsc2019

# Restore the default Windows shell for correct batch processing.
SHELL ["cmd", "/S", "/C"]

# Download the Build Tools bootstrapper.
ADD https://aka.ms/vs/16/release/vs_buildtools.exe C:\TEMP\vs_buildtools.exe

# Install Build Tools with the Microsoft.VisualStudio.Workload.AzureBuildTools workload, excluding workloads and components with known issues.
RUN C:\TEMP\vs_buildtools.exe --quiet --wait --norestart --nocache \
    --installPath C:\BuildTools \
    --add Microsoft.VisualStudio.Workload.AzureBuildTools \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10240 \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10586 \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.14393 \
    --remove Microsoft.VisualStudio.Component.Windows81SDK \
 || IF "%ERRORLEVEL%"=="3010" EXIT 0
```
지금 위 Dockerfile 의 경우 `Microsoft.VisualStudio.Workload.AzureBuildTools` 워크로드만 설치하도록 지정되어 있는데요, .Net 앱 빌드에 필요한 도구와 MSBuild도 설치 하도록 추가로 지정해 줍시다. Microsoft 문서 중 [Visual Studio Build Tools 구성 요소 디렉터리](https://docs.microsoft.com/ko-kr/visualstudio/install/workload-component-id-vs-build-tools?view=vs-2019) 문서를 참고하여 아래와 같은 워크로드(구성 요소 묶음) 을 추가로 설치하도록 지정하겠습니다.

- .NET Core 빌드 도구 `Microsoft.VisualStudio.Workload.NetCoreBuildTools`
- 웹 개발 빌드 도구 `Microsoft.VisualStudio.Workload.WebBuildTools`
- MSBuild 도구 `Microsoft.VisualStudio.Workload.MSBuildTools`

웹 개발 도구(`Microsoft.VisualStudio.Workload.WebBuildTools`) 워크로드의 경우, "권장" 에 해당하는 구성요소도 설치하도록 하기 위해 `;includeRecommended` 를 붙여 주었습니다.

```dockerfile
...

# Install Build Tools with the Microsoft.VisualStudio.Workload.AzureBuildTools workload, excluding workloads and components with known issues.
RUN C:\TEMP\vs_buildtools.exe --quiet --wait --norestart --nocache \
    --installPath C:\BuildTools \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10240 \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10586 \
    --remove Microsoft.VisualStudio.Component.Windows10SDK.14393 \
    --remove Microsoft.VisualStudio.Component.Windows81SDK \
    --add Microsoft.VisualStudio.Workload.AzureBuildTools \
    --add Microsoft.VisualStudio.Workload.MSBuildTools \
    --add Microsoft.VisualStudio.Workload.NetCoreBuildTools \
    --add Microsoft.VisualStudio.Workload.WebBuildTools;includeRecommended \
 || IF "%ERRORLEVEL%"=="3010" EXIT 0
```