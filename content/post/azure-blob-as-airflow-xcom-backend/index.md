---
title: Azure Blob Storage 와 XCom 백엔드 커스터마이징으로 Airflow Task 간 큰 데이터 공유하기
authors:
- youngbin-han # 저자 프로필 페이지 경로 입력
date: 2022-04-13T09:51:53+09:00
categories:
- Tech # 새로운 기술 소개 또는 Hands On 이외 기술 관련 글
tags:
- Azure
- Airflow
- Blob Storage
feature_image: 'images/keda-http.png' # 포스트 커버 이미지 경로 (포스트에 포함된 이미지 중 하나 지정. 필드 제거하면 기본 이미지가 나옵니다.)
---

안녕하세요, 지난번에 작성한 Airflow 관련 포스팅인 (사실 Airflow 보다는 Kubernetes 와 연관성이 더 있는 글 입니다만.) ["KEDA를 활용하여 방문자가 있을 때만 작동하는 서비스 배포하기"](/2022/03/11/http-traffic-based-autoscaling-with-keda/)에 이어서, 이번 글에서는 Airflow 에서 Task 간 크기가 큰 데이터를 공유하는 방법에 대해 이야기 해 보고자 합니다. XCom 을 통해 Task 간 데이터를 전달하는 것을 알아보고, 전달할 데이터 크가가 클 때는 어떤 방법을 시도 해 볼 수 있는지, 마지막으로 커스텀 XCom 백엔드 작성으로 큰 데이터도 DAG 에서 별도의 추가적인 업로드나 다운로드 작업 없이 Task 간 주고 받는 방법에 대해 알아보겠습니다.

## XCom - Cross Comunication
Airflow 의 각 워크플로우에 해당하는 DAG는 여러개의 Task로 구성되어 있습니다. 여러 Task가 지정된 순서대로 실행되어 작업 하나가 완성 되는데요, 각 Task는 기본적으로 서로 다른 환경 또는 완전히 다른 머신에서 작동하고 있다고 가정하고 실행됩니다. 이러한 경우 Task간 데이터를 공유 하기가 어려운데요, 이런 환경에서도 데이터를 공유 할 수 있도록 해 주는 Airflow의 기능이 XCom(cross-communication)기능입니다.

각 XCom은 해당 XCom 의 Key 값과, 데이터를 Push 한 Task의 ID(`task_id`), 그리고 해당 Task 를 포함하는 DAG의 ID(`dag_id`)로 구별됩니다.
기본적으로 XCom 을 사용 할 때는 데이터를 공유 할 Task에서 `xcom_push` 메소드로 데이터를 `push` 하거나, 데이터를 가져올 Task에서 `xcom_pull` 메소드로 데이터를 `pull`하여 가져옵니다. 

아래처럼 명시적으로 메소드를 호출하여 XCom 을 통해 데이터를 공유 할 수도 있고.
```python
# Pulls the return_value XCOM from "pushing_task"
value = task_instance.xcom_pull(task_ids='pushing_task')
```

