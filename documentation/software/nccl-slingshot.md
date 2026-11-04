# NCCL 활용 및 Slingshot 연동

한강 슈퍼컴퓨터에서는 NVIDIA Collective Communications Library(NCCL)를 이용하여 GPU 간 고속 통신을 수행할 수 있습니다. NCCL은 GPU 간 데이터 교환 및 Collective 통신을 지원하는 라이브러리로, PyTorch 등의 AI 프레임워크에서 다중 GPU 및 다중 노드 분산학습에 활용됩니다.

한강에서는 노드 내부의 GPU 간 통신뿐만 아니라 Slingshot 인터커넥트를 이용한 노드 간 통신도 수행할 수 있습니다. 이때 NCCL이 Slingshot을 효율적으로 활용하기 위해서는 적절한 네트워크 플러그인과 통신 라이브러리 구성이 필요합니다.

본 페이지에서는 NCCL의 기본적인 활용 방법과 NCCL Tests를 이용한 통신 성능 측정, Slingshot 연동 환경 및 통신 경로 확인 방법을 설명합니다.

## 가. NCCL 개요

### 1. NCCL 소개 및 주요 통신 기능

NCCL은 NVIDIA GPU 간 Collective 및 Point-to-Point 통신을 지원하는 라이브러리입니다. 단일 노드의 여러 GPU뿐만 아니라 네트워크로 연결된 여러 노드의 GPU 사이에서도 통신을 수행할 수 있습니다.

대표적인 NCCL 통신 연산은 다음과 같습니다.

| 통신 연산         | 설명                               | 주요 활용                  |
| ------------- | -------------------------------- | ---------------------- |
| AllReduce     | 여러 GPU의 데이터를 집계하고 결과를 모든 GPU에 전달 | 데이터 병렬학습의 Gradient 동기화 |
| AllGather     | GPU별로 분할된 데이터를 수집하여 모든 GPU에 전달   | FSDP 등의 분산 파라미터 수집     |
| ReduceScatter | 데이터를 집계한 후 결과를 GPU별로 분산          | FSDP 등의 Gradient 분산    |
| Broadcast     | 특정 GPU의 데이터를 다른 GPU에 전달          | 모델 파라미터 및 데이터 전달       |
| Send/Recv     | GPU 간 Point-to-Point 데이터 전송      | 모델 병렬화 및 데이터 교환        |

NCCL은 GPU 간 통신을 위한 라이브러리로, 일반적인 HPC 프로그램에서 널리 사용하는 MPI와는 지원 범위에 차이가 있습니다.

MPI는 CPU 및 GPU 데이터를 대상으로 다양한 통신 패턴을 지원하는 범용 통신 라이브러리이며, NCCL은 NVIDIA GPU 간 통신에 특화되어 있습니다. 일부 GPU 가속 HPC 애플리케이션에서는 MPI와 NCCL을 함께 사용하기도 합니다.

PyTorch에서는 `torch.distributed`를 통해 NCCL 백엔드를 사용할 수 있으며, DDP 및 FSDP 등의 분산학습에서 GPU 간 통신을 수행합니다.

PyTorch 분산학습의 실제 실행 방법은 「PyTorch 활용」 페이지를 참고하시기 바랍니다.

NCCL의 통신 연산과 API에 대한 자세한 내용은 NVIDIA NCCL 공식 문서를 참고하시기 바랍니다.

### 2. NCCL 실행 환경

NCCL은 CUDA 및 GPU 통신 환경과 연동하여 실행됩니다.

한강에서 NCCL을 사용하는 경우 KISTI 제공 AI 컨테이너를 활용하거나, 사용자 정의 컨테이너 및 Conda 환경에서 필요한 라이브러리를 구성할 수 있습니다.

특히 다중 노드 작업에서는 다음 사항을 확인해야 합니다.

* CUDA 및 NCCL 라이브러리의 호환성
* GPU와 네트워크 장치의 접근 가능 여부
* NCCL 네트워크 플러그인의 설치 및 로딩
* Slingshot 인터커넥트를 활용하는 통신 경로 구성

