# AI 프레임워크

한강 슈퍼컴퓨터에서는 PyTorch를 비롯하여 NVIDIA NeMo, Megatron 및 DeepSpeed 등 다양한 AI 프레임워크를 활용할 수 있습니다.

이러한 프레임워크는 대규모 AI 모델의 학습을 위한 데이터 병렬화, 모델 병렬화 및 GPU 메모리 최적화 기능을 제공하며, 여러 노드의 GPU 자원을 활용하는 분산학습에 사용됩니다.

본 페이지에서는 주요 AI 프레임워크의 특징과 한강에서의 기본적인 실행 방법을 소개합니다. 각 프레임워크의 상세한 모델 구성 및 학습 방법은 공식 문서를 참고하시기 바랍니다.

## 가. 주요 AI 프레임워크

대규모 AI 모델 학습에서는 모델 크기, GPU 메모리 요구량 및 병렬화 방식에 따라 적절한 프레임워크를 선택할 수 있습니다.

대표적인 프레임워크는 다음과 같습니다.

| 프레임워크                   | 주요 특징                           | 활용 분야                                |
| ----------------------- | ------------------------------- | ------------------------------------ |
| NVIDIA NeMo             | AI 모델 개발 및 학습을 위한 프레임워크         | 대규모 언어모델 학습 및 미세조정                   |
| Megatron                | 대규모 Transformer 모델을 위한 고성능 분산학습 | Tensor/Pipeline Parallelism 기반 모델 학습 |
| DeepSpeed               | ZeRO 등을 이용한 메모리 및 분산학습 최적화      | 대규모 모델 학습 및 메모리 최적화                  |
| Hugging Face Accelerate | 여러 분산학습 백엔드를 활용하기 위한 실행 인터페이스   | PyTorch 분산학습 및 FSDP/DeepSpeed 활용     |

NeMo/Megatron은 대규모 Transformer 모델의 학습과 다양한 모델 병렬화에 활용할 수 있습니다.

DeepSpeed는 ZeRO(Zero Redundancy Optimizer)를 통해 학습에 필요한 GPU 메모리 사용량을 줄이는 기능을 제공하며, 대규모 모델 학습에 활용됩니다.

이러한 프레임워크는 PyTorch를 기반으로 동작하는 경우가 많으며, GPU 간 통신에 NCCL을 활용할 수 있습니다.

분산학습의 기본적인 실행 방법은 「PyTorch 활용」 페이지를, NCCL 통신 환경은 「NCCL 활용 및 Slingshot 연동」 페이지를 참고하시기 바랍니다.

## 나. NVIDIA NeMo 및 Megatron 활용

### 1. NeMo 및 Megatron 소개

NVIDIA NeMo는 대규모 AI 모델의 학습과 미세조정 등을 지원하는 프레임워크입니다.

Megatron은 대규모 Transformer 모델을 효율적으로 학습하기 위한 병렬화 및 최적화 기능을 제공합니다.

주요 구성 요소는 다음과 같습니다.

| 구성 요소           | 역할                                                |
| --------------- | ------------------------------------------------- |
| NeMo Framework  | AI 모델 학습 및 미세조정을 위한 프레임워크                         |
| Megatron Core   | 대규모 Transformer 학습을 위한 병렬화 및 최적화 기능               |
| Megatron Bridge | Megatron Core 기반 학습 구성 및 Hugging Face 모델·체크포인트 연동 |

Megatron 기반 학습에서는 데이터 병렬화(DP) 외에도 Tensor Parallelism(TP), Pipeline Parallelism(PP), Context Parallelism(CP), Expert Parallelism(EP) 등을 활용할 수 있습니다.

여러 병렬화 방식을 조합하여 모델 크기와 GPU 자원에 적합한 학습 환경을 구성할 수 있습니다.

다만 NeMo 및 Megatron의 구성과 API는 소프트웨어 버전에 따라 달라질 수 있으므로, 사용하는 컨테이너에 포함된 프레임워크 버전을 확인해야 합니다.

### 2. 한강에서 NeMo/Megatron 실행

한강에서는 NeMo/Megatron이 설치된 컨테이너 이미지를 이용하여 분산학습을 수행할 수 있습니다.

기존 `kisti-container` 예제에서는 GH200 GPU 노드 2개를 사용하여 다음과 같은 병렬화 구성을 검증하였습니다.

| 항목                   | 설정   |
| -------------------- | ---- |
| 계산노드                 | 2개   |
| 노드당 GPU              | 4개   |
| 전체 GPU               | 8개   |
| Tensor Parallelism   | TP=4 |
| Data Parallelism     | DP=2 |
| Pipeline Parallelism | PP=1 |

