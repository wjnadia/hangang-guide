# PyTorch

한강 슈퍼컴퓨터에서는 PyTorch를 이용하여 GPU 기반 인공지능(AI) 모델의 학습 및 추론을 수행할 수 있습니다. 사용자는 KISTI에서 제공하는 AI 컨테이너 또는 직접 구성한 AI 실행 환경에서 PyTorch 프로그램을 실행할 수 있습니다. PyTorch는 단일 GPU뿐만 아니라 여러 GPU를 활용하는 병렬 처리를 지원하며, NCCL 등의 통신 라이브러리를 이용하여 다중 노드 분산학습을 수행할 수 있습니다.

본 페이지에서는 PyTorch의 기본적인 사용 방법과 DistributedDataParallel(DDP), Fully Sharded Data Parallel(FSDP)을 활용한 분산학습 방법을 설명합니다. 또한 대규모 모델 학습에 활용되는 Tensor Parallelism 및 Pipeline Parallelism을 간략히 소개합니다.

## 가. PyTorch 기본 사용

### 1. PyTorch 실행 환경 확인

PyTorch를 실행하기 전에 GPU 및 CUDA 사용 가능 여부를 확인해야 합니다.

KISTI 제공 AI 컨테이너 또는 사용자가 구성한 PyTorch 환경에서 다음 명령어를 실행하여 PyTorch 버전을 확인합니다.

```bash
python -c "import torch; print(torch.__version__)"
```

설치된 PyTorch 버전과 CUDA 사용 가능 여부는 다음과 같이 확인할 수 있습니다.

```python
import torch

print("PyTorch Version:", torch.__version__)
print("CUDA Version:", torch.version.cuda)
print("CUDA Available:", torch.cuda.is_available())
print("GPU Count:", torch.cuda.device_count())

if torch.cuda.is_available():
    for i in range(torch.cuda.device_count()):
        print(f"GPU {i}: {torch.cuda.get_device_name(i)}")
```

정상적으로 GPU가 인식되면 `CUDA Available`이 `True`로 출력되며, 현재 작업에서 사용 가능한 GPU의 수와 장치 이름을 확인할 수 있습니다.

GPU 장치는 Slurm을 통해 할당받은 자원을 기준으로 확인해야 합니다.

### 2. 단일 GPU 프로그램 실행

PyTorch에서는 Tensor 및 모델을 CUDA 장치로 이동하여 GPU 연산을 수행할 수 있습니다.

다음은 단일 GPU에서 간단한 신경망 연산을 수행하는 예제입니다.

**single\_gpu.py**

```python
import torch

assert torch.cuda.is_available(), "CUDA GPU를 사용할 수 없습니다."

device = torch.device("cuda:0")

model = torch.nn.Linear(1024, 1024).to(device)

x = torch.randn(32, 1024, device=device)
y = model(x)

print("Device:", device)
print("Output Shape:", y.shape)
```

GPU 자원을 할당받고 PyTorch 실행 환경을 준비한 후 다음과 같이 실행합니다.

```bash
python single_gpu.py
```

정상적으로 실행되면 다음과 같은 결과가 출력됩니다.

```
Device: cuda:0
Output Shape: torch.Size([32, 1024])
```

실제 AI 모델의 학습 및 추론도 동일한 방식으로 모델과 데이터를 GPU에 배치하여 수행할 수 있습니다.

## 나. 단일 노드 Multi-GPU 활용

### 1. DistributedDataParallel(DDP)

PyTorch는 `torch.distributed`를 통해 다수의 GPU를 이용한 분산학습을 지원합니다.

DistributedDataParallel(DDP)은 대표적인 데이터 병렬학습 방식으로, 각 GPU에 동일한 모델을 배치하고 서로 다른 데이터를 처리합니다. 학습 과정에서는 GPU별로 계산한 Gradient를 동기화하여 모델을 갱신합니다.

일반적으로 GPU마다 하나의 프로세스를 실행하며, GPU 간 Gradient 동기화에는 NCCL 백엔드를 사용할 수 있습니다.

DDP는 모델 전체가 단일 GPU 메모리에 적재될 수 있는 경우에 적합하며, 여러 GPU를 이용하여 학습 처리량을 높이는 데 활용할 수 있습니다.