단일 노드에서 NCCL이 정상적으로 실행되더라도 다중 노드에서 Slingshot 통신이 올바르게 구성되었다고 판단할 수는 없습니다.

따라서 다중 노드 AI 작업에서는 NCCL 실행 여부뿐만 아니라 실제 통신에 사용되는 네트워크 백엔드도 확인하는 것이 좋습니다.

## 나. NCCL 활용

### 1. NCCL 기본 사용 방법

NCCL은 C/C++ API를 통해 직접 사용하거나, PyTorch 등의 AI 프레임워크에서 제공하는 분산통신 기능을 통해 사용할 수 있습니다.

PyTorch에서는 일반적으로 NCCL API를 직접 호출하지 않고 `torch.distributed`를 이용합니다. 반면 GPU 가속 애플리케이션을 직접 개발하는 경우에는 `ncclAllReduce()` 등의 NCCL API를 활용할 수 있습니다.

NCCL을 사용하는 프로그램의 실행 환경을 확인할 때는 다음 사항을 확인하는 것이 좋습니다.

#### 1) GPU 장치 확인

GPU 자원을 할당받은 계산노드에서 다음 명령어로 GPU를 확인합니다.

```
nvidia-smi
```

#### 2) NCCL 버전 확인

PyTorch가 설치된 환경에서는 다음 명령어로 PyTorch에서 사용하는 NCCL 버전을 확인할 수 있습니다.

```
python -c "import torch; print(torch.cuda.nccl.version())"
```

이는 해당 PyTorch 환경에서 사용하는 NCCL 버전을 확인하는 방법이며, 별도로 설치된 NCCL 라이브러리 또는 NCCL Tests에서 사용하는 버전과 다를 수 있습니다.

#### 3) NCCL 로그 활성화

NCCL 실행 과정의 정보를 확인하려면 다음 환경변수를 설정할 수 있습니다.

```
export NCCL_DEBUG=INFO
export NCCL_DEBUG_SUBSYS=INIT,NET
```

이 설정은 NCCL 초기화 및 네트워크 관련 로그를 출력합니다.

로그를 이용하면 NCCL 버전, 통신 초기화 과정, 네트워크 플러그인 로딩 여부 등을 확인할 수 있습니다.

NCCL의 상세 실행 방법과 API는 NVIDIA NCCL User Guide를 참고하시기 바랍니다.

### 2. NCCL Tests 활용

NCCL Tests는 NVIDIA에서 제공하는 통신 성능 및 정확성 검증 도구입니다.

주요 Collective 통신 연산에 대한 실행 시간과 대역폭을 측정할 수 있으며, 단일 노드 및 다중 노드 GPU 통신 환경을 확인하는 데 활용할 수 있습니다.

대표적인 테스트 프로그램은 다음과 같습니다.

| 테스트 프로그램              | 측정 대상            |
| --------------------- | ---------------- |
| `all_reduce_perf`     | AllReduce 통신     |
| `all_gather_perf`     | AllGather 통신     |
| `reduce_scatter_perf` | ReduceScatter 통신 |
| `broadcast_perf`      | Broadcast 통신     |
| `alltoall_perf`       | AllToAll 통신      |

NCCL Tests는 실행 환경에 따라 미리 제공된 바이너리를 사용하거나 직접 빌드하여 사용할 수 있습니다.

소스코드와 빌드 방법은 NVIDIA NCCL Tests GitHub를 참고하시기 바랍니다.

#### 1) 단일 노드 NCCL 테스트

단일 노드에서 GPU 4개를 할당받은 경우 다음과 같이 AllReduce 테스트를 실행할 수 있습니다.

```
module load nvidia/26.3 cuda/12.9_26.3 nccl-tests/2.19.2

unset NCCL_NET
export NCCL_NET_PLUGIN=none

all_reduce_perf \
    -b 8 \
    -e 256M \
    -f 2 \
    -g 4
```

단일 노드 테스트에서는 `aws-ofi-nccl` 모듈을 불러오지 않습니다.

