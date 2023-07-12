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
&darr; 


























