---
title: KEDA를 활용하여 방문자가 있을 때만 작동하는 서비스 배포하기
authors:
- youngbin-han # 저자 프로필 페이지 경로 입력
date: 2022-03-11T09:51:53+09:00
categories:
- Tech # 새로운 기술 소개 또는 Hands On 이외 기술 관련 글
tags:
- Kubernetes
- Container
- KEDA
feature_image: 'images/nanosyswow64.png' # 포스트 커버 이미지 경로 (포스트에 포함된 이미지 중 하나 지정. 필드 제거하면 기본 이미지가 나옵니다.)
---

최근 사내 통합 빌링 시스템의 빌링 데이터 수집 프로그램을 Apache Airflow로 이전하기 위해, Kubernetes 환경에서 Airflow 를 구축하여 운영 해 보는 것을 테스트 해 보고 있습니다. 
Kubernetes 환경의 경우 안정적인 클러스터 운영을 위해 둘 이상의 노드로 클러스터를 구성하는 것이 일반적입니다. 그러다 보니 테스트 환경을 운영 하거나 작은 규모의 서비스를 운영하기에는 비용 부담이 있을 수도 있습니다. 그렇다고 낮은 사양의 노드로 클러스터를 구성하면, 테스트 할 서비스를 모두 배포하기에 어려움이 있을 수도 있습니다. 

그런데 테스트 환경이면 보통 항상 서비스를 켜 두기 보단, 테스트가 필요할 때만 켜는 방식으로 비용 부담을 조금은 줄여 볼 수 있습니다. 이를 매번 수동으로 하기는 번거로우니, 자동으로 필요할 때만 켜지도록 하면 편리할 텐데요, Kubernetes 환경에서는 이 글에서 소개할 KEDA를 활용하여 필요할 때만 자동으로 켜지는 서비스를 배포할 수 있습니다. Airflow 테스트 환경을 구축 사례를 통해 왜 KEDA 를 사용해 보게 되었는지, 어떻게 사용했는지에 대해 정리 해 보고자 합니다.

# 높은 리소스를 사용하지만, 자주 접속할 일은 없는 Airflow Webserver

Airflow 는 다양한 컴포넌트로 구성되어 있는데요. DAG 스케줄링과 실행 상태를 보여주는 Webserver, DAG로 정의된 각종 작업이 정해진 시간에 실행될 수 있도록 스케줄링 하는 Scheduler, 실제 작업을 스케줄링 하고 실행하는 Executor 와 Worker, Airflow 환경 설정 데이터와 DAG 실행 상태와 기록 등을 저장하는 Metadata Database (보통 PostgreSQL 많이 사용) 등으로 구성 되어 있습니다.

이들 중 Metadata Database 의 경우 기존에 사용중인 외부 DB에 연동하는 형태로 구성할 수 있고, Scheduler, Executor, Worker 는 작업이 정해진 시간에 잘 수행되기 위해 항상 실행 되고 있어야 하지만. Webserver 의 경우 항상 켜져 있을 필요는 없습니다. Airflow 환경 설정을 하거나 DAG 상태를 볼 때만 잠깐 켜도 되고, 꺼져 있어도 DAG 가 실행 되는 것에는 문제가 없습니다.

그런데 이렇게 자주 사용할 일이 없는 Webserver 가 아래 사진처럼 메모리를 많이 사용하고, 또 사용 가능한 리소스가 적은 테스트 용도로 구축한 Kubernetes 클러스터에서 운영하다 보니, Webserver 때문에 다른 Airflow 컴포넌트가 제대로 실행되지 않거나, 정해진 시간에 DAG 가 실행 되다가 메모리가 부족하여 실행에 실패하는 경우도 있고, DAG 를 실행하기 위해 Kubernetes 에서 Webserver 를 종료 시키는 경우가 많았습니다. 그래서 Webserver 를 항상 띄어 놓기 보단, 이 글에서 소개할 KEDA 를 활용해서 필요할 때만 잠깐 띄우는 형태로 구성 하여 DAG 가 실행될 때 문제가 없도록 구성 해 보기로 하였습니다. 

# KEDA와 KNative

[KEDA(Kubernetes Event-driven Autoscaling)](https://keda.sh/)는 이벤트 기반으로 Kuibernetes 환경에서 운영중인 컨테이너를 자동 스케일링 하는 Kubernetes 컴포넌트 입니다. HTTP 트래픽, 메시지 큐에서 온 이벤트, 데이터베이스 쿼리 결과 등 다양한 이벤트를 수신 받아 이를 기반으로 컨테이너를 스케일링 할 수 있도록 해 줍니다. 또한 기존에 Kubernetes 환경에 배포한 Service 나 Deployment 를 수정하지 않고 KEDA로 확장하여 사용할 수 있어, 운영중인 Kubernetes 클러스터에 어렵지 않게 도입하여 사용할 수 있습니다.

KEDA와 비슷한 프로젝트로, [KNative](https://knative.dev/) 프로젝트가 있습니다. KNative 의 경우 Kubernetes 환경에 서비리스를 쉽게 구현할 수 있도록 다양한 도구와 컴포넌트를 제공하는 점에서 KEDA 와는 조금 다르다고 할 수 있습니다. 또한 자체적으로 제공하는 자동 스케일러[(KNative Pod Scaler)](https://knative.dev/docs/serving/autoscaling/autoscaling-concepts/#knative-pod-autoscaler-kpa) 등 서버리스 앱 배포에 유용한 기능을 다양하게 제공합니다. 하지만 KNative가 제공하는 강력한 기능을 사용하려면, 기존 Service나 Deployment를 KNative 에서 제공하는 KNative 서비스로 변환해서 사용해야 합니다. 기존 Service 를 그대로 두고 추가적으로 확장하여 사용하고 싶은 경우에는 적합하지 않다고 할 수 있습니다.

자동 스케일링을 적용 하고자 하는 Airflow Webserver 의 경우, [공식 Airflow Helm 차트](https://airflow.apache.org/docs/helm-chart/stable/index.html)를 통해 배포된 서비스 입니다. 이 경우 KNative 를 사용 하려면, Helm 차트를 직접 수정해야 KNative를 사용할 수 있는데 이는 복잡한 방법이여서, KEDA 를 사용하여 기존 서비스를 확장하는 형태로 구성하는 것을 시도 하였습니다.

# KEDA HTTP Add-on

KEDA, KEDA Http 구성 및 작동 원리

KEDA 설치 및 구성

테스트