이 모듈은 `NCCL_NET="AWS Libfabric"`을 설정하므로, Slingshot VNI가 할당되지 않는 단일 노드 작업에서는 NCCL 초기화에 실패할 수 있습니다.

사용자 정의 환경에서 OFI 플러그인 관련 설정이 적용되어 있다면 `NCCL_NET_PLUGIN=none`을 통해 외부 네트워크 플러그인 사용을 비활성화할 수 있습니다.

주요 옵션은 다음과 같습니다.

| 옵션   | 설명               |
| ---- | ---------------- |
| `-b` | 최소 메시지 크기        |
| `-e` | 최대 메시지 크기        |
| `-f` | 메시지 크기 증가 배율     |
| `-g` | 프로세스당 사용하는 GPU 수 |

위 명령어는 4개 GPU를 대상으로 메시지 크기를 증가시키면서 AllReduce 통신 성능을 측정합니다.

단일 노드 테스트는 GPU 간 통신이 정상적으로 동작하는지 확인하는 데 사용할 수 있습니다.

단, 노드 간 Slingshot 통신 성능을 검증하려면 별도의 다중 노드 테스트가 필요합니다.

#### 2) 다중 노드 NCCL 테스트

여러 계산노드에 걸친 NCCL 통신을 측정하려면 모듈로 제공되는 MPI 지원 NCCL Tests를 활용합니다.

NCCL Tests는 MPI를 이용하여 여러 프로세스를 초기화하고 실행할 수 있으며, 실제 GPU 간 Collective 통신은 NCCL을 통해 수행합니다.

다음은 2개 노드에서 총 8개 GPU를 사용하는 Slurm 작업 스크립트 예제입니다.

**nccl\_multinode.sh**

```
#!/bin/bash
#SBATCH --job-name=nccl-test
#SBATCH --partition=normal
#SBATCH --comment="field=misc;appl=misc"
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=1
#SBATCH --gpus-per-node=4
#SBATCH --exclusive
#SBATCH --time=00:10:00

module purge
module load nvidia/26.3 cuda/12.9_26.3 nccl-tests/2.19.2 aws-ofi-nccl/1.20.0

# NCCL 로그 설정
export NCCL_DEBUG=INFO
export NCCL_DEBUG_SUBSYS=INIT,NET

# Slingshot CXI 통신 설정
export NCCL_NET="AWS Libfabric"
export FI_PROVIDER=cxi
export NCCL_IB_DISABLE=1

# NCCL 프로토콜 설정
export NCCL_PROTO=^LL128

# PMIx 설정
export PMIX_MCA_psec=native

# NCCL Tests 실행
srun --mpi=pmix --network=disable_rdzv_get \
    --ntasks=2 --ntasks-per-node=1 \
    all_reduce_perf \
    -b 8 -e 256M -f 2 -g 4
```

작업은 다음과 같이 제출할 수 있습니다.

```
sbatch nccl_multinode.sh
```

위 예제에서는 각 노드에서 하나의 테스트 프로세스를 실행하며, 프로세스당 GPU 4개를 사용하여 총 8개 GPU 간 AllReduce 통신을 측정합니다.

주요 통신 설정의 역할은 다음과 같습니다.

| 설정                           | 설명                             |
| ---------------------------- | ------------------------------ |
| `NCCL_NET="AWS Libfabric"`   | AWS OFI NCCL 네트워크 백엔드 선택       |
| `FI_PROVIDER=cxi`            | libfabric의 CXI Provider 선택     |
| `NCCL_IB_DISABLE=1`          | NCCL 기본 InfiniBand 백엔드 사용 비활성화 |
| `NCCL_PROTO=^LL128`          | LL128 프로토콜 비활성화                |
| `PMIX_MCA_psec=native`       | PMIx의 native 보안 컴포넌트 사용        |
| `--mpi=pmix`                 | Slurm의 PMIx 기반 프로세스 초기화        |
| `--network=disable_rdzv_get` | Slurm 네트워크 rendezvous 관련 동작 설정 |

