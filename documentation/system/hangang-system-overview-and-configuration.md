# 시스템 개요 및 구성

## 가. 국가슈퍼컴퓨터 6호기

### 1. HPE Cray EX4000 기반 슈퍼컴퓨터

HANGANG은 모델링, 시뮬레이션, AI/ML 워크로드를 통합 지원하는 차세대 국가 플래그십 슈퍼컴퓨터입니다. GH200 기반 GPU 파티션, H200 GPU-Fat 노드, AMD Turin 기반 CPU 파티션, 205PB ClusterStor 스토리지와 HPE Slingshot 인터커넥트를 함께 제공합니다.

## 나. 시스템 구성

### 1. 계산노드

#### 1) GPU Partition

<table data-search="false"><thead><tr><th width="272.28466796875" align="center" valign="middle">구분</th><th align="center">내용</th></tr></thead><tbody><tr><td align="center" valign="middle">모델</td><td align="center">HPE Cray EX4000 MPP</td></tr><tr><td align="center" valign="middle">운영체제</td><td align="center">Cray OS (SUSE)</td></tr><tr><td align="center" valign="middle">노드수</td><td align="center">2,084개 노드 (1노드당 4개 GH200 SuperChip 탑재)</td></tr><tr><td align="center" valign="middle">CPU</td><td align="center">Arm Grace CPU / 72-core 3.1GHz / 4 CPUs per node</td></tr><tr><td align="center" valign="middle">GPU</td><td align="center">NVIDIA Hopper GPU / 4 GPUs per node (총 8,336 GPUs)</td></tr><tr><td align="center" valign="middle">메인 메모리</td><td align="center">노드 당 LPDDR5X 480GB + HBM3 384GB</td></tr><tr><td align="center" valign="middle">인터커넥트</td><td align="center">노드 당 1,600Gbps (400Gbps/GPU)</td></tr><tr><td align="center" valign="middle">이론 성능</td><td align="center">588.28PF (FP64, Peak)</td></tr></tbody></table>

#### 2) CPU Partition

<table data-search="false"><thead><tr><th align="center" valign="middle">구분</th><th align="center" valign="middle">내용</th></tr></thead><tbody><tr><td align="center" valign="middle">모델</td><td align="center" valign="middle">HPE Cray EX4000 MPP</td></tr><tr><td align="center" valign="middle">운영체제</td><td align="center" valign="middle">Cray OS (SUSE)</td></tr><tr><td align="center" valign="middle">노드수</td><td align="center" valign="middle">800개 노드</td></tr><tr><td align="center" valign="middle">CPU</td><td align="center" valign="middle">AMD EPYC 9745 / 2 CPUs per node / 256 CPU cores per node (총 1,600 CPUs)</td></tr><tr><td align="center" valign="middle">메인 메모리</td><td align="center" valign="middle">노드 당 DDR5 768GB</td></tr><tr><td align="center" valign="middle">인터커넥트</td><td align="center" valign="middle">노드 당 400Gbps</td></tr><tr><td align="center" valign="middle">이론 성능</td><td align="center" valign="middle">15.72PF (FP64, Peak)</td></tr></tbody></table>

#### 3) GPU-Fat 노드

<table data-search="false"><thead><tr><th align="center">구분</th><th align="center">내용</th></tr></thead><tbody><tr><td align="center">모델</td><td align="center">HPE Cray XD685 GPU 클러스터</td></tr><tr><td align="center">운영체제</td><td align="center">Cray OS (SUSE)</td></tr><tr><td align="center">노드수</td><td align="center">20개 노드 (1노드당 8 H200 GPU 탑재)</td></tr><tr><td align="center">CPU</td><td align="center">AMD EPYC 9355 / 2 CPUs per node / 64 CPU cores per node (총 40 CPUs)</td></tr><tr><td align="center">GPU</td><td align="center">NVIDIA H200 / 8 GPUs per node (총 160 GPUs)</td></tr><tr><td align="center">메인 메모리</td><td align="center">노드 당 DDR5 1,536GB + HBM3e 1,128GB</td></tr><tr><td align="center">인터커넥트</td><td align="center">노드 당 1,600Gbps</td></tr><tr><td align="center">이론 성능</td><td align="center">10.72PF (FP64, Peak)</td></tr></tbody></table>

#### 4) 대용량 메모리 노드

|   구분   |              내용             |
| :----: | :-------------------------: |
|   노드수  |            2개 노드            |
|   CPU  | Intel SPR 32 CPUs x 2 nodes |
| 메인 메모리 |       노드 당 48TB Memory      |

## 다. 주요 구성

### 1. GPU 계산노드

노드당 4개의 GH200 SuperChip을 탑재하며, 각 GH200은 Arm Grace CPU 72Core 3.1GHz와 NVIDIA Hopper GPU로 구성됩니다.

GH200당 LPDDR5X 120GB와 HBM3 96GB를 제공하고, CPU-GPU NVLink 900GB/s 및 GPU-GPU NVLink 150GB/s per link를 지원합니다.

### 2. 인터커넥션 네트워크

