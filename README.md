# Apache Airflow 운영자 매뉴얼
# 목차

[1.소개](#1-소개)
> [1.1 목적 및 범위](#11-목적-및-범위)

> [1.2 대상 고객](#12-대상-고객)

> [1.3 Airflow 접근 계정](#13-airflow-접근-계정)

[2. Airflow 아키텍처](#2-airflow-아키텍처)
>[2.1 구성 요소 및 종속성](#21-구성-요소-및-종속성)

>[2.2 데이터 처리 파이프라인](#22-데이터-처리-파이프라인)

>[2.3 Workflow 및 Job](#23-workflow-및-job)

[3. 설치 및 구성](#3-설치-및-구성)
>[3.1 시스템 요구 사항](#31-시스템-요구-사항)

>[3.2 설치 옵션](#32-설치-옵션)

>[3.3 구성 매개변수](#33-구성-매개변수)

>[3.4 데이터베이스 백엔드](#34-데이터베이스-백엔드)

>[3.5 메시지 브로커](#35-메시지-브로커)

[4. DAG 및 Job](#4-dag-및-Job)

>[4.1 정의 및 종속성](#41-정의-및-종속성)

>[4.2 오퍼레이터와 센서](#42-오퍼레이터와-센서)

>[4.3 템플릿 및 변수](#43-템플릿-및-변수)

[5. Workflow 및 스케쥴링](#5-workflow-및-스케쥴링)

>[5.1 DAG 실행 및 Job 인스턴스](#51-dag-실행-및-job-인스턴스)

>[5.2 일정 정책 및 트리거](#52-일정-정책-및-트리거)

>[5.3 병렬성과 동시성](#53-병렬성과-동시성)

>[5.4 Job 대기열 및 실행자](#54-job-대기열-및-실행자)

[6. 모니터링 및 경고](#6-모니터링-및-경고)

>[6.1 웹 UI 및 CLI](#61-웹-ui-및-cli)

>[6.2 로그 및 지표](#62-로그-및-지표)

>[6.3 대시보드 및 시각화](#63-대시보드-및-시각화)

>[6.4 경고 및 알림](#64-경고-및-알림)

>[6.5 Airflow Scheduler](#65-airflow-scheduler)

[7. 유지 관리 및 문제 해결](#7-유지-관리-및-문제-해결)

[8. 맞춤화 및 통합](#8-맞춤화-및-통합)

[9. 참조 및 리소스](#9-참조-및-리소스)

----
## 1. 소개
----
### 1.1 목적 및 범위

프로덕션 환경에서 시스템을 설치, 구성 및 관리하기 위한 가이드

### 1.2 대상 고객

배포 및 유지 관리를 담당하는 운영자, 관리자 및 지원 팀

### 1.3 Airflow 접근 계정
----
* <mark>[Airflow UI]</mark>
  * UserName
  * Password
----
* MasterNode02에서 airflow 계정 접근하는 방법

<img width="637" alt="스크린샷 2023-07-13 오전 12 43 50" src="https://github.com/herehyun/airflow/assets/82385436/3409b027-0113-4b02-b18e-ac1412bca60d">

----
## 2. Airflow 아키텍처
----
### 2.1 구성 요소 및 종속성

Workflow를 프로그래밍 방식으로 작성, 예약 및 모니터링하기 위한 플랫폼

<mark> 핵심 구성 요소 </mark>

* 웹 서버: 웹 서버는 Apache Airflow의 <mark>사용자 인터페이스</mark>. workflow 관리, 작업 실행모니터링, 로그 및 매트릭 보기를 위한 웹 기반 콘솔을 제공
  
<img width="282" alt="스크린샷 2023-07-13 오전 12 41 47" src="https://github.com/herehyun/airflow/assets/82385436/441c0a29-7d43-4971-90a6-b115ffe12bbd">

* 스케쥴러 : 스케쥴러는 종속성과 구성된 <mark>일정에 따라 작업을 예약하고 실행</mark>. DAG(Directed Acyclic Graphs)를 사용하여 workflow 및 해당 종속성을 정의

```
  with DAG(
    dag_id = "Schedule_test",
    default_args = default_args,
    schedule_interval = '0 13 * * *',
    catchup = D_CATCHUP,
    max_active_runs = 1,
    tags = ['test', 'scheduler']
  ) as dag:
```
* 실행자 : 작업자 노드에서 작업 실행을 담당. 대기열에서 작업을 가져와 <mark>작업자 노드에 배포</mark>하고 진행 상황을 모니터링
  ```
    [mbsadmin@mn02p ~/bin]$ vi scp_dag.sh 안에 배포하는 데이터 노드들이 적혀있습니다.
    [mbsadmin@mn02p ~]$ vi .bashrc 안에 alias ss='scp_dag.sh' 로 설정해놨습니다.
    [mbsadmin@mn02p <배포를 원하는 파일의 디렉토리>] ss <파일명> 하면 배포가 진행됩니다.
  ```
* 메타데이터 데이터베이스 : 메타데이터 데이터베이스는 <mark>DAG, 작업 및 작업 인스턴스에 대한 메타데이터 저장</mark>을 담당합니다. 스케줄러와 웹 서버에서 workflow를 관리하고 진행상황을 모니터링 하는 데 사용

<img width="816" alt="스크린샷 2023-07-13 오전 12 54 18" src="https://github.com/herehyun/airflow/assets/82385436/0254cdc3-f990-4991-8b90-0e2c13fbee08">
<mark>task status(queued, scheduled, running, success, failed, etc)등등을 메타데이터</mark>라고 함

<img width="805" alt="스크린샷 2023-07-13 오전 12 55 03" src="https://github.com/herehyun/airflow/assets/82385436/2b18b88d-53ff-44ca-9560-134e374d91b9">
<mark>DAGS목록들도 메타데이터</mark>라고 할 수 있습니다.
* 메시지 브로커: Airflow는 메시지 브로커를 사용하여 분산 작업 대기열을 구현합니다.
메시지 브로커는 <mark>작업이 사용 가능한 작업자 간에 공정하게 분배</mark>되고 각 직업의 상태가 시스템을 통해 진행됨에 따라 적절하게 추적되고 업데이트 됨

  + <mark>확장성</mark>: 여러 작업자간에 작업을 분산하고 병렬로 실행할 수 있으므로 Airflow가 수평으로 확장할 수 있음
  + <mark>내결함성</mark>: 작업이 실패하거나 작업자가 다운된 경우 작업을 다시 시도하도록 보장하여 내결함성을 제공
  + <mark>신뢰성</mark>: 작업이 직업자에게 안정적으로 전달되고 상태가 정확하게 추적되고 업데이트 되도록 함
  + <mark>비동기 처리</mark>: 작업을 비동기적으로 처리할 수 있으므로 작업자가 기존 작업을 실행하는 동안 스케쥴러가 새 작업을 계속 예약 할 수 있음

<mark> 구성 요소 간의 종속성</mark>

웹 서버는 메타데이터 데이터베이스에 의존하여 workflow및 작업에 대한 정보를 표시
스케쥴러는 메타데이터 데이터베이스를 사용하여 workflow 및 작업 종속성을 관리하고 메시지 브로커에 의존하여 작업 일정을 처리
실행자는 메시지 브로커에 의존하여 작업을 수신하고 작업자 노드에 배포
작업자 노드는 작업을 수신 및 실행하기 위해 executor에 의존하고, 작업 메시지를 수신하고 executor와 통신하기 위해 메시지 브로커에 의존


### 2.2 데이터 처리 파이프라인

Airflow는 workflow를 프로그래밍 방식으로 작성, 예약 및 모니터링하기 위한 플랫폼
사용자는 독립적으로 실행할 수 있는 작업 단위인 작업으로 구성된 DAG로 workflow를 정의할 수 있음
이러한 작업은 데이터 수집, 정리, 변환, 보강 및 분석과 같은 광범위한 데이터 처리 작업을 수행가능.

Airflow의 데이터 처리 파이프라인 개요

* <mark>데이터 수집</mark> : 파이프라인의 첫 번째 단계는 데이터베이스, API 또는 외부 데이터 피드와 같은 다양한 소스에서 데이터를 수집하는 것 입니다. MySQL, PostgreSQL등 여러 소스에서 데이터를 쉽게 수집 할 수 있음.
* <makr>데이터 정리</mark> : 데이터가 수집되면 유효하지 않거나 누락된 값, 중복 또는 기타 불일치를 제거하기 위해 정리가 필요할 수 있습니다. 이것은 정리작업을 수행하기 위해 shell명령을 실행 할 수있는 BashOperaotr와 같은 연산자를 사용하여 수행할 수 있음.
* <mark>데이터 변환</mark> : 파이프라인의 다음 단계는 데이터를 쉽게 처리하거나 분석할 수 있는 형식으로 변환하는 것입니다. Airflow는 Python스크립트를 실행하여 변환을 수행할 수 있는 PythonOperator와 같은 다양한 데이터 변환 작업을 위한 연산자를 제공.
* <mark>데이터 보강</mark> : 데이터 보강에는 데이터에 더 많은 컨텍스트 또는 가치를 제공하기 위해 기존 데이터 세트에 추가 데이터 또는 정보를 추가하는 작업이 포함됨. Airflow는 SQL쿼리를 실행하여 데이터 집합을 조인할 수 있는 SQLOperator와 같은 다양한 데이터 보강 작업을 위한 연산자를 제공.
* <mark>데이터 분석</mark> : 파이프라인의 마지막 단계는 데이터를 분석하여 통찰력을 얻거나 예측하는 것. Airflow는 데이터 분석을 위해 Spark 작업을 제출할 수 있는 SparkSubmintOperator와 같은 다양한 데이터 분석작업을 위한 연산자를 제공.

### 2.3 Workflow 및 Job

workflow는 <mark>작업</mark>과 <makr>작업 간 종속성</mark>으로 구성된 DAG로 정의.
* <mark>default_arg</mark>라는 딕셔너리를 DAG에 파라미터로 넘기면 모든 operator들에 적용됨.

  ```
  default_args = {
    'start_date' : D_START_DATE,
    'owner': D_OWNER,
    'run_as_user' : D_RUN_AS_USER,
    'on_success_callback' : d_success_callback,
    'on_failure_callback' : d_failure_callback,
    'trigger_rule' : D_TRIGGER_RULE_DEFAULT,
  }
  ```
  ```
  with DAG(
    dag_id = "WORK_TEST",
    default_args = default_args,
    schedule_interval = '0 13 * * *',
    catchup = D_CATCHUP,
    max_active_runs = 1,
    tags = ['TEST', 'schedule']
  ) as dag:
  ```

  workflow의 각 작업은 실행해야 하는 작업 단위. 작업은 수행할 특정 유형의 작업을 나타내는 미리 정의된 Airflow 클래스인 연산자를 사용하여 정의
  Airflow는 shell 명령을 실행하기 위한 BashOperaotr, Python 스크립트를 실행하기 위한 PythonOperator 등과 같은 다양한 유형의 작업을 위한 광범위한 기본 제공 연산자를 제공합니다.

* <mark>DummyOperator</mark> : 아무 작업도 하지 않는 연산자로 여러 다른 작업들을 그룹화 하는데 사용
    ```
    from airflow.operators.dummy_operator import DummyOperator

    with TaskGroup("section_1", tooltip="Tasks for section_1") as section_1:
    task_1 = DummyOperator(task_id="task_1")
    task_2 = BashOperator(task_id="task_2" bash_command='echo_1')
    task_3 = DummyOperator(task_id="task_3")
    ```

* <mark>BashOperator</mark> : Bash Shell Script를 실행하는 Operator

  ```
  # bash_command(str) : bash Shell Script
  # env(dict) : 기본 값은 none 이나 사용될 경우 dict 형태의 환경변수 선언하는 내용을 기술
  # output_encoding(str) : bash_command의 출력값 인코딩

  from airflow.operator.bash_operator import BashOperator
  bash_task = BashOperator(
    task_id="bash_task",
    bash_command='echo "here is the message: \ '$message\'"',
    env={'message': '{{ dag_run.conf["message"] if dag_run else "" }}'},
    output_encoding='utf-8',
  )
  ```
* <mark>TriigerDagRunOperator</mark> : 지정된 DAG를 실행하는 트리거
  ```
  # trigger_dag_id (str) : 트리거할 dag_id
  # conf (dict) : dag 실행에 대한 구성
  # execution_date ( str or datetime.datetime) : 실행할 날짜
  # reset_dag_run (bool) : 이미 실행되고 있는 dag라면 기존의 실행을 지우고 재실행을 하는지 여부
  # wait_for_completion (book) : 이전 dag의 실행완료를 기다릴지 여부
  # poke_interval (int) : wait_for_completion=True일 때 dag 실행 상태를 확인하기 위한 interval

  from airflow.operators.trigger_dagrun import TriggerDagRunOperator

  trigger = TriggerDagRunOperator(
    task_id="test_trigger_dagrun",
    trigger_dag_id="example_trigger_target_dag",
    conf={"message": "Hellow World"},
  )
  ```
  DAG는 <mark>작업 간의 흐름과 종속성</mark>을 정의합니다. DAG의 각 작업은 현재 작업을 시작하기 전에 완료해야 하는 작업인 하나 이상의 업스트림 종속성을 가질 수 있습니다.
  마찬가지로 각 작업에는 현재 작업이 완료된 후에만 시작할 수 있는 작업인 하나 이상의 다운스트림 종속성이 있을 수 있습니다.

  <img width="282" alt="스크린샷 2023-07-13 오전 12 41 47" src="https://github.com/herehyun/airflow/assets/82385436/441c0a29-7d43-4971-90a6-b115ffe12bbd">
  
<div align="center">
&darr; DAG가 정의되면 특정 간격으로 실행되도록 <mark>예약</mark>하거나
</div>

```
with DAG(
  schedule_interval = '0 7 2 * *',
) as dag:
```

<div algin="cnter">
&darr; 수동으로 트리거할 수 있습니다.
</div>
<img width="940" alt="스크린샷 2023-07-13 오전 1 51 32" src="https://github.com/herehyun/airflow/assets/82385436/0a40d372-8aa5-4cab-81f5-161a986e2a49">
<div align="center">
  &darr;
</div>

<img width="893" alt="스크린샷 2023-07-13 오전 1 51 52" src="https://github.com/herehyun/airflow/assets/82385436/699ed2de-be51-4d4f-8989-d2ce1811e789">

&darr; DAG가 실행되면 Airflow는 실행 중인 DAG의 <mark>특정 인스턴스를 나타내는 DAG</mark> 실행을 생성합니다.

<img width="578" alt="스크린샷 2023-07-13 오전 2 02 16" src="https://github.com/herehyun/airflow/assets/82385436/665a637d-0356-4002-9b4c-08ad4ffac6db">

DAG 실행의 각 작업은 실행 중인 작업의 특정 인스턴스를 나타내는 <mark>작업 인스턴스</mark>로 표시됩니다.
&darr; <img width="752" alt="스크린샷 2023-07-13 오전 2 09 58" src="https://github.com/herehyun/airflow/assets/82385436/6ae5aa51-f728-4214-9c5e-dcd23babd6f8">

Airflow의 작업은 DAG 실행에서 작업을 실행하는 실제 프로세스입니다. 작업은 대기열에서 작업을 가져와 작업자노드에서 실행하고 진행 상황을 모니터링하는 역할을 합니다.
작업은 단일 시스템에서 실행되거나 여러 시스템에 분산 될 수 있습니다.

Airflow는 workflow 및 작업 실행을 모니터링하기 위한 <mark>웹 기반 UI</mark>를 제공합니다.
<mark>UI</mark>를 통해 사용자는 DAG를 시각화하고, 작업 로그를 보고, 실행 중인 작업의 진행 상황을 모니터링 할 수 있습니다.
&darr; <img width="799" alt="스크린샷 2023-07-13 오전 11 53 19" src="https://github.com/herehyun/airflow/assets/82385436/6713afcc-ac50-454e-8ceb-dd1fb885e4b3">

또한 실패한 작업의 디버깅 및 문제 해결을 위한 도구도 제공.
<img width="1104" alt="스크린샷 2023-07-13 오후 7 48 37" src="https://github.com/herehyun/airflow/assets/82385436/0d06eea4-c8c8-4a62-959c-3533922185b1">

요약하면 Apache Airflow의 workflow는 운영자가 나타내는 작업으로 구성된 DAG로 정의됩니다. 작업은 DAG 실행에서 작업을 실행하는 프로세스입니다.
Airflow는 workflow 및 작업 실행을 모니터링하기 위한 웹 기반 UI를 제공하여 복잡한 데이터 처리 workflow를 쉽게 시각화하고 관리할 수 있도록 합니다.
* clear 방법: 원하는 task 클릭 &rarr; 우측에 <mark>clear</mark> 클릭 후 clear
  현재 <mark>downstream</mark> 과 <mark>Reculsive</mark> 에 선택되어있는데
     - <mark>downstream</mark>은 dependency에 속해있는 하위 task도 모두 clear된다.
     - <mark>Reculsive</mark>은 하위 DAG 및 상위 DAG의 모든 task를 의미
 
<img width="1106" alt="스크린샷 2023-07-13 오후 7 54 54" src="https://github.com/herehyun/airflow/assets/82385436/09977512-91b1-4fbc-ac63-534d20857279">

* Dynamic Mapping
  ```
  PythonOperator.partial(
         task_id = "FORECAST",
         python_callable = forecast,
  ).expand(op_args = XComArg(FORECAST_LIST))
  ```
 + <mark>expand()</mark> : mapping하고자 하는 파라미터를 전달합니다. 각 입력 값에 대해 별도의 병렬 Task가 생성.
 + <mark>partial()</mark> : expand() 함수로 생성된 모든 mapped Task에 일정하게 유지되는 파라미터를 전다.ㄹ

<img width="761" alt="스크린샷 2023-07-13 오후 7 59 59" src="https://github.com/herehyun/airflow/assets/82385436/415f7df5-9c29-41e5-8bf3-246ae0a95dcc">

----
## 3. 설치 및 구성
----
### 3.1 시스템 요구 사항

Apache Airflow를 설치하기 전에 운영자는 시스템이 원하는 배포 옵션에 대한 하드웨어 및 소프트웨어 요구 사항을 충족하는지 확인해야 합니다.

### 3.2 설치 옵션

Apache Airflow는 독립 실행형, 클러스터 또는 컨테이너화된 배포를 포함하여 여러 가지 방법으로 설치할 수 있습니다.
각 옵션에는 사용 사례와 데이터 처리 파이프라인의 규모에 따라 고유한 장점과 장단점이 있습니다.

### 3.3 구성 매개변수

Apache Airflow는 사용자가 특정 요구 사항과 요구 사항을 충족하도록 다양한 매개 변수를 구성할 수 있는 고도로 사용자 지정 가능한 플랫폼

#### 원래는 <mark>airflow.cfg</mark> 에서 통제를 하는것이지만, 현재 Ambari라는 UI를 사용하고있어서 대부분 Ambari UI에서 수정하면 다 적용됨.
* <mark>airflow_home</mark> : 구성 파일, 로그 및 기타 시스템 파일을 포함하는 Airflow 홈 디렉토리의 위치를 지정
  + AIRFLOW_HOME = /usr/hmg/2.2.1.0-552/airflow
* <mark>dags_folder</mark> : worflow를 정의하는 Python 스크립트가 포함된 DAG 디렉터리의 위치를 지정
  + dags_floder = /usr/hmg/2.2.1.0-552/airflow/dags
* <mark>max_active_runs_per_dags</mark> : 이 매개변수는 동시에 실행할 수 있는 최대 활성 DAG 실행 수를 지정
   + Ambari &rarr; Airflow &rarr; CONFIGS &darr; 
<img width="960" alt="스크린샷 2023-07-13 오후 8 05 47" src="https://github.com/herehyun/airflow/assets/82385436/ec56631f-7a68-4c18-aba7-006cf101b4e9">
* <mark>max_threads</mark> : 스케쥴러와 웹 서버에서 사용할 수 있는 최대 스레드 수를 지정
* <mark>parallelism</mark> : 실행자가 동시에 실행할 수 있는 최대 작업 수를 지정
  + 병렬성을 최대로 활용하기 위해서, parallelism은 "시스템 cpu 코어수 -1개"로 설정해주는 것이 좋다고 함
  + Ambari &rarr; Ariflow &rarr; CONFIGS &darr; <img width="1053" alt="스크린샷 2023-07-13 오후 9 12 06" src="https://github.com/herehyun/airflow/assets/82385436/970b4fca-6ae7-4eb5-ad34-ae7f83cd36a2">
* <mark>sql_alchemy_conn</mark>: 이 매개변수는 Airflow가 DAG, 작업 및 작업 인스턴스에 대한 정보를 저장하는 데 사용하는 메타데이터 데이터베이스에 대한 연결 문자열을 지정
  + 현재 connection 정보는 'mysql+mysqldb://airflow_test:airflow_password@mn01p:[port]/airflow_test/charset=utf8`를 쓰고있음
  + Ambari &rarr; Ariflow &rarr; CONFIGS &darr; <img width="1053" alt="스크린샷 2023-07-13 오후 9 12 17" src="https://github.com/herehyun/airflow/assets/82385436/0c3f5c1e-1837-4627-8ca7-76b8b6b552c5">
* <mark>logging_level></mark> : Airflow의 로깅 수준을 지정. 이 매개변수를 낮은 수준으로 설정하면 로깅 출력의 양이 줄어들고 시스템 성능이 향상될 수 있음
  + DEBUG : DEBUG 수준은 디버깅에 사용할 수 있는 자세한 정보를 제공하는 메시지에 사용.
  + INFO : 시스템 또는 작업 흐름의 상태에 대한 일반적인 정보를 제공하는 메시지에 사용.
  + WARNING : 잠재적인 문제 또는 워크플로 실행에 영향을 줄 수 있는 문제를 나타내는 메시지에 사용.
  + ERROR : workflow가 실패할 수 있는 오류가 발생했음을 나타내는 메시지에 사용.
  + CRITICAL : workflow가 완전히 실패할 수 있는 심각한 오류가 발생했음을 나타내는 메시지에 사용.
<img width="1048" alt="스크린샷 2023-07-13 오후 9 12 28" src="https://github.com/herehyun/airflow/assets/82385436/f3198d39-a419-4398-b3f1-f64ddf00010d">
* <mark>remote_log_conn_id</mark> : 이 매개변수는 Airflow가 로그를 보는데 사용하는 원격 로깅 서버의 연결 문자열을 지정
  + 우리는 local로 지정하므로 따로 지정 할 필요가 없음
* <mark>python_callable</mark> : DAG실행 시 실행할 PythonOperator가 호출할 함수에 매개변수를 전달
  <img width="321" alt="스크린샷 2023-07-13 오후 9 28 24" src="https://github.com/herehyun/airflow/assets/82385436/448bb624-59cb-45ac-b90f-eb5a799915eb">

&uarr; 대부분 함수를 불러오기 위해 사용됨

* <mark>schedule_interval</mark> : 스케쥴링 횟수 날짜 시간을 지정하는 항목
  * <mark>운영관점</mark> Airlfow에 내장되어있는 schedule_interval 로 schedule을 관리해야하지만 세밀한 schedule관리는 힘들어 각 <mark>MAIN_DAG안</mark>에 <mark>schedule 함수</mark>를 선언하고 관리함
 <img width="1058" alt="스크린샷 2023-07-13 오후 9 28 38" src="https://github.com/herehyun/airflow/assets/82385436/d6db3520-147a-44f7-9b44-9cbebd7b5c94">
* <mark>wait_for_completion</mark> : 외부 dag가 완료될때까지 기다릴 지 유부값
  * wait_for_completion 가 'True'로 설정되면 작업이 실패하면 전체 워크플로가 실패하고 후속 작업이 실행되지 않음(상위 DAG가 성공해야함)
  * 'False'로 설정되면 작업이 실제로 성공적으로 완료되었는지 여부에 관계없이 작업 인스턴스가 대기열에 들어간 직후에 성공한 것으로 표시
  * <mark>reset_dag_run</mark> : 이미 존재하는 경우 기존 dag 실행을 지울지 여부
  * <mark>trigger_dag_id</mark> : 사용할 DAG를 불러오는 역할

<div align="center">
<img width="302" alt="스크린샷 2023-07-13 오후 9 35 48" src="https://github.com/herehyun/airflow/assets/82385436/4f90c569-1016-4d02-ae5f-114ec5acd300">
</div>

* <mark>conf</mark> : 외부 dag를 호출하기 위한 config

<div align="center">
<img width="657" alt="스크린샷 2023-07-13 오후 9 35 58" src="https://github.com/herehyun/airflow/assets/82385436/07fe151b-4b12-4bef-8a41-614dee1090b8">
</div>

<div align="center">
  &uarr; dict 형태로 받아서 config를 불러온다.
</div>

* <mark>trigger_rule</mark> : 의존하는 작업의 상태에 따라 작업을 트리거하는 방법을 지정할 수 있음
  * 현재는 all_success로 지행하고있음 

| 값 | 동작 방식 |
|:--:|:---:|
| all_success | 모든 상위 Task 실행 성공 |
| all_failed  | 모든 상위 Task가 실행 실패, 또는 upstream_failed 상태 |
| all_done    | 모든 상위 Task 실행 완료 |
| one_failed  | 하나 이상의 상위 Task 실패 |
| one_success | 하나 이상의 상위 Task 성공 |
| none_failed | 모든 상위 Task가 실패 또는 upstream_failed가 아니다 |
| none_failed_min_one_success | 모든 상위 Task가 실패 또는 upstream_failed가 아니고 하나 이상의 상위 Task가 성공 |
| none_skipped | 건너뛴 상위 Task 없음 |
| always | Task 종속성 없이 항상 실행 |

### 3.4 데이터베이스 백엔드

Apache Airflow는 <mark>데이터베이스</mark> 백엔드를 사용하여 DAG, Job 및 Job 인스턴스에 대한 메타데이터를 저장합니다.
운영자는 Airflowdhk <mark>호환</mark>되고 예상되는 데이터 불륨 및 동시성을 처리할 수 있는 데이터베이스 백엔드를 선택해야 합니다.
* 현재 <mark>DBeaver</mark>에서 <mark>MariaDB</mark>를 이용해서 관리하고 있음

<div align="center">
<img width="413" alt="스크린샷 2023-07-13 오후 9 48 17" src="https://github.com/herehyun/airflow/assets/82385436/396b0dba-b133-4d1d-93fe-cb491e9c329b">
</div>

### Airflow with MariaDB 접속 정보

<div align="center">
DBeaver 접속
</div>
<div align="center">
&darr;
</div>
<div align="center">
<img width="1121" alt="스크린샷 2023-07-13 오후 9 48 28" src="https://github.com/herehyun/airflow/assets/82385436/e3f8cfff-e0d9-4cf5-93ba-5875acf67c90">
</div>
<div align="center">
&darr;
</div>
<div align="center">
MariaDB 클릭
</div>
<div align="center">
&darr;
</div>
<div align="center">
<img width="1116" alt="스크린샷 2023-07-13 오후 9 48 41" src="https://github.com/herehyun/airflow/assets/82385436/13cb7564-1dc5-4d94-9c67-d375ef0b8234">
</div>
<div align="center">
&darr;
</div>

```
          Server  Host : ip주소
          port         : port주소
          database     : airflow_test
          username     : airflow_test
          password     : airlfow_password

```

<div align="center">
<img width="1117" alt="스크린샷 2023-07-13 오후 9 48 50" src="https://github.com/herehyun/airflow/assets/82385436/cd77b579-6f72-47bf-8786-d57145586a7b">
</div>

> Airflow meta export/import

   1. [airflow@mn02p ~]$ airflow connections export --format json meta.json
   운영에서 메타 땡겨옴


   2. [airflow@개발mn01 ~]$ airflow connections import meta.json
   개발에서 import

   로 개발서버와 운영서버의 airflow meta정보를 맞출 수 있음

### 3.5 메시지 브로커

Apache Airflow는 메시지 브로커를 사용하여 시스템 구성 요소 간의 Job 예약 및 메시징을 처리.
운영자는 Airflow와 호환되고 예상되는 메시지 양과 안정성을 처리할 수 있는 메시지 브로커를 선택해야함

### 3.6 보안 및 인증

운영자는 무단 액세스 또는 데이터 유출을 방지하기 위해 Airflow가 안전하게 보호되고 적절하게 인증되었는지 확인 필요
여기에는 인증 및 권한 부여 정책 구성, 비밀 및 자격 증명 관리, 조직 및 규정 요구 사항 준수 보장이 포함됩니다.

<div align="center">
<img width="591" alt="스크린샷 2023-07-13 오후 10 01 11" src="https://github.com/herehyun/airflow/assets/82385436/fd8fd678-bfaa-4a4f-bb39-c12dedd995b4">
</div>

<div align="center">
&uarr; 위처럼 권한 : 그룹이 airflow:hadoop으로 맞춰져 있어야 함.
</div>

권한이 달라서 변경이 필요할 시

```
[airflow@mn02p /usr/hmg/2.2.1.0-552/airflow/dags]$ chown <소유자명>:<그룹명> <파일이름>
[airflow@mn02p /usr/hmg/2.2.1.0-552/airflow/dags/PROC_TEST/PKG_TEST]$ chown airflow:hadoop
PR_TEST.py
# chown -R <소유자명>:<그룹명> <폴더> 를 사용하면 폴더 밑 하위 디렉토리의 권한도 적용시켜준다.
```

```
[mbsadmin@mn02p ~/bin]$ vi scp_dag.sh 안에 배포하는 데이터 노드들이 적혀있음.
[mbsadmin@mn02p ~]$ vi .bashrc 안에 alias ss='scp_dag.sh'로 설정해놨습니다.
[mbsadmin@mn02p <배포를 원하는 파일의 디렉토리>] ss <파일명> 하면 배포
```
<img width="1082" alt="스크린샷 2023-07-13 오후 10 06 16" src="https://github.com/herehyun/airflow/assets/82385436/da1de746-f239-490b-84f0-076739ddd60d">

----
## 4. DAG 및 Job
----
### 4.1 정의 및 종속성

Apache Airflow Workflow는 특정 순서로 특정 종속성과 함께 실행되는 여러 Job으로 구성된 DAG(Directed Acyclic Graph)를 사용하여 정의됩니다.
운영자는 DAG 및 관련 Job을 정의하는 방법과 Job간의 종속성을 처리하는 방법을 이해해야 합니다.

### 4.2 오퍼레이터와 센서

Apache Airflow는 SQL 쿼리 실행, 이메일 전송 또는 특정 이벤트 발생 대기와 같은 일반적인 Job을 수행하기 위한 몇 가지 기본 제공 연산자 및 센서를 제공합니다.
오퍼레이터는 이러한 오퍼레이터와 센서를 사용하는 방법과 맞춤형 오퍼레이터와 센서를 만드는 방법을 이해해야 합니다.

* Operator : 2.3 에서 설명됨
* Sensor : 현재 사용하고 있는 Sensor는 <mark>PythonSensor</mark>를 사용하고있다.
  * 사용 목적 : 수신 IF가 잘 들어와있는지 확인하는 목적
  * PythonSensor
    ```
       from airflow.sensor.python import PythonSensor

       op_kwargs = {
            "conf_fname" : conf_fname,
            "P_YYYYMMDD" : '{{ dag_run.conf.get("P_YYYYMMDD") }}',
       }
       wait_for_input_file = PythonSensor(
           task_id = f'wait_for_input_file',
           python_callable = recv.sftp_sensor,
           op_kwoargs = op_kwargs,
           poke_interval = T_POKE_INTERVAL,
           timeout = T_TIMEOUT,
       )

        # sftp_sensor 라는 함수를 생성해서 sftp파일은 센싱할 수 있게 한다.
    
    ```
   * poke_interval : Sensor가 조건을 확인하는 다음번 주기(초)를 결정함
   * timeout : Sensor가 조건을 확인을 시도할 최대 시간(초)을 설정. 이 시간까지도 조건을 만족하지 못하면, 해당 태스크는 실패

* Hook : <mark>FTPHook</mark>와 <mark>SFTPHook</mark>를 사용함
   * SFTPHook

       ```
       from airflow.providers.sftp.hooks.sftp import SFTPHook
       ```
       <div align="center">
       <img width="633" alt="스크린샷 2023-07-13 오후 10 22 34" src="https://github.com/herehyun/airflow/assets/82385436/7cabe9b2-a708-4b56-a588-32294e4a9961">
       </div>
   * FTPHook
     ```
     from airflow.providers.sftp.hooks.sftp import FTPHook
     ```
     <div align="center">
     <img width="476" alt="스크린샷 2023-07-13 오후 10 23 22" src="https://github.com/herehyun/airflow/assets/82385436/267bf3b5-e91c-44bd-ae5c-06bc19574bff">
     </div>

### 4.3 템플릿 및 변수

여러 템플릿과 변수를 제공 하고, DAG 및 Job을 매개변수화하여 재사용 가능하고 유연한 Workflow를 보다 쉽게 생성할 수 있음
<img width="636" alt="스크린샷 2023-07-13 오후 10 23 33" src="https://github.com/herehyun/airflow/assets/82385436/e4ad3b1d-1894-4138-a6d8-3c3679e3fc39">

* ti : ti 개체는 "TaskInstance"를 나타내며 현재 실행 중인 작업 인스턴스에 대한 참조
* Xcom : Xcom은 DAG 내의 작업이 서로 소량의 데이터를 교환할 수 있도록 하는 Airflow에서 제공하는 메커니즘

## 5. Workflow 및 스케쥴링

### 5.1 DAG 실행 및 Job 인스턴스

Airflow Workflow는 특정 시간에 또는 특정 이벤트에 대한 응답으로 싱행되도록 예약된 여러 Job 인스턴스로 구성된 DAG실행으로 실행됩니다.
운영자는 DAG 실행 및 Job 인스턴스를 관리하는 방법과 진행 상황 및 상태를 모니터링 하는 방법을 이해해야 합니다.
* 현재 DAG를 실행시키는 방법 두개 존재

  <img width="216" alt="스크린샷 2023-07-13 오후 10 23 41" src="https://github.com/herehyun/airflow/assets/82385436/31282733-74ab-4a43-b2de-bc250269a195">

  1. Trigger DAG
 
  2. Trigger DAG w/ config
 
    + config를 넣을 때
 
<div align="center">
<img width="1125" alt="스크린샷 2023-07-13 오후 10 23 51" src="https://github.com/herehyun/airflow/assets/82385436/13e52b4d-f1f6-46ce-bcf0-c9e3b3c08b9c">
</div>
<div align="center">
필요한 config를 dictionary 형태로 넣어주면 된다.
</div>

### 5.2 일정 정책 및 트리거

Airflow는 시간 기반 일정, <mark>cron</mark> 표현식 및 트리거 기반 일정을 포함하여 Job 및 Workflow를 예약하기 위한 여러 일정 정책 및 트리거를 제공

<mark>def _check_schedule</mark>를 사용하는 이유
  1. Airflow를 그냥 실행하게 되면 처음 시작지점에서 바로 airflow가 작동하게 되는 이슈가 있는데 이를 방지하고자 적용함
  2. crontabl schedule로는 복잡한 schedule을 만들 수 없음


### 5.3 병렬성과 동시성

Apache Airflow는 Job 동시성, Job자 동시성 및 풀 관리르 포함하여 Job 및 Workflow를 병렬화하고 실행하기 위한 여러 옵션을 제공 운영자는 Airflow에서 parallelism 및 동시성을 관리하는 방법과 성능 및 리소스 사용을 최적화하는 방법을 인지해야함

> [3.3구성 매개변수 참고](#3-구성-매개변수)

### 5.4 Job 대기열 및 실행자

Apache Airflow는 Job 대기열 및 executor를 사용하여 Job 및 Workflow의 예약 및 실행을 관리합니다.
운영자는 데이터 Workfolw의 효율적이고 안정적인 처리를 보장하기 위해 Job대기열 및 executor를 구성하고 관리하는 방법을 이해해야 합니다.

## 6. 모니터링 및 경고 

### 6.1 웹 UI 및 CLI

Apache Airflow는 모니터링 및 관리를 위한 웹 기반 사용자 인터페이스와 명령줄 인터페이스를 제공합니다.
운영자는 이러한 인터페이스를 사용하여 Job, Workflow 및 시스템 구성 요소의 상태를 보는 방법을 이해해야 합니다.

### 6.2 로그 및 지표

Apache Ariflow는 모니터링 및 문제 해결을 위해 로그 및 지표를 생성합니다. 운영자는 시스템의 문제를 감지하고 해결하기 위해 이러한 로그 및 매트릭을 관리하고 분석하는 방법을 이해해야 합니다.

* pyspark log 확인하는 방법:

<div align="center">
<img width="1094" alt="스크린샷 2023-07-13 오후 10 39 27" src="https://github.com/herehyun/airflow/assets/82385436/beb36ac5-3841-4b99-b29d-ca3e548aa54f">
</div>
<div align="center">
&darr;
</div>
<div align="center">
<img width="1094" alt="스크린샷 2023-07-13 오후 10 39 36" src="https://github.com/herehyun/airflow/assets/82385436/88e733ae-034b-416f-ab0f-249e164dc1a5">
</div>
<div align="center">
&darr;
</div>
<div align="center">
<img width="1104" alt="스크린샷 2023-07-13 오후 10 39 44" src="https://github.com/herehyun/airflow/assets/82385436/71b132c7-8ca9-4867-89fa-a6a1dac74d98">
</div>
<div align="center">
&darr;
</div>
<div align="center">
<img width="1104" alt="스크린샷 2023-07-13 오후 10 39 50" src="https://github.com/herehyun/airflow/assets/82385436/214a18ae-4186-4f73-8e5e-292eed7945fd">
</div>
<div align="center">
&darr;
</div>
<div align="center">
UI에 들어가서 다양한 정보를 얻을 수 있음
</div>

### 6.3 대시보드 및 시각화

Apache Airflow는 시스템 메트릭 및 성능의 대시보드 및 시각화를 만들기 위한 몇 가지 옵션을 제공

### 6.4 경고 및 알림

Apache Airflow는 Job 실패 또는 시스템 오류와 같은 시스템 이벤트에 대한 경고 및 알림을 설정하기 위한 몇가지 옵션을 제공합니다.
운영자는 문제에 대한 시기적절하고 효과적인 대응을 보장하기 위해 이러한 경고 및 알림을 구성하고 관리하는 방법을 이해해야 합니다.

```
SMTP(Simple Mail Transfer Protocol)
: HTTP가 인터넷을 통해 웹 페이지를 전송하기 위해 컴퓨터가 사용하는 프로토콜인 것처럼 단순 메일 전송 프로토콜(SMTP)은 이메일을 전송할 때 사용되는 프로토콜.
```
> SMTP 서버 연결하기 : smtplib.SMTP()

>이메일 서버에 hello 메세지 보내기 : ehlo()

>SMTP 서버의 포트 연결: starttls()

>이메일 보내기 : sendmail()

### 6.5 Airflow Scheduler

* 일정 간격으로 Airflow 메타데이터 데이터베이스를 쿼리하여 실행 준비가 된 작업을 찾고, 이러한 작업 각각에 대해 작업 인스턴스를 생성하고 적절한 실행기로 보냄.
* Ariflow 스케쥴러는 작업간의 의존성도 관리

Airflow 스케쥴러가 중지되면 예약된 작업이 더 이상 실행되지 않고, 작업 인스턴스가 생성되지 않고 실행되지 않기 때문에 스케쥴링된 워크플로우가 중단되기 때문에 주기적인 확인 필요

<img width="1109" alt="스크린샷 2023-07-13 오후 10 45 33" src="https://github.com/herehyun/airflow/assets/82385436/10f90670-cc57-41e1-b2e6-43e8e214d570">

<div align="center"> &uarr; AIRFLOW SCHEDULER 가 STARTED가 아닌 <mark>STOPPED</mark> 되었을 시</div>

* AIRFLOW -> SUMMARY -> ARIFLOW SCHEDULER 클릭 -> AIRFLOW SCHEDULER ACTION 클릭 -> RESTART 클릭

* 실시간으로 SCHEDULER를 확인하는데는 한계가 있으므로 현재는 airflow 계정안에 1분 간격으로 scheduler가 죽었는지 확인하고 있음
```
[airflow@mn02p] */1 * * * * sh /usr/hmg/2.2.1.0-552/airflow/dags/NGIOS_UTIL/serviceCheckAndRun.sh
```