**주의:** 한강에서 수행한 대규모 분산학습 검증 환경에서는 LL128 프로토콜 사용 시 노드 간 AllReduce 연산 결과가 올바르지 않은 현상이 확인되었습니다. 해당 검증 환경에서는 `NCCL_PROTO=^LL128` 설정을 적용해야 합니다.

또한 위 스크립트는 기존 제공 모듈인 `aws-ofi-nccl/1.20.0`을 사용하는 예제입니다. 대규모 Apertus 성능 재현에서는 호스트 libfabric 2.3.1과 호환되도록 구성한 aws-ofi-nccl 1.22.0 환경을 사용했습니다. 두 환경의 호환성과 성능이 동일하다고 보장되는 것은 아니므로 실제 사용 환경에 맞게 확인해야 합니다.

참고: 위 스크립트는 한강에서 확인한 실행 옵션을 반영한 예제이며, GPU 할당 정책, MPI 초기화 방식, 제공 NCCL Tests 경로 및 소프트웨어 버전에 따라 수정이 필요할 수 있습니다.

컨테이너에서 NCCL Tests를 실행하는 경우에는 MPI 및 NCCL 라이브러리뿐만 아니라 OFI 플러그인과 호스트 libfabric/CXI 라이브러리의 호환성도 확인해야 합니다.

### 3. 통신 성능 측정

NCCL Tests를 실행하면 메시지 크기별 통신 시간과 대역폭이 출력됩니다.

대표적인 출력 항목은 다음과 같습니다.

| 항목      | 설명                                      |
| ------- | --------------------------------------- |
| `size`  | 통신 메시지 크기(Byte)                         |
| `time`  | 통신 연산 수행 시간(μs)                         |
| `algbw` | 메시지 크기와 수행 시간으로 계산한 Algorithm Bandwidth |
| `busbw` | Collective별 통신량을 고려하여 환산한 Bus Bandwidth |
| `error` | 통신 결과의 정확성 검증 결과                        |

`algbw`는 메시지 크기를 수행 시간으로 나눈 값이며, `busbw`는 Collective 통신 패턴에 따른 보정 계수를 적용한 값입니다.

예를 들어 AllReduce에서는 GPU 수가 N개일 때 다음과 같이 Bus Bandwidth를 계산합니다.

```
busbw = algbw × 2(N-1)/N
```

`busbw`는 NCCL Tests에서 정의한 성능 지표로, 실제 네트워크 인터페이스의 송수신 대역폭을 직접 측정한 값과 동일하지는 않습니다.

또한 단일 노드와 다중 노드에서는 통신에 사용되는 하드웨어가 다르므로 측정 결과를 구분하여 해석해야 합니다.

* **단일 노드:** 주로 GPU 간 NVLink 등의 통신 성능 확인
* **다중 노드:** 노드 내부 GPU 통신과 노드 간 네트워크 통신이 결합된 성능 확인

성능 측정 시에는 GPU 수, 노드 수, 메시지 크기, NCCL 버전 및 네트워크 설정 등을 함께 기록하는 것이 좋습니다.

특히 다중 노드 통신에서는 성능 측정값뿐만 아니라 NCCL Tests의 정확성 검증 결과도 함께 확인해야 합니다.

`busbw` 계산에 대한 상세 내용은 NCCL Tests Performance 문서를 참고하시기 바랍니다.

## 다. Slingshot 환경에서의 NCCL 활용

### 1. NCCL과 Slingshot 통신 환경

한강 슈퍼컴퓨터에서는 노드 간 고속 통신을 위해 Slingshot 인터커넥트를 사용합니다.

NCCL이 Slingshot을 이용하여 노드 간 GPU 통신을 수행하려면, Slingshot 네트워크와 연동되는 적절한 통신 백엔드가 필요합니다.

Slingshot 환경에서는 aws-ofi-nccl과 같은 네트워크 플러그인을 통해 NCCL과 libfabric을 연동할 수 있습니다. 한강에서는 제공되는 통신 소프트웨어 환경에 따라 적절한 네트워크 플러그인을 활용해야 합니다.

일반적인 통신 소프트웨어 구성은 다음과 같습니다.