위 구성은 노드 내부의 4개 GPU를 이용하여 TP를 수행하고, 노드 간에는 DP를 적용하는 방식입니다.

한강에서 제공하는 `kisti-container` 예제는 다음 경로에서 확인할 수 있습니다.

```bash
/apps/common/kisti-container/examples/06-nemo-megatron/
```

예제 파일이 제공되는 경우 사용자 작업 공간으로 복사하여 사용할 수 있습니다.

```bash
mkdir -p /scratch/$USER/ai-frameworks/nemo-megatron

cp -a /apps/common/kisti-container/examples/06-nemo-megatron/. \
    /scratch/$USER/ai-frameworks/nemo-megatron/

cd /scratch/$USER/ai-frameworks/nemo-megatron
```

배치 스크립트의 컨테이너 이미지, 실행 프로그램 및 자원 요청 설정을 확인한 후 다음과 같이 제출합니다.

```bash
sbatch run-pyxis.sbatch
```

제공 예제는 NeMo/Megatron의 분산학습 동작을 확인하기 위한 기능 검증용 프로그램입니다.

실제 연구 모델을 학습하려면 사용하는 모델과 데이터에 맞게 학습 프로그램, 병렬화 설정 및 체크포인트 경로 등을 수정해야 합니다.

또한 제공 예제는 특정 NeMo/Megatron 버전을 기준으로 작성되었으므로, 다른 버전의 컨테이너를 사용하는 경우 관련 API 및 실행 설정을 확인해야 합니다.

### 3. 모델 학습 구성 및 참고 자료

NeMo/Megatron을 이용한 실제 모델 학습에서는 다음 항목을 구성해야 합니다.

| 항목     | 설명                                                  |
| ------ | --------------------------------------------------- |
| 모델 구성  | 모델 종류, 계층 수, Hidden Size, Attention 구성 등            |
| 병렬화 설정 | DP, TP, PP, CP, EP 등의 병렬화 크기                        |
| 학습 데이터 | 데이터 경로, Tokenizer 및 데이터 형식                          |
| 학습 설정  | Batch Size, Sequence Length, Optimizer, Precision 등 |
| 체크포인트  | 모델 및 학습 상태 저장과 재시작                                  |

모델 크기와 학습 환경에 따라 적절한 병렬화 및 최적화 설정을 선택해야 합니다.

자세한 내용은 다음 공식 문서를 참고하시기 바랍니다.

