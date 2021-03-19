---
feature_image: images/img-1-4.png
authors:
- sejun kim
date: "2019-11-28T00:00:00Z"
categories:
- Mate Story

tags:
- NetApp
- NetApp Insight
- Data Fabric
- 후기
title: NetApp Insight 2019 후기(1/2)
---

## 주저리주저리
10월 28일~30일에 라스베가스에서 NetApp의 최대 행사인 NetApp Insight 2019를 다녀왔습니다. 이미 한달(!!)이나 지난 후기입니다만, 그래도 너그러운 마음으로 봐주시면 감사하겠습니다. 우선 행사에 참석할 수 있게 많은 도움을 주신 NetApp 관계자 여러분들께 진심으로 감사의 말씀 올립니다.(열심히 하겠습니다!) 이번 행사는 곧이어 쓸 Microsoft Ignite 2019 행사와 패키지(?)로 가게되어 앞뒤로 휴가를 붙여서 샌프란시스코-라스베가스(행사)-올랜도(행사)-뉴욕 이렇게 다녀왔습니다. 이번 라스베가스에서 느낀 점은 사막은 사막이라는 겁니다. 엄청 건조해서 로션을 하루에 두번씩 발라도 몸이 갈라지는게 느껴질 정도네요. 사막인데다 미국은 온풍기가 돌기에 물에 젖은 수건이 4시간만에 마르는 기적을 볼 수 있었습니다. 다음에 라스베가스를 갈 기회가 또 생긴다면 미니가습기를 챙겨가야겠어요. 서론이 너무 길어졌네요. 행사에 대해 이야기 해보죠~!

## Insight 2019를 가기 전에
NetApp Insight 2019는 작년도, 올해도, 내년에도 만달레이베이 호텔에서 진행됩니다. 우선 티켓을 구매하고 호텔을 잡아야 하는데요. 만달레이베이 리조트와 델라노 호텔 중 하나를 선택하여 숙소를 잡을 수 있습니다. 저는 델라노호텔에서 묵었는데요. 개인적으로는 델라노호텔에서 묵은걸 잘했다고 생각합니다. 행사장과의 거리는 만달레이베이 리조트가 더 좋아보입니다만, 행사장을 가는 길이 카지노를 지나게 되어있고, 우버픽업장소가 지하에 있기에 관광도 함께한다면 델라노호텔이 좀 더 좋지 않았나 생각합니다. 참고로 델라노 호텔은 리셉션에 2~3명만 있어 체크인/아웃이 오래걸리지만 만달레이베이에서도 델라노호텔 체크인이 됩니다. 호텔을 고르실 때 참고하시면 좋습니다. 호텔비는 후불이며 예약시 등록한 카드로 확인을 합니다. 체크아웃 시 결재는 다른 카드로 해도 무방합니다. 참고로 호텔스닷컴이나 기타 홈페이지에서 Wifi 유료라고 써있는건 비즈니스용일 시 유료인 것이며, 기본적으로 무료 Wifi가 제공됩니다. 무료 Wifi의 경우 RDP 및 SSH 연결 등이 제한되어있으며 웹서핑만 가능합니다.

티켓과 호텔예약이 끝났다면 E-mail을 기다립니다. 바로 시간표를 짜기 위해선데요. NetApp 시간표는 같은 시간에 하나의 세션만 등록할 수 있으며, 스케줄에 등록한 세션은 무조건 입장이 가능합니다. 스케줄에 등록되지 않은 세션도 들어갈 수 있습니다만, 만석일 시 입장을 거부당할 수 있기 때문에 시간표를 짤 때 참고하면 좋습니다. 대부분의 세션이 2번씩 하기때문에 스케줄을 잡을 때 참고하시면 좋습니다. 핸드폰에는 NetApp Events라는 App을 설치해 가시면 실시간으로 스케줄 변경 및 장소확인이 가능하니 App을 꼭 다운로드 받아 가시길 바랍니다.

- Android: https://play.google.com/store/apps/details?id=com.netapp.events&hl=en
- IOS: https://apps.apple.com/us/app/netapp-events/id1390017846

![NetApp Insight 2019 시간표](images/netapp_insight_schedule_web.png)