```
PyTorch / GPU Application
           |
          NCCL
           |
      aws-ofi-nccl
           |
       libfabric
      (CXI Provider)
           |
     Slingshot Network
```

aws-ofi-nccl은 NCCL이 libfabric을 네트워크 통신 계층으로 사용할 수 있도록 지원하는 플러그인입니다.

Slingshot 환경에서는 libfabric의 CXI Provider를 통해 네트워크 장치와 연동할 수 있습니다.

이러한 구성을 통해 지원되는 환경에서는 GPUDirect RDMA 기반 GPU 간 통신을 활용할 수 있습니다.

다만 실제 통신 방식은 GPU와 네트워크 장치의 구성, 라이브러리 버전 및 실행 환경 설정에 따라 달라질 수 있습니다.

### 2. NCCL 네트워크 플러그인

NCCL은 실행 환경에 따라 내부 네트워크 백엔드 또는 외부 네트워크 플러그인을 사용할 수 있습니다.

Slingshot 기반 시스템에서는 aws-ofi-nccl과 같은 플러그인을 이용하여 libfabric 기반 통신을 수행할 수 있습니다.

#### 1) KISTI 제공 AI 환경

KISTI 제공 AI 컨테이너는 한강의 GPU 및 Slingshot 환경을 고려하여 구성된 실행 환경입니다.

다중 노드 NCCL 작업에서는 제공 환경에서 지원하는 네트워크 플러그인과 통신 설정을 활용하는 것을 권장합니다.

사용자는 컨테이너 실행 후 NCCL 로그를 이용하여 실제 사용되는 네트워크 백엔드를 확인할 수 있습니다.

호스트에서 실행하는 경우 다음 모듈을 사용할 수 있습니다.

```
module load aws-ofi-nccl/1.20.0
```

이 모듈은 `NCCL_NET`, `NCCL_SOCKET_IFNAME=hsn0,hsn1,hsn2,hsn3`, `FI_CXI_*` 등의 기본값을 함께 설정합니다.

환경변수는 모듈 로딩 시 설정되는 기본값을 우선 사용하며, 개별 값을 변경할 경우 실제 NCCL 통신 경로 및 정상 동작 여부를 확인해야 합니다.

#### 2) 사용자 정의 AI 환경

사용자 정의 컨테이너 또는 Conda 환경에서는 필요한 네트워크 플러그인과 관련 라이브러리가 올바르게 연동되어야 합니다.

기존 SIF 또는 SQSH 컨테이너 이미지를 사용하는 경우, `kisti-container`를 이용하여 지원되는 범위 내에서 한강의 통신 환경과 연동할 수 있습니다.

직접 컨테이너를 구성하는 경우에는 NCCL, 네트워크 플러그인, libfabric 및 CUDA 라이브러리 간 호환성을 고려해야 합니다.

특히 NGC 등 외부에서 제공하는 컨테이너는 기본 설정만으로 Slingshot CXI 경로가 선택되지 않을 수 있습니다.

다중 노드 작업에서는 다음 설정을 이용하여 통신 백엔드를 명시적으로 선택할 수 있습니다.

```
export NCCL_NET="AWS Libfabric"
export FI_PROVIDER=cxi
export NCCL_IB_DISABLE=1
```

단, 해당 설정만으로 호환되는 OFI 플러그인이 설치되는 것은 아닙니다. 컨테이너 내부의 NCCL과 호스트의 libfabric/CXI 라이브러리에 맞는 플러그인이 필요합니다.

구체적인 구성 방법은 「사용자 정의 AI 환경 구성」 페이지를 참고하시기 바랍니다.

### 3. Slingshot 통신 경로 확인

NCCL 프로그램이 정상적으로 실행되더라도 실제 노드 간 통신에 Slingshot의 고속 통신 경로가 사용되고 있는지 확인하는 것이 중요합니다.

다음 환경변수를 설정하여 NCCL의 네트워크 초기화 정보를 확인할 수 있습니다.

```
export NCCL_DEBUG=INFO
export NCCL_DEBUG_SUBSYS=INIT,NET
```