HPE Slingshot 기반 400Gbps/GPU 네트워크와 Non-blocking Dragonfly Topology를 적용합니다.

GPU 노드당 400Gbps 4포트 구성을 기반으로 파티션 간 대규모 병렬 통신과 파일 I/O 트래픽을 지원합니다.

### 3. 직접수냉식 냉각

9MW급 시스템을 대상으로 liquid-to-air-cool ratio 96:4, inlet temperature 26℃ 기준의 직접수냉식 냉각 구조를 적용합니다.

Primary side는 Facility water와 CDU, Secondary side는 CDU와 Compute 장비 사이에서 PGW를 활용해 순환합니다.

## 라. 스토리지 및 데이터 환경

한강 시스템은 고성능 스토리지 HPE Cray CS-E2000에 Lustre 병렬 파일시스템을 적용하여 HPC 시뮬레이션과 AI 학습·추론 작업을 모두 지원합니다. Flash와 HDD를 결합한 계층형(Tiered) 스토리지와 분산 메타데이터 구조를 통해, HPC의 대규모 병렬 입출력뿐 아니라 AI 환경에서 요구되는 작은 데이터 단위의 무작위 접근(Small Random Access)과 다수 파일의 동시 접근을 지원합니다.

#### 1) Lustre 병렬 파일시스템 구성

한강 시스템은 `/home01`, `/apps`, `/appsdata`, `/scratch`  4개의  병렬 파일시스템을 제공합니다. 전체 데이터 용량은 약 206.94 PiB이며, 총 648개의 OST(Object Storage Target)로 구성됩니다.

`/home01`과 `/apps`는 HDD 기반으로 구성되며, `/appsdata`와 `/scratch`는 Flash와 HDD 계층을 함께 제공합니다. 파일시스템별 데이터 용량과 OST 구성은 다음과 같습니다.

| 항목          | /home01  | /apps    | /appsdata | /scratch   |
| ----------- | -------- | -------- | --------- | ---------- |
| 파일시스템 이름    | home     | apps     | appsdata  | scratch    |
| 전체 데이터 용량   | 5.67 PiB | 5.67 PiB | 88.30 PiB | 107.30 PiB |
| Flash 용량    | —        | —        | 8.94 PiB  | 10.94 PiB  |
| HDD 용량      | 5.67 PiB | 5.67 PiB | 79.35 PiB | 96.36 PiB  |
| 전체 OST 수    | 8        | 8        | 224       | 408        |
| Flash OST 수 | 0        | 0        | 112       | 272        |
| HDD OST 수   | 8        | 8        | 112       | 136        |

용량은 구성 사양 기준이며, 반올림으로 인해 세부 용량의 합과 전체 용량에 차이가 있을 수 있습니다. PiB와 TiB는 각각 2⁵⁰바이트와 2⁴⁰바이트를 의미합니다.

#### 2) 스토리지 서버 및 고가용성 구성

메타데이터 처리는 MDS/MGS 노드가 담당하고, 파일 데이터 입출력은 OSS(Object Storage Server) 노드가 담당합니다. 전체 파일시스템은 MDS/MGS 28대와 OSS 196대로 구성되며, 각각 HA Pair를 구성합니다.

| 항목         | /home01 | /apps | /appsdata | /scratch |
| ---------- | ------- | ----- | --------- | -------- |
| MDS/MGS 노드 | 2       | 2     | 12        | 12       |
| OSS 노드     | 2       | 2     | 56        | 136      |

`/scratch`는 OSS 136대가 노드당 3개의 OST를 담당하여 총 408개의 OST를 제공합니다. 이 중 Flash OST는 272개, HDD OST는 136개입니다.

#### 3) 메타데이터 및 데이터 보호

파일과 디렉터리의 메타데이터를 저장하는 MDT(Metadata Target)는 전체 56개이며, MDU 14개에 MDU당 4개씩 구성됩니다.

| 항목         | /home01        | /apps          | /appsdata       | /scratch        |
| ---------- | -------------- | -------------- | --------------- | --------------- |
| MDU 수      | 1              | 1              | 6               | 6               |
| MDT 수      | 4              | 4              | 24              | 24              |
| MDT 전체 용량  | 70.18 TiB      | 70.18 TiB      | 421.10 TiB      | 421.10 TiB      |
| 전체 inode 수 | 17,177,164,320 | 17,177,164,320 | 103,062,985,920 | 103,062,985,920 |

MDT당 용량은 약 17.55 TiB이며, MDT당 inode 수는 4,294,291,080개입니다. 전체(/scratch+/appsdata) inode 수는 240,480,300,480개입니다. 메타데이터 용량은 앞서 제시한 데이터 용량과 구분합니다.

#### 4) Lustre Pool 구성

계층형(Tiering) 기반의 Lustre Pool은 OST를 논리적으로 묶어 관리하는 구성입니다. `/appsdata와` `/scratch`에는 기본적으로 계층에 따라 2개의 Pool이 구성되어 있습니다.

`/scratch, /appsdata`의 Pool은 다음과 같습니다.