Insight 2019 에서 시험을 무료로 볼 수 있는 혜택이 제공됩니다. 따라서 NetApp 자격증 시험을 보길 원하시는 분은 가이드에 따라 [NetApp Pearson VUE](https://home.pearsonvue.com/netapp) 페이지에서 미리 등록할 수 있습니다. 현장에서 등록도 가능하지만 대기자가 많아 시간을 아끼시려면 사전등록을 하길 추천합니다. 올해는 NetApp 자격증뿐만이 아니라 Microsoft Azure 자격증 중 AZ-300, AZ-301 시험도 무료로 칠 수 있었습니다. 내년에도 시험을 볼 수 있을진 모르겠지만, 현재 NetApp과 Microsoft의 관계를 볼때 내년에도 Insight에서 Microsoft Azure 시험을 볼 수 있지 않을까 예상됩니다.

이제 모든 준비가 끝났습니다. 이제 행사장에 가서 할 일들에 대해 알아봅시다.

## Insight 2019 

### 사전 등록
NetApp Insight는 생각보다 자유로운 분위기의 행사였습니다. 전 비행기 시간으로 인해 사전 등록은 하지 않았지만, 행사 전날 행사장을 가면 사전등록을 할 수 있습니다. 전 당일 등록을 했습니다만, 사전등록을 많이 했는지 당일 등록하는 사람들은 그리 많지 않았습니다. 따라서 굳이 사전 등록은 하지 않아도 될 것 같습니다. 하지만 사전 답사는 가볼만 합니다. 사람이 없는 행사장의 사진을 획득하시려면 이때 가는 것을 추천드립니다. Insight는 Keynote가 둘째 날에 있기에 등록을 급하게 받을 필요가 없습니다.
![Insight2019_행사장1](images/Insight01_web.jpg)
![Insight2019_행사장2](images/Insight02_web.jpg)
![Insight2019_행사장3](images/Insight03_web.jpg)
![Insight2019_행사장4](images/Insight04_web.jpg)
![Insight2019_행사장5](images/Insight05_web.jpg)
![Insight2019_행사장6](images/Insight06_web.jpg)
벌써부터 다음해 행사를 광고하는 NetApp!

### 행사 1일차
첫째날 아침 일찍 행사장에 가서 등록을 한 후 여기저기 둘러보았습니다. 행사장은 열려있지만 부스는 들어갈 수 없었으며, 파트너 부스는 10시경 열었던 걸로 기억합니다. 
![Insight2019_행사장7](images/Insight07_web.jpg)

세션은 9시부터였지만 8시부터 HOL을 할 수 있다고 하여 HOL장으로 이동했습니다. HOL은 훌륭했습니다. Lab과 Lab 환경이 모두 갖춰져있었으며 총 15개의 HOL이 준비되어 있었습니다. HOL 하나에 30분~90분이 소요되기 때문에 매우 많은 시간을 투자해야 하지만, HOL 문서를 다운로드 받을 수 있기에 나중에 HOL 환경은 아니지만 해볼 수 있습니다.
![Insight_HOL_행사장1](images/Insight_hol_01_web.jpg)
![Insight_HOL_행사장2](images/Insight_hol_02_web.jpg)
![Insight_HOL_행사장3](images/Insight_hol_03_web.jpg)

NetApp Insight의 아쉬웠던 부분은 녹화를 제공하지 않는 점에 있습니다. 가뜩이나 영어를 잘 못해서 프레젠테이션을 보고 이해하는 수준인 저로써는 영어력을 최대한 끌어올려 하루종일 영어듣기평가를 하는 기분이었습니다. 영어를 잘하고 싶다는 목표는 매번 있었지만 다시 한번 다짐하게 되는 기회였습니다. NetApp은 스토리지 회사로 시작했지만 최근 Cloud 제품을 출시하면서 DevOps와 Cloud Storage의 혁신을 불러일으키고 있습니다.

첫번째로 들은 세션은 Cloud Sync라는 제품입니다. 이 제품은 AWS S3, Azure Blob, Google Storage, NFS, CIFS, Storage Grid간 데이터 복제를 GUI로 제공하는 것이 특징입니다. 손쉽게 설정하고 증분백업을 이용한 데이터 압축전송을 제공합니다. 생각보다 데이터 복제를 위한 여러가지 방법 및 관리방법이 있습니다만, Cloud Sync를 이용하면 쉽게 설정이 가능하기에 관리자가 스크립트를 짜고 실행하는 업무가 줄어들며 특히 복제 후 파일 확인을 할 필요가 없습니다.
![CloudSync](images/img-1-4.png?width=732&height=474&name=img-1-4.png)

두번째로 들은 세션은 Data Fabric에 대한 내용입니다. Data Fabric이라는 단어는 Azure를 비롯한 AWS에서도 사용할 만큼 NetApp이 만들어낸 대표적인 용어인데요. Data Fabric은 온프레스 및 멀티클라우드 환경에 걸친 다양한 엔드 포인트에서 일관된 기능을 제공하는 아키텍처 및 데이터 서비스 세트를 말합니다. 즉, 어디서든 동일한 데이터 세트를 접근할 수 있도록 지원하는 환경 또는 서비스를 Data Fabric이라고 지칭하고 있습니다. 이를 구현하기 위해 NetApp이 개발한 제품들의 소개와 사용 방법에 대한 내용들을 다뤘고 그 핵심에는 Cloud Sync가 있었습니다.

세번째로 들은 세션은 NetApp Kubernetes Service(NKS) 입니다. NetApp이 Kubernetes Cluster 관리 전문기업인 StackPointCloud라는 업체를 인수하면서 출시한 제품입니다. NKS는 AWS, Azure, GCP, NetApp HCI(물리 장비)에 Kubernetes Cluster를 배포하여 하나의 콘솔에서 서비스를 관리하는 서비스입니다. NKS는 VM을 직접 띄워 Cluster를 구성하는 것도 지원하지만, EKS, AKS, GKS 등 Cloud 벤더사가 제공하는 Kubernetes Service도 NKS Cluster로 연결하여 중앙에서 관리가 가능합니다. 특히 NetApp에서 개발한 [Trident](https://github.com/NetApp/trident)라는 Opensource 제품을 이용한 동적 Disk Mount방식은 매우 편리해 보였으며 사용하는 업체가 많을 것이라 예상되었습니다.

온프레미스 장비 세션도 들었지만 어려운 내용이어서 그냥 나왔습니다. 첫째날 NS0-302 시험도 봤는데 떨어져서 매우 슬펏습니다ㅠㅠ

스압관계로 글을 두개로 나눴습니다!
다음 글도 잘 봐주세요~

[NetApp Insight 2019 후기(2/2)](https://tech.cloudmt.co.kr/2019/11/28/Insight2019_2/)