이후 다중 노드 NCCL 프로그램 또는 NCCL Tests를 실행합니다.

aws-ofi-nccl 플러그인을 사용하는 환경에서는 정상적인 네트워크 선택 시 다음과 같은 로그가 출력될 수 있습니다.

```
NCCL INFO Using network AWS Libfabric
```

이는 NCCL이 AWS Libfabric 네트워크 백엔드를 선택했음을 나타냅니다.

반면 다음과 같은 로그가 출력된다면 해당 환경에서 다른 네트워크 백엔드를 사용하고 있을 수 있습니다.

```
NCCL INFO Using network Socket
```

다중 노드에서 Socket 백엔드가 사용된다면 고속 통신 플러그인이 정상적으로 로드되지 않았거나 다른 네트워크 경로가 선택되었을 가능성을 확인해야 합니다.

단, 단일 노드 테스트에서는 노드 간 데이터 통신이 발생하지 않으므로 이러한 로그만으로 Slingshot 활용 여부를 판단할 수 없습니다.

또한 초기 연결에 사용되는 Bootstrap Socket과 실제 Collective 데이터 전송에 사용되는 네트워크 백엔드는 구분하여 확인해야 합니다.

NCCL에서 특정 네트워크 백엔드를 강제로 선택하려면 `NCCL_NET` 환경변수를 사용할 수 있습니다.

예를 들어 aws-ofi-nccl 플러그인이 제공되는 환경에서는 다음과 같이 설정할 수 있습니다.

```
export NCCL_NET="AWS Libfabric"
```

이 설정은 NCCL이 지정된 백엔드를 사용하도록 강제합니다.

다만 해당 플러그인이 설치되어 있지 않거나 사용할 수 없는 경우 프로그램이 실행되지 않을 수 있으므로, 실제 지원 환경을 확인한 후 적용해야 합니다.

또한 단일 노드 작업에서는 이 설정을 사용하지 않습니다.

### 4. 통신량이 많은 작업의 노드 배치

한강은 Slingshot의 Non-blocking Dragonfly Topology로 구성되어 있으며, GPU 계산노드는 대부분 104노드 단위의 네트워크 그룹으로 구분됩니다.

Dragonfly Topology는 그룹 내 통신과 그룹 간 통신에 서로 다른 링크 구조를 사용하도록 설계되어 있습니다. 이러한 네트워크 구성 특성으로 인해 **한강에서 측정한 그룹 간 통신 대역폭은 그룹 내 통신보다 약 15\~20% 낮게 나타났으며**, MoE 학습 등 노드 간 통신량이 많은 AI 작업의 성능에 영향을 줄 수 있습니다.

104노드 이하 작업에서는 다음 Slurm 옵션으로 동일 네트워크 그룹 내 배치를 우선 요청할 수 있습니다.

```
#SBATCH --switches=1@02:00:00
```

위 옵션은 최대 2시간 동안 동일 그룹 내 노드 배치를 기다리며, 지정한 시간이 지나면 여러 그룹에 분산 배치될 수 있습니다.

작업 대기 시간이 증가할 수 있으므로 통신량이 많은 작업에 선택적으로 사용하는 것을 권장합니다.

## 라. 주요 환경 설정 및 문제 해결

### 1. NCCL 환경변수

NCCL에서는 환경변수를 이용하여 네트워크 백엔드, 디버깅 정보 및 통신 동작을 제어할 수 있습니다.

대표적인 NCCL 및 관련 통신 환경변수는 다음과 같습니다.