### 2. DDP 분산학습 예제

다음은 PyTorch DDP를 이용한 간단한 분산학습 예제입니다.

**ddp\_example.py**

```python
import os
import torch
import torch.distributed as dist

from torch.nn.parallel import DistributedDataParallel as DDP


def main():
    local_rank = int(os.environ["LOCAL_RANK"])

    torch.cuda.set_device(local_rank)
    dist.init_process_group(backend="nccl")

    rank = dist.get_rank()
    world_size = dist.get_world_size()

    device = torch.device(f"cuda:{local_rank}")

    model = torch.nn.Linear(1024, 1024).to(device)
    model = DDP(model, device_ids=[local_rank])

    optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

    torch.manual_seed(2026 + rank)

    for step in range(10):
        x = torch.randn(32, 1024, device=device)
        target = torch.randn(32, 1024, device=device)

        optimizer.zero_grad()

        output = model(x)
        loss = torch.nn.functional.mse_loss(output, target)

        loss.backward()
        optimizer.step()

    torch.cuda.synchronize()

    print(
        f"Rank: {rank}, "
        f"World Size: {world_size}, "
        f"GPU: {local_rank}, "
        f"Loss: {loss.item():.4f}",
        flush=True
    )

    dist.destroy_process_group()


if __name__ == "__main__":
    main()
```

이 예제에서는 GPU마다 하나의 프로세스가 실행되며, DDP가 학습 과정에서 Gradient 동기화를 수행합니다.

예제에서는 분산학습 동작을 확인하기 위해 임의의 입력 데이터를 생성합니다. 실제 학습에서는 GPU별로 서로 다른 학습 데이터를 처리하도록 `DistributedSampler` 등을 활용해야 합니다.