대부분의 Operator에서는 값을 반환할 때 XCom 에 Push 까지 같이 처리하기 때문에, 보통 명시적으로 값을 Push하는 경우는 많지 않습니다. [예를 들어 아래 DAG 는 TaskFlow API로 작성된 ETL 작업을 수행하는 DAG 입니다. (코드 일부 생략, Airflow 소스코드에서 발췌)](https://github.com/apache/airflow/blob/main/airflow/example_dags/tutorial_taskflow_api_etl.py)

`@task` 데코레이터가 있는 각 함수가 Airflow 에서 실행시 각각 별도의 Task 로 실행 되는데요, 이들 Task 함수가 반환하는 값은 Task 함수 내에서 명시적으로 XCom 에 Push/Pull 하지 않아도 Airflow 에서 내부적으로 Push/Pull 을 수행하여 Task 간 데이터가 공유됩니다.

```python
@dag(
    schedule_interval=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=['example'],
)
def tutorial_taskflow_api_etl():
  
    @task()
    def extract():
        data_string = '{"1001": 301.27, "1002": 433.21, "1003": 502.22}'
        order_data_dict = json.loads(data_string)
        return order_data_dict

    @task(multiple_outputs=True)
    def transform(order_data_dict: dict):
        total_order_value = 0

        for value in order_data_dict.values():
            total_order_value += value

        return {"total_order_value": total_order_value}

    @task()
    def load(total_order_value: float):
        print(f"Total order value is: {total_order_value:.2f}")

    order_data = extract() # "extract" Task 에서 반환된 값이 Airflow 에서 내부적으로 XCom 에 자동으로 Push됨
    order_summary = transform(order_data) # "transform" Task 시작시 XCom 에서 Pull. 마찬가지로 값 반환과 함께 XCom 에 자동으로 Push
    load(order_summary["total_order_value"]) # "transform" 에서 Push 한 XCom 데이터 Pull 하여 Task 실행


tutorial_etl_dag = tutorial_taskflow_api_etl()
```

## XCom 에 넣을 데이터가 큰 경우
이러한 XCom 은 Task 간 데이터를 공유할 수 있게 해 주므로 유용하고 많이 쓰이지만, Task 실행시 필요한 설정 값 정도만 전달하는 것을 목적으로 만들어 진 것이여서 큰 사이즈의 데이터를 전달하기에 적합하지는 않습니다. 기본적으로 전달 가능한 데이터 크기가 최대 48KB로 제한되어 있기도 합니다.

```python
# MAX XCOM Size is 48KB
# https://github.com/apache/airflow/pull/1618#discussion_r68249677
MAX_XCOM_SIZE = 49344
XCOM_RETURN_KEY = 'return_value'
```
> [Airflow의 XCom 관련 소스코드 일부. 최대 크기가 고정 되어 있음을 확인할 수 있습니다.](https://github.com/apache/airflow/blob/70065e656950a5948351164bb8f1c5454ed5b7f7/airflow/models/xcom.py#L46)

그렇다면 만약, Task 간 큰 데이터를 주고 받아야 할 때는 어떻게 해야 할까요? 생각 해 볼 수 있는 방법 중 하나로 데이터를 파일로 외부 저장소에 저장하고, 저장된 파일의 정보만 XCom 을 통해 전달하는 방법이 있습니다. 예를 들면 아래의 예제 코드 처럼, 데이터를 Azure Blob Storage 에 파일로 저장한 후 해당 파일의 위치를 XCom 으로 공유하고, 다음 Task 에서 다시 Azure Blob Storage 에서 파일을 가져오는 방법을 생각 해 볼 수 있습니다.

```python
...
@task()
def task_upload_json():
    import json

    large_json_data = {...} # 48KB를 넘는 큰 사이즈의 딕셔너리 데이터라고 가정

    wasb_conn_id="<Airflow 에 설정한 Azure 스토리지 계정 연결 정보 ID>"
    blob_name="<Blob Storage 저장시 파일 이름>"
    container_name="<파일 저장한 Blob Container 이름>"

    # Blob Storage 에 업로드
    hook = WasbHook(wasb_conn_id=wasb_conn_id)
    hook.load_string(json.dumps(large_json_data),
        blob_name=blob_name,
        container_name=blob_name)
    
    # 파일 정보만 반환
    return {"container_name": container_name, "blob_name": blob_name}

@task()
def task_get_and_print_json(container_name, blob_name):
    import json
    wasb_conn_id="<Airflow 에 설정한 Azure 스토리지 계정 연결 정보 ID>"

    # Blob Storage 에서 불러오기
    hook = WasbHook(wasb_conn_id=wasb_conn_id)
    json_data = json.loads(hook.read_file(container_name, blob_name))
    
    # 불러 온 데이터 출력
    print(json_data)
  
# Task 순서 정의
file_meta = task_upload_json()
task_get_and_print_json(file_meta["container_name"], file_meta["blob_name"])
```

Azure Blob Storage 외에, AWS를 사용한다면 S3에 업로드/다운로드 하는 로직을 추가로 넣을 수도 있을 것이고, 로컬 파일시스템에 저장하는 로직을 생각 해 볼 수도 있는 등 다양한 방법이 있겠습니다.

하지만 이렇게 외부 저장소에 직접 올리고 받는 로직을 각 Task 에 넣으면, 비슷한 경우를 만날 때 마다 매번 해당 작업을 수행하는 코드를 넣어줘야 해서 번거롭다는 단점이 있습니다.
또한 DAG 에서 Airflow 가 기본 제공하는 Operator나 필요에 의해 별도로 설치해 사용하는 Operator가 있다면 함께 사용하기 어려울 수도 있다는 문제도 있습니다.

## Custom XCom Backend 작성 및 설정

앞서 소개된 해결 방안은 Task 에 추가적인 로직을 넣어서 해결 해 보는 방법이였다면, Airflow 에서 XCom 을 처리하는 방식을 커스터마이징 하여 내부적으로 작동하는 방식을 변경하여 해결하는 방법도 있습니다. 바로 커스텀 XCom 백엔드를 작성하여 적용하는 것 입니다.