| 환경변수                 | 설명                                                |
| -------------------- | ------------------------------------------------- |
| `NCCL_DEBUG`         | NCCL 로그 출력 수준 설정                                  |
| `NCCL_DEBUG_SUBSYS`  | 로그를 출력할 NCCL 하위 시스템 지정                            |
| `NCCL_NET`           | 사용할 NCCL 네트워크 백엔드 지정                              |
| `NCCL_NET_PLUGIN`    | NCCL 외부 네트워크 플러그인 선택. `none` 지정 시 외부 플러그인 사용 비활성화 |
| `NCCL_SOCKET_IFNAME` | NCCL Socket 통신에 사용할 네트워크 인터페이스 선택                 |
| `NCCL_NET_GDR_LEVEL` | GPU와 NIC 사이의 GPUDirect RDMA 사용 범위 설정              |
| `NCCL_CROSS_NIC`     | NIC 간 경로 사용 방식에 영향을 주는 설정                         |
| `NCCL_IB_DISABLE`    | NCCL 기본 InfiniBand 백엔드 사용 여부 설정                   |
| `NCCL_PROTO`         | NCCL 통신 프로토콜 선택 또는 비활성화                           |
| `FI_PROVIDER`        | libfabric에서 사용할 Provider 지정                       |
| `PMIX_MCA_psec`      | PMIx 보안 컴포넌트 설정                                   |

한강에서 검증된 다중 노드 실행 환경에서는 다음 설정을 사용할 수 있습니다.

```
export NCCL_NET="AWS Libfabric"
export FI_PROVIDER=cxi
export NCCL_IB_DISABLE=1
export NCCL_PROTO=^LL128
```

위 설정은 Slingshot CXI 통신 경로를 사용하고 LL128 프로토콜을 비활성화하기 위한 것입니다. 실제 실행에는 호환되는 NCCL OFI 플러그인이 필요합니다.

**주의: NCCL 통신 결과 정확성**

한강에서 수행한 대규모 분산학습 검증 환경에서는 NCCL LL128 프로토콜 사용 시 노드 간 AllReduce 결과가 올바르지 않은 현상이 확인되었습니다.

해당 검증 환경에서 NCCL을 사용하는 경우 다음 환경변수를 적용해야 합니다.

```
export NCCL_PROTO=^LL128
```

이 설정은 LL128 프로토콜을 비활성화하며, 다른 NCCL 통신 프로토콜의 사용까지 금지하는 것은 아닙니다.

해당 현상은 검증 당시의 소프트웨어 및 시스템 구성을 기준으로 하며, 다른 NCCL 버전이나 실행 환경에서의 발생 여부는 별도 확인이 필요합니다.

통신 문제를 확인할 때는 우선 다음과 같이 로그 출력을 활성화하는 것이 좋습니다.

```
export NCCL_DEBUG=INFO
export NCCL_DEBUG_SUBSYS=INIT,NET
```

필요한 경우 상세 로그를 파일로 저장할 수 있습니다.

```
export NCCL_DEBUG_FILE="nccl.%h.%p.log"
```

파일명에 사용되는 `%h`와 `%p`는 각각 호스트 이름과 프로세스 ID로 대체됩니다.

다중 노드 작업에서는 프로세스별 로그를 구분하여 수집하는 것이 문제 분석에 도움이 됩니다.

NCCL 및 libfabric 관련 환경변수는 시스템 구성과 라이브러리 버전에 따라 적절한 설정값이 달라질 수 있습니다.

따라서 일반적인 성능 최적화 설정을 임의로 적용하기보다는 KISTI 제공 환경의 기본 설정을 우선 사용하는 것을 권장합니다.

환경변수별 상세 설명은 NVIDIA NCCL Environment Variables를 참고하시기 바랍니다.

### 2. 통신 오류 및 성능 문제 확인

NCCL 실행 과정에서 오류가 발생하거나 예상보다 낮은 통신 성능이 나타나는 경우에는 다음 항목을 확인할 수 있습니다.

| 증상               | 확인 사항                                    |
| ---------------- | ---------------------------------------- |
| NCCL 초기화 실패      | NCCL 및 CUDA 라이브러리 호환성, GPU 할당 상태         |
| 네트워크 플러그인 로딩 실패  | 플러그인 설치 여부, 라이브러리 경로, 의존성                |
| 다중 노드 통신 실패      | Slurm 작업 구성, PMIx 초기화, 네트워크 접근, 프로세스 초기화 |
| Socket 백엔드 사용    | 외부 네트워크 플러그인 로딩 및 선택 여부                  |
| 통신 성능 저하         | 실제 통신 경로, GPU/NIC 배치, NCCL 설정            |
| 통신 중단 또는 Timeout | 프로세스별 로그, Collective 호출 순서, 네트워크 상태      |
| AllReduce 결과 불일치 | NCCL 프로토콜 설정, NCCL Tests 정확성 검증 결과       |