| Pool 이름          | 구분       |
| ---------------- | -------- |
| `appsdata.flash` | Flash 계층 |
| `appsdata.disk`  | HDD 계층   |

#### 5) Pool 구성 확인

Pool의 실제 OST 구성은 다음 명령으로 확인할 수 있습니다.

```
lfs pool_list appsdata.flash
lfs pool_list appsdata.disk

lfs pool_list scratch.flash
lfs pool_list scratch.disk
```

#### 6) 파일시스템별 용도 및 할당량

한강 시스템은 작업 특성에 따라 `/scratch`와 `/appsdata`를 구분하여 사용합니다. 두 파일시스템 모두 Flash와 HDD 계층을 제공하며, 각 용도에 맞는 데이터 배치 정책을 적용합니다.

<table data-header-hidden><thead><tr><th width="158"></th><th width="206"></th><th width="196"></th><th width="199"></th></tr></thead><tbody><tr><td>구분</td><td><code>/home01</code></td><td><code>/scratch</code></td><td><code>/appsdata</code></td></tr><tr><td>권장 용도</td><td>사용자 기본 디렉터리</td><td>AI 기반 응용 프로그램 및 데이터 입출력이 집중되는 작업</td><td>전통적인 HPC 시뮬레이션 및 대용량 데이터 처리 작업</td></tr><tr><td>주요 활용</td><td>사용자의 기본환경 설정 및 코드 컴파일, 작업제출</td><td>학습 데이터의 반복 읽기, 전처리, 학습 중 체크포인트 저장</td><td>대용량 입력 데이터의 순차 읽기, 계산 결과 저장</td></tr><tr><td>용량 한도</td><td>64 GB (user)</td><td>80 TB (group)</td><td>20 TB (group)</td></tr><tr><td>그룹별 파일 수 한도</td><td>50만 개</td><td>500만 개</td><td>500만 개</td></tr><tr><td>자동 삭제 기준</td><td>해당 없음</td><td>마지막 접근 후 15일 경과</td><td>-</td></tr><tr><td>데이터 보유 기간</td><td>계정 만료 후 6개월(180일)</td><td>해당 없음</td><td>해당 없음</td></tr><tr><td></td><td></td><td></td><td></td></tr></tbody></table>

할당량은 그룹 구성원이 공동으로 사용하는 한도입니다. 용량 또는 파일 수 중 하나라도 한도에 도달하면 파일 생성이나 데이터 저장이 제한됩니다.\
\* 추가 쿼터 요구사항에 대해서는 별도의  요금 정책에 따라 사용량 단위로 과금 됩니다.

#### 7) 작업별 사용 가이드

* **AI 작업:** `/scratch` 사용을 우선 권장합니다. 학습 데이터를 반복적으로 읽거나 다수 GPU가 동시에 입출력하는 작업에 맞춰 활용합니다.
* **전통 HPC 작업:** `/appsdata` 사용을 우선 권장합니다. 대용량 데이터를 순차적으로 읽고 쓰는 시뮬레이션과 계산 작업에 활용합니다.

위 구분은 권장 용도이며, 실제 성능은 파일 크기, 접근 방식, 데이터 배치 및 시스템 부하에 따라 달라집니다. 작은 파일을 과도하게 생성하지 않도록 하고, 가능한 경우 데이터를 적절한 크기의 파일로 묶어 사용합니다.

#### 8) 데이터 자동 삭제 정책

/scratch 파일시스템은 **마지막 접근 시각(Access Time)을 기준으로 15일 동안 접근하지 않은 데이터를 자동으로 삭제하는 퍼지(purge) 정책**을 적용합니다.

`/scratch`는 영구 보관 공간이 아닙니다. 재사용할 입력 데이터, 최종 결과물 및 보존이 필요한 체크포인트는 삭제 전에 별도의 보관 공간으로 복사해야 합니다. 장기간 실행하는 작업도 실행 중이라는 이유만으로 모든 파일이 삭제 대상에서 제외되는 것은 아니므로 주의해야 합니다.



## 마. 사용자 환경

<table><thead><tr><th width="180.96044921875" align="center">구분</th><th align="center">내용</th></tr></thead><tbody><tr><td align="center">운영체제</td><td align="center">Cray OS (SUSE)</td></tr><tr><td align="center">스케줄러</td><td align="center">Slurm</td></tr><tr><td align="center">컴파일러</td><td align="center">HPE Compile Environment, Intel oneAPI, AMD AOCC compiler</td></tr><tr><td align="center">디버깅·프로파일링</td><td align="center">NVIDIA Nsight, Cray Comparative Debugger, Cray Programming Environment</td></tr><tr><td align="center">라이브러리·프레임워크</td><td align="center">HPE Cray MPI, OpenMPI, NVIDIA HPC SDK, PyTorch, Quantum Interface Library</td></tr><tr><td align="center">웹 서비스</td><td align="center">MyKSC 기반 VS Code, RStudio, JupyterLab, File Browser, Web Terminal, VNC, Batch Job launch</td></tr></tbody></table>