자세한 내용은 [PyTorch DistributedDataParallel 공식 문서](https://docs.pytorch.org/tutorials/intermediate/ddp_tutorial.html)를 참고하시기 바랍니다.

### 3. torchrun을 이용한 실행

PyTorch에서 제공하는 `torchrun`을 이용하면 분산학습에 필요한 프로세스를 실행하고 관리할 수 있습니다.

예를 들어 단일 노드에서 GPU 4개를 할당받은 경우 다음과 같이 실행합니다.

```bash
torchrun \
    --standalone \
    --nnodes=1 \
    --nproc-per-node=4 \
    ddp_example.py
```

주요 옵션은 다음과 같습니다.

| 옵션                 | 설명              |
| ------------------ | --------------- |
| `--standalone`     | 단일 노드 분산학습 실행   |
| `--nnodes`         | 분산학습에 참여하는 노드 수 |
| `--nproc-per-node` | 노드별 실행 프로세스 수   |

위 명령어는 하나의 노드에서 4개의 프로세스를 생성하여 각 GPU에서 분산학습을 수행합니다.

`torchrun`은 프로세스마다 `RANK`, `LOCAL_RANK`, `WORLD_SIZE` 등의 환경변수를 설정하므로, 학습 프로그램에서 이를 이용하여 GPU와 프로세스를 구분할 수 있습니다.

## 다. 다중 노드 분산학습

### 1. 다중 노드 분산학습 개요

단일 노드에서 사용할 수 있는 GPU 수를 초과하는 AI 작업은 여러 계산노드에 분산하여 수행할 수 있습니다.

PyTorch에서는 `torch.distributed`와 `torchrun`을 이용하여 다중 노드 분산학습을 구성할 수 있습니다.

한강에서는 노드 간 통신에 Slingshot 인터커넥트를 사용하며, NCCL이 이를 효율적으로 활용하기 위해서는 적절한 통신 라이브러리 및 실행 환경 구성이 필요합니다.

따라서 다중 노드 분산학습을 수행할 때는 KISTI 제공 AI 컨테이너 또는 Slingshot 통신 환경과 연동된 사용자 정의 AI 환경을 사용하는 것을 권장합니다.

### 2. Slurm을 이용한 다중 노드 실행

다중 노드 분산학습에서는 각 노드에서 `torchrun`을 실행하고, 전체 프로세스가 동일한 분산학습 작업에 참여하도록 구성해야 합니다.

예를 들어 2개 노드에서 각각 4개 GPU를 사용하는 경우 총 8개의 학습 프로세스가 실행됩니다.

Slurm을 이용한 실행 예제는 다음과 같습니다.

> **참고:** 아래 스크립트는 표준 Slurm과 PyTorch를 이용한 구성 예시입니다. 한강의 실제 파티션, GPU 자원 요청 방식 및 제공 AI 환경 실행 명령어는 확인 후 적용해야 합니다.

**pytorch\_multinode.sh**

```bash
#!/bin/bash
#SBATCH --job-name=pytorch-ddp
#SBATCH --partition=normal
#SBATCH --comment="field=misc;appl=pytorch"
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=1
#SBATCH --gpus-per-node=4
#SBATCH --cpus-per-task=72
#SBATCH --exclusive
#SBATCH --time=00:10:00

KC=/apps/common/kisti-container/bin/kisti-container
IMAGE=/apps/common/kisti-container/images/pytorch-25.03-py3-aarch64.sif

module purge
module load aws-ofi-nccl/1.20.0 libfabric/2.3.1 singularity/4.5.0

export MASTER_ADDR=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n 1)
export MASTER_PORT=$((20000 + SLURM_JOB_ID % 20000))

srun --mpi=none --kill-on-bad-exit=1 --distribution=block:block \
     --gpu-bind=none --gres-flags=allow-task-sharing \
  "$KC" --runtime singularity --workload generic --mpi none --nccl ofi "$IMAGE" \
    torchrun \
      --nnodes="$SLURM_JOB_NUM_NODES" \
      --nproc-per-node=4 \
      --rdzv-id="$SLURM_JOB_ID" \
      --rdzv-backend=c10d \
      --rdzv-endpoint="$MASTER_ADDR:$MASTER_PORT" \
      ddp_example.py
```

스크립트를 작성한 후 다음 명령어로 작업을 제출합니다.

```bash
sbatch pytorch_multinode.sh
```

위 예제에서는 Slurm이 2개 노드를 할당하고, `srun`이 각 노드에서 하나의 `torchrun`을 실행합니다. 이후 각 `torchrun`이 4개의 GPU 프로세스를 생성합니다.

실제 한강 환경에서는 Slurm의 GPU 할당 및 바인딩 정책에 따라 실행 옵션이 달라질 수 있습니다. 또한 컨테이너를 사용하는 경우에는 제공되는 실행 도구에 맞게 명령어를 조정해야 합니다.

### 3. 분산학습 실행 결과 확인

분산학습이 정상적으로 구성되면 각 프로세스에서 Rank와 World Size 정보를 확인할 수 있습니다.

2개 노드에서 각각 GPU 4개를 사용하는 경우 다음과 같이 총 8개의 Rank가 생성됩니다.

```
Rank: 0, World Size: 8, GPU: 0, Loss: ...
Rank: 1, World Size: 8, GPU: 1, Loss: ...
Rank: 2, World Size: 8, GPU: 2, Loss: ...
Rank: 3, World Size: 8, GPU: 3, Loss: ...
Rank: 4, World Size: 8, GPU: 0, Loss: ...
Rank: 5, World Size: 8, GPU: 1, Loss: ...
Rank: 6, World Size: 8, GPU: 2, Loss: ...
Rank: 7, World Size: 8, GPU: 3, Loss: ...
```

실제 출력 순서는 프로세스 실행 순서에 따라 달라질 수 있으며, GPU 번호는 각 노드 내에서 지정된 `LOCAL_RANK`를 나타냅니다.

모든 Rank가 정상적으로 실행되었다는 사실만으로 Slingshot의 고속 통신 경로가 활용되고 있다고 판단할 수는 없습니다.

NCCL 통신 라이브러리의 설정 방법과 Slingshot 인터커넥트 사용 여부를 확인하는 절차는 「NCCL 및 Slingshot 활용」 페이지에서 자세히 설명합니다.

## 라. PyTorch 분산학습 방식

### 1. 주요 분산학습 방식

PyTorch에서는 모델 크기와 GPU 자원, 학습 목적에 따라 다양한 분산학습 방식을 활용할 수 있습니다.

| 방식                                | 주요 특징                                  | 권장 사용 환경                    |
| --------------------------------- | -------------------------------------- | --------------------------- |
| DistributedDataParallel(DDP)      | GPU별 모델 복제 및 Gradient 동기화              | 모델이 단일 GPU 메모리에 적재되는 경우     |
| Fully Sharded Data Parallel(FSDP) | 모델 파라미터, Gradient 및 Optimizer State 분산 | 단일 GPU의 메모리 용량이 부족한 경우      |
| Tensor Parallelism(TP)            | 모델 내부 텐서를 여러 GPU에 분할                   | 대규모 모델의 연산 및 메모리 분산이 필요한 경우 |
| Pipeline Parallelism(PP)          | 모델의 계층을 여러 GPU에 분할                     | 여러 계층으로 구성된 대규모 모델 학습       |

이러한 방식은 서로 독립적으로 사용하거나 조합하여 활용할 수 있으며, 모델 구조와 통신 환경에 따라 적절한 병렬화 구성을 선택해야 합니다.

### 2. Fully Sharded Data Parallel(FSDP2)

DDP에서는 각 GPU가 전체 모델의 파라미터와 Gradient, Optimizer State를 유지해야 하므로 모델 크기가 증가하면 GPU 메모리 사용량이 제한 요소가 될 수 있습니다.

FSDP는 이러한 모델 상태를 여러 GPU에 분산하여 메모리 사용량을 줄이는 방식입니다.

PyTorch의 FSDP2는 `fully_shard()` API를 이용하여 모델과 하위 모듈에 샤딩을 적용합니다. 학습 과정에서 필요한 파라미터를 AllGather 통신으로 수집하고, Gradient는 ReduceScatter 등을 이용하여 분산 처리합니다.

다음은 FSDP2의 기본적인 동작을 확인하기 위한 예제입니다.

**fsdp2\_example.py**

```python
import os
import torch
import torch.distributed as dist

from torch import nn
from torch.distributed.fsdp import fully_shard


class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()

        self.layers = nn.ModuleList([
            nn.Sequential(
                nn.Linear(1024, 1024),
                nn.ReLU()
            )
            for _ in range(4)
        ])

        self.output = nn.Linear(1024, 1024)

    def forward(self, x):
        for layer in self.layers:
            x = layer(x)

        return self.output(x)


def main():
    local_rank = int(os.environ["LOCAL_RANK"])

    torch.cuda.set_device(local_rank)
    dist.init_process_group(backend="nccl")

    rank = dist.get_rank()
    world_size = dist.get_world_size()

    device = torch.device(f"cuda:{local_rank}")

    # 모든 Rank에서 동일한 초기 모델 생성
    torch.manual_seed(2026)

    model = SimpleModel().to(device)

    # 하위 계층부터 FSDP2 적용
    for layer in model.layers:
        fully_shard(layer)

    # 나머지 모델 파라미터에도 적용
    fully_shard(model)

    # FSDP2 적용 후 Optimizer 생성
    optimizer = torch.optim.AdamW(
        model.parameters(),
        lr=0.001
    )

    # Rank별로 서로 다른 입력 데이터 생성
    torch.manual_seed(2026 + rank)

    for step in range(10):
        x = torch.randn(32, 1024, device=device)
        target = torch.randn(32, 1024, device=device)

        optimizer.zero_grad()

        output = model(x)
        loss = torch.nn.functional.mse_loss(output, target)

        loss.backward()
        optimizer.step()

    torch.cuda.synchronize()

    print(
        f"Rank: {rank}, "
        f"World Size: {world_size}, "
        f"Loss: {loss.item():.4f}",
        flush=True
    )

    dist.destroy_process_group()


if __name__ == "__main__":
    main()
```

위 예제에서는 `fully_shard()`를 각 계층에 적용한 후 전체 모델에도 적용합니다. 이를 통해 학습 과정에서 모델 파라미터와 Gradient 등을 여러 GPU에 분산하여 관리할 수 있습니다.

FSDP2를 적용한 후에는 샤딩된 파라미터를 기준으로 Optimizer를 생성해야 합니다.

단일 노드에서 GPU 4개를 사용하는 경우 다음과 같이 실행합니다.

```bash
torchrun \
    --standalone \
    --nnodes=1 \
    --nproc-per-node=4 \
    fsdp2_example.py
```

다중 노드에서는 앞서 설명한 Slurm 실행 스크립트를 활용할 수 있으며, 실행 대상 Python 파일을 `fsdp2_example.py`로 변경하면 됩니다.

> **참고:** 이 예제는 FSDP2의 기본 동작을 확인하기 위한 소규모 모델입니다. 실제 대규모 모델의 메모리 절감 효과나 학습 성능을 측정하기 위한 예제는 아닙니다.

FSDP2에서는 일반적인 DDP와 체크포인트 처리 방식이 다를 수 있으므로, 실제 학습에서는 PyTorch Distributed Checkpoint(DCP) 등의 분산 체크포인트 기능을 활용해야 합니다.

자세한 사용 방법은 다음 공식 문서를 참고하시기 바랍니다.

* [PyTorch FSDP2 Tutorial](https://docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html)
* [PyTorch fully\_shard API](https://docs.pytorch.org/docs/stable/distributed.fsdp.fully_shard.html)
* [PyTorch Distributed Checkpoint](https://docs.pytorch.org/docs/stable/distributed.checkpoint.html)

### 3. Tensor Parallelism 및 Pipeline Parallelism

대규모 AI 모델 학습에서는 데이터 병렬화 외에도 모델 자체를 여러 GPU에 분할하는 방식이 사용됩니다.

#### **1) Tensor Parallelism(TP)**

Tensor Parallelism은 모델의 가중치 텐서나 연산을 여러 GPU에 분할하여 처리하는 방식입니다.

예를 들어 Transformer 모델의 Linear 연산을 여러 GPU에 분산하고, GPU별 연산 결과를 통신으로 결합할 수 있습니다.

TP는 모델의 개별 계층이 큰 경우에 유용하지만, 분할된 연산 과정에서 GPU 간 통신이 발생하므로 통신 성능을 고려해야 합니다. 일반적으로 고속 GPU 간 연결을 활용할 수 있는 환경에서 효과적입니다.

PyTorch는 `torch.distributed.tensor.parallel`을 통해 Tensor Parallelism을 지원합니다.

자세한 사용 방법은 [PyTorch Tensor Parallelism Tutorial](https://docs.pytorch.org/tutorials/intermediate/TP_tutorial.html)을 참고하시기 바랍니다.

#### **2) Pipeline Parallelism(PP)**

Pipeline Parallelism은 모델의 계층을 여러 GPU에 나누어 배치하고, 입력 데이터를 여러 Microbatch로 분할하여 처리하는 방식입니다.

각 GPU는 담당하는 모델 계층을 실행하며, 서로 다른 Microbatch를 병렬로 처리할 수 있습니다.

PP는 모델이 여러 계층으로 구성되어 있고 단일 GPU에 전체 모델을 배치하기 어려운 경우에 활용할 수 있습니다. 다만 Pipeline Stage 간 통신과 Microbatch 구성에 따라 성능이 달라질 수 있습니다.

PyTorch는 `torch.distributed.pipelining`을 통해 Pipeline Parallelism을 지원합니다.

자세한 사용 방법은 [PyTorch Pipeline Parallelism Tutorial](https://docs.pytorch.org/tutorials/intermediate/pipelining_tutorial.html)을 참고하시기 바랍니다.

#### **3) 병렬화 방식의 조합**

대규모 모델을 학습할 때는 DDP, FSDP, TP, PP 등의 병렬화 방식을 조합할 수 있습니다.

예를 들어 노드 내부에서는 고속 GPU 간 연결을 활용하여 TP를 수행하고, 노드 간에는 데이터 병렬화를 적용하는 방식으로 구성할 수 있습니다.

이러한 병렬화 구성은 모델 크기, GPU 수, 메모리 사용량 및 통신 환경에 따라 달라지므로 실제 학습 환경에 적합한 방식을 선택해야 합니다.

다차원 병렬화의 구성과 구현 방법은 [PyTorch Distributed Overview](https://docs.pytorch.org/tutorials/beginner/dist_overview.html)를 참고하시기 바랍니다.