#### 1) 네트워크 플러그인 로딩 오류

다음과 같은 메시지가 출력되는 경우가 있습니다.

```
NCCL WARN Failed to initialize any NET plugin
```

이는 `NCCL_NET="AWS Libfabric"`이 설정되어 있지만 플러그인을 불러오지 못했거나, 단일 노드 작업에서 필요한 네트워크 자원이 할당되지 않아 플러그인 초기화에 실패한 경우 등에 발생할 수 있습니다.

이때는 플러그인 설치 여부와 라이브러리 검색 경로, 컨테이너 내부의 통신 환경 구성을 확인해야 합니다.

단일 노드 작업에서 외부 OFI 플러그인 설정이 남아 있는 경우에는 다음과 같이 설정할 수 있습니다.

```
unset NCCL_NET
export NCCL_NET_PLUGIN=none
```

#### 2) 다중 노드 통신 성능 저하

다중 노드 NCCL 작업의 통신 성능이 낮은 경우에는 우선 실제 통신에 사용되는 네트워크 백엔드를 확인해야 합니다.

특히 NCCL이 Socket 기반 데이터 통신을 사용하고 있다면 Slingshot과 연동된 고속 통신 백엔드를 사용할 수 있도록 환경을 확인해야 합니다.

NCCL Tests를 이용하여 단일 노드와 다중 노드의 성능을 비교하면 통신 성능 저하의 원인을 파악하는 데 도움이 될 수 있습니다.

다만 GPU 수와 메시지 크기 등에 따라 성능이 달라질 수 있으므로 동일한 조건에서 비교해야 합니다.

#### 3) NCCL 통신 결과 불일치

NCCL 프로그램이 정상적으로 종료되더라도 Collective 연산 결과가 올바르지 않을 수 있으므로, 통신 성능뿐만 아니라 결과의 정확성도 확인해야 합니다.

특히 한강의 검증 환경에서 확인된 LL128 문제를 방지하기 위해 다음 설정을 사용할 수 있습니다.

```
export NCCL_PROTO=^LL128
```

NCCL Tests의 `error` 항목을 확인하여 통신 결과의 정확성을 검증하시기 바랍니다.

#### 4) NCCL과 GPU-aware MPI의 동시 사용

일부 GPU 가속 HPC 애플리케이션에서는 NCCL과 GPU-aware MPI를 함께 사용하기도 합니다.

이 경우 GPU 통신 및 동기화 과정에서 두 라이브러리의 통신 진행 조건이 충돌하면 통신 중단이나 Deadlock이 발생할 수 있으므로 주의해야 합니다.

특히 GPU-aware MPI와 NCCL을 동시에 사용하는 프로그램에서는 통신 연산의 실행 순서와 동기화 조건을 확인해야 합니다.

#### 5) 문제 해결을 위한 정보 수집

문제가 지속되는 경우 다음 정보를 함께 확인하는 것이 좋습니다.

* 사용한 AI 실행 환경 및 컨테이너 이미지
* CUDA 및 NCCL 버전
* 사용한 노드 수와 GPU 수
* 실행 명령어 및 Slurm 작업 스크립트
* NCCL 로그 및 오류 메시지
* NCCL Tests 결과
* NCCL 통신 프로토콜 및 네트워크 백엔드 설정

이러한 정보를 함께 기록하면 환경 구성 문제와 통신 성능 문제를 구분하는 데 도움이 됩니다.

자세한 내용은 「문제 해결 및 운영 정보」 페이지를 참고하시기 바랍니다.

## 참고 자료

* [NVIDIA NCCL User Guide](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/index.html)
* [NVIDIA NCCL Tests](https://github.com/NVIDIA/nccl-tests)
* [AWS OFI NCCL Plugin](https://github.com/aws/aws-ofi-nccl)