* [NVIDIA NeMo Framework](https://docs.nvidia.com/nemo-framework/user-guide/latest/overview.html)
* [NVIDIA Megatron Bridge](https://docs.nvidia.com/nemo/megatron-bridge/latest/)
* [Megatron Bridge Parallelisms Guide](https://docs.nvidia.com/nemo/megatron-bridge/latest/parallelisms.html)
* [Megatron Bridge Training Guide](https://docs.nvidia.com/nemo/megatron-bridge/latest/training/README.html)

## 다. DeepSpeed 활용

### 1. DeepSpeed 소개

DeepSpeed는 Microsoft에서 개발한 대규모 딥러닝 모델 학습 최적화 라이브러리입니다.

DeepSpeed는 ZeRO(Zero Redundancy Optimizer)를 이용하여 데이터 병렬학습 과정에서 발생하는 GPU 메모리 사용량을 줄일 수 있습니다.

ZeRO는 모델 학습에 필요한 상태를 GPU 간에 분산하는 방식으로, 다음과 같은 단계로 구분됩니다.

| 구분           | 주요 기능                                    |
| ------------ | ---------------------------------------- |
| ZeRO Stage 1 | Optimizer State 분산                       |
| ZeRO Stage 2 | Optimizer State 및 Gradient 분산            |
| ZeRO Stage 3 | Optimizer State, Gradient 및 Parameter 분산 |

이러한 기능을 통해 일반적인 데이터 병렬학습보다 GPU별 모델 상태 메모리 사용량을 줄일 수 있습니다.

또한 DeepSpeed는 CPU/NVMe Offloading 등 다양한 메모리 최적화 기능을 지원합니다.

다만 메모리 사용량을 줄이는 과정에서 통신량이나 데이터 이동 비용이 증가할 수 있으므로, 모델 크기와 GPU 및 네트워크 환경에 따라 적절한 설정을 선택해야 합니다.

### 2. 한강에서 DeepSpeed 실행

한강에서는 DeepSpeed가 설치된 컨테이너를 이용하여 분산학습을 수행할 수 있습니다.

기존 `kisti-container` 예제에서는 GH200 GPU 노드 2개, 총 8개 GPU에서 DeepSpeed ZeRO-2 학습을 수행하는 구성을 검증하였습니다.

해당 예제는 다음 경로에서 확인할 수 있습니다.

```bash
/apps/common/kisti-container/examples/07-pytorch-deepspeed/
```

예제 파일이 제공되는 경우 다음과 같이 사용자 작업 공간으로 복사할 수 있습니다.

```bash
mkdir -p /scratch/$USER/ai-frameworks/deepspeed

cp -a /apps/common/kisti-container/examples/07-pytorch-deepspeed/. \
    /scratch/$USER/ai-frameworks/deepspeed/

cd /scratch/$USER/ai-frameworks/deepspeed
```

DeepSpeed는 JSON 설정 파일을 통해 ZeRO 단계와 학습 관련 옵션 등을 설정할 수 있습니다.

제공된 배치 스크립트에서 사용하는 `ds_config.json` 파일의 위치와 설정을 확인하고, 필요한 경우 연구 목적에 맞게 수정해야 합니다.

예제의 컨테이너 이미지, 학습 프로그램 및 Slurm 설정을 확인한 후 다음과 같이 제출합니다.

```bash
sbatch run-pyxis.sbatch
```

제공 예제는 DeepSpeed ZeRO-2 학습의 기본 동작을 확인하기 위한 프로그램입니다.

사용자 모델에 DeepSpeed를 적용하려면 해당 모델의 학습 코드가 DeepSpeed를 지원하도록 구성되어 있어야 합니다.

### 3. DeepSpeed 설정 및 참고 자료

DeepSpeed를 사용하는 경우 다음 항목을 확인해야 합니다.

| 항목                    | 설명                            |
| --------------------- | ----------------------------- |
| ZeRO Stage            | 모델 상태 분산 수준                   |
| Batch Size            | 전체 학습 Batch 및 GPU별 Microbatch |
| Gradient Accumulation | Gradient 누적 횟수                |
| Precision             | FP32, FP16, BF16 등 학습 정밀도     |
| Offloading            | CPU 또는 NVMe를 활용한 메모리 관리       |
| Checkpoint            | 분산 모델 및 Optimizer 상태 저장       |

DeepSpeed 설정과 모델 학습 코드의 세부 구성은 공식 문서를 참고하시기 바랍니다.

* [DeepSpeed Getting Started](https://www.deepspeed.ai/getting-started/)
* [DeepSpeed ZeRO Documentation](https://deepspeed.readthedocs.io/en/stable/zero3.html)
* [DeepSpeed Tutorials](https://www.deepspeed.ai/tutorials/)
* [DeepSpeed GitHub](https://github.com/deepspeedai/DeepSpeed)

## 라. 기타 AI 프레임워크 및 활용 시 유의사항

### 1. Hugging Face Accelerate

Hugging Face Accelerate는 PyTorch 기반 AI 프로그램의 분산학습 실행을 지원하는 라이브러리입니다.

DDP, FSDP 및 DeepSpeed 등의 분산학습 방식을 활용할 수 있으며, 설정 파일과 실행 도구를 이용하여 여러 GPU 및 노드에서 학습 프로그램을 실행할 수 있습니다.

구체적인 사용 방법은 다음 공식 문서를 참고하시기 바랍니다.

* [Hugging Face Accelerate](https://huggingface.co/docs/accelerate/index)
* [Accelerate with DeepSpeed](https://huggingface.co/docs/accelerate/usage_guides/deepspeed)

### 2. AI 프레임워크 활용 시 유의사항

사용자 정의 AI 프레임워크를 실행할 때는 다음 사항을 확인해야 합니다.

* 사용 중인 컨테이너와 AI 프레임워크의 버전 호환성
* GPU 드라이버, CUDA 및 NCCL 라이브러리 호환성
* GH200 계산노드의 CPU 아키텍처(aarch64) 지원 여부
* 다중 노드 작업에서 NCCL의 Slingshot 통신 환경 연동
* 모델 학습 데이터 및 체크포인트의 저장 경로
* GPU 수와 모델 병렬화 설정의 일치 여부

특히 대규모 모델 학습에서는 컨테이너, 프레임워크 및 통신 라이브러리의 버전 조합에 따라 실행 방법이 달라질 수 있으므로, KISTI에서 제공하는 실행 환경과 검증된 예제를 우선 활용하는 것을 권장합니다.

사용자 정의 컨테이너 구성 방법은 「사용자 정의 AI 환경 구성」 페이지를 참고하시기 바랍니다.
