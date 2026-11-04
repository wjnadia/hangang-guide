# VASP

VASP(Vienna Ab initio Simulation Package)는 평면파 기반 제일원리(DFT) 전자구조 계산 프로그램입니다. 이 문서는 KISTI 6호기(한강) **CPU 파티션**과 \*\*GPU 파티션(GH200)\*\*에서 VASP를 설치하고 실행하는 방법을 설명합니다.

> * VASP는 **6.5.0 버전부터** 아래 방법으로 컴파일할 수 있습니다.
> * 6호기 기본 환경에서 추가로 설치해야 하는 필수 라이브러리는 없습니다.
> * VASP는 라이선스 소프트웨어이므로, 라이선스를 보유한 사용자만 소스를 받아 설치할 수 있습니다.

***

## 1. CPU 파티션

### 설치 환경

| 항목   | 내용                                                               |
| ---- | ---------------------------------------------------------------- |
| 시스템  | 6호기 한강 CPU 파티션                                                   |
| OS   | SUSE Linux Enterprise Server 15 SP6                              |
| CPU  | AMD EPYC 9745 128-Core Processor × 2                             |
| 메모리  | 768 GB (노드당)                                                     |
| 컴파일러 | AOCC 5.2.0 / Cray 21.0.0 / GNU 14.2.0 / Intel oneAPI 2025.3 중 선택 |
| MPI  | Cray MPICH 9.1.0                                                 |
| VASP | 6.5.0 이후 버전                                                      |

### 가. 설치 방법

#### 1) 소스 다운로드

VASP 포털에서 라이선스에 해당하는 VASP 6.5.0 이후 버전 소스를 받아 압축을 해제합니다.

```bash
$ tar xzf vasp.6.5.x.tgz
$ cd vasp.6.5.x
```

#### 2) 모듈 설정

사용할 컴파일러에 맞는 모듈을 로드합니다. `PrgEnv-*` 부분만 다릅니다.

| 컴파일러                | 모듈 설정                                |
| ------------------- | ------------------------------------ |
| AOCC 5.2.0          | `module load PrgEnv-aocc cray-fftw`  |
| Cray 21.0.0         | `module load PrgEnv-cray cray-fftw`  |
| GNU 14.2.0          | `module load PrgEnv-gnu cray-fftw`   |
| Intel oneAPI 2025.3 | `module load PrgEnv-intel cray-fftw` |

```bash
$ module purge
$ module load xpmem libfabric craype-x86-turin
$ module load PrgEnv-<aocc|cray|gnu|intel> cray-fftw
```

#### 3) build 디렉터리 생성 및 설치 경로 설정

VASP는 별도의 build 디렉터리 없이 **소스 최상위 디렉터리에서 빌드**하며, 실행 파일은 `bin/` 아래에 생성됩니다. 설치 경로는 압축을 해제한 디렉터리(`vasp.6.5.x`)가 됩니다.

#### 4) cmake/make 실행 (makefile.include 작성)

VASP는 cmake를 사용하지 않고, 소스 최상위 디렉터리의 \*\*`makefile.include`\*\*로 빌드 옵션을 지정합니다. 사용할 컴파일러에 맞는 아래 내용으로 `makefile.include`를 작성합니다.

<details>

<summary>AOCC 컴파일러용 makefile.include</summary>

```makefile
# Default precompiler options
CPP_OPTIONS = -DHOST=\"CrayEX\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Dtbdyn \
              -Dfock_dblbuf

CPP         = flang -E -ffree-form -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

FC          = ftn -g
FCL         = $(FC)

FREE        = -ffree-form -ffree-line-length-none

FFLAGS      = -w -fno-fortran-main -Mbackslash

OFLAG       = -O2
OFLAG_IN    = $(OFLAG)
DEBUG       = -O0

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = $(FC)
CC_LIB      = cc
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = CC
LLIBS       = -lstdc++

# WAR for AOCC versions >= 4.0
OBJECTS_O1 += symlib.o

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -march=znver5
FFLAGS     += $(VASP_TARGET_CPU)

LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
INCS       = -I$(FFTW_ROOT)/include
```

</details>

<details>

<summary>Cray 컴파일러용 makefile.include</summary>

```makefile
# Precompiler options
CPP_OPTIONS = -DHOST=\"CrayEX\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -DMPI_INPLACE \
              -Dvasp6 \
              -Dtbdyn \
              -Dfock_dblbuf

CPP        = cpp --traditional -E -P -Wno-endif-labels $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

FC         = ftn -g
FCL        = $(FC)

FREE       = -ffree -N 1023
FFLAGS     = -dC -rmo -emEb
# suppress warnings
FFLAGS     += -m 4

# O1 is recommended, higher opt levels cause compiler problems, performance difference between O1 and O2 is negligible
OFLAG      = -O1
OFLAG_IN   = $(OFLAG)
DEBUG      = -O0

LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
INCS       = -I$(FFTW_ROOT)/include
#
# For what used to be vasp.5.lib
CPP_LIB    = $(CPP)
FC_LIB     = $(FC)
CC_LIB     = cc
CFLAGS_LIB = -O
FFLAGS_LIB = -O1
FREE_LIB   = $(FREE)

OBJECTS_LIB= linpack_double.o getshmem.o

# For the parser library
CXX_PARS   = CC
LLIBS      += -lstdc++

# special cray workarounds
CRAYFTNVER=$(shell crayftn --version 2>/dev/null | grep "Version" | sed -n 's/.*Version \([0-9]\+\)\..*/\1/p')
CPP_OPTIONS += -D__DCRAYFTN_VERSION=$(CRAYFTNVER)
```

</details>

<details>

<summary>GNU 컴파일러용 makefile.include</summary>

```makefile
# Default precompiler options
CPP_OPTIONS = -DHOST=\"CrayEX\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Dtbdyn \
              -Dfock_dblbuf

CPP         = gcc -E -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

FC          = ftn -g
FCL         = $(FC)

FREE        = -ffree-form -ffree-line-length-none

FFLAGS      = -w -ffpe-summary=none

OFLAG       = -O2
OFLAG_IN    = $(OFLAG)
DEBUG       = -O0

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = $(FC)
CC_LIB      = cc
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = CC
LLIBS       = -lstdc++

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -march=znver5
FFLAGS     += $(VASP_TARGET_CPU)

# For gcc-10 and higher (comment out for older versions)
FFLAGS     += -fallow-argument-mismatch

LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
INCS       = -I$(FFTW_ROOT)/include
```

</details>

<details>

<summary>Intel oneAPI 컴파일러용 makefile.include</summary>

```makefile
# Default precompiler options
CPP_OPTIONS = -DHOST=\"CrayEX\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Dtbdyn \
              -Dfock_dblbuf

CPP         = fpp -f_com=no -free -w0  $*$(FUFFIX) $*$(SUFFIX) $(CPP_OPTIONS)

FC          = ftn -g
FCL         = $(FC)

FREE        = -free -names lowercase

FFLAGS      = -assume byterecl -w

OFLAG       = -O2
OFLAG_IN    = $(OFLAG)
DEBUG       = -O0

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = $(FC)
CC_LIB      = cc
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = CC
LLIBS       = -lstdc++

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -march=znver5
FFLAGS     += $(VASP_TARGET_CPU)

LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
INCS       = -I$(FFTW_ROOT)/include
```

</details>

#### 5) 빌드 및 설치

```bash
$ make -j std DEPS=1
$ make -j gam DEPS=1
```

**설치 확인**

정상적으로 컴파일이 완료되면 `bin/` 아래에 `vasp_std`와 `vasp_gam` 실행 파일이 생성됩니다.

```bash
$ ls bin/
vasp_gam  vasp_std
```

### 나. 실행 방법

> 5호기와의 가장 큰 차이는 MPI 실행 명령이 `mpirun`이 아니라 \*\*`srun`\*\*이라는 점입니다.

#### 배치 스크립트 샘플 (`vasp_cpu.sh`)

2노드 × 256 MPI rank(노드당 256코어)로 `vasp_std`를 실행하는 예제입니다. 입력 파일(`INCAR`, `POSCAR`, `POTCAR`, `KPOINTS`)이 있는 디렉터리에서 제출합니다.

```bash
#!/bin/bash
#SBATCH --job-name=vasp_cpu
#SBATCH --comment="field=mse;appl=vasp"
#SBATCH --partition=<CPU 파티션명>
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=256
#SBATCH --time=01:00:00
#SBATCH --exclusive
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

# 1. 환경 설정 (빌드에 사용한 PrgEnv와 동일하게 로드)
module purge
module load xpmem libfabric
module load PrgEnv-gnu          # PrgEnv-aocc / PrgEnv-cray / PrgEnv-intel 중 빌드 시 사용한 것

export FI_CXI_RX_MATCH_MODE=software
export FI_MR_CACHE_MONITOR=kdreg2

VASP_BIN=/path/to/vasp.6.5.x/bin

cd ${SLURM_SUBMIT_DIR}

# 2. 실행 (Gamma-point 전용 계산은 vasp_gam 사용)
#srun -m block:block,Pack --cpu-bind=cores ${VASP_BIN}/vasp_gam
srun -m block:block,Pack --cpu-bind=cores ${VASP_BIN}/vasp_std
```

```bash
$ sbatch vasp_cpu.sh
```

> * `--nodes`, `--ntasks-per-node`, `--time`은 계산 규모에 맞게 조정합니다.
> * `PrgEnv-*`는 빌드할 때 사용한 컴파일러와 같아야 합니다.

***

## 2. GPU 파티션 (GH200, CUDA 버전)

### 설치 환경

| 항목   | 내용                                       |
| ---- | ---------------------------------------- |
| 시스템  | 6호기 한강 GPU 파티션                           |
| OS   | SUSE Linux Enterprise Server 15 SP6      |
| CPU  | NVIDIA GH200 Grace 72-Core Processor × 4 |
| 메모리  | 480 GB (노드당)                             |
| 컴파일러 | NVHPC 25.3 + CUDA 12.8                   |
| MPI  | Cray MPICH 9.1.0                         |
| VASP | 6.5.0 이후 버전                              |

### 가. 설치 방법

#### 1) 소스 다운로드

CPU 파티션과 동일하게 VASP 6.5.0 이후 버전 소스를 받아 압축을 해제합니다.

```bash
$ tar xzf vasp.6.5.x.tgz
$ cd vasp.6.5.x
```

#### 2) 모듈 설정

```bash
$ module purge
$ module load xpmem libfabric craype-arm-grace
$ module load PrgEnv-nvidia cray-fftw
$ module swap nvidia nvidia/25.3
$ module load cudatoolkit/25.3_12.8 craype-accel-nvidia90 cray-pmi
```

#### 3) build 디렉터리 생성 및 설치 경로 설정

별도 build 디렉터리 없이 소스 최상위 디렉터리에서 빌드하며, 실행 파일은 `bin/` 아래에 생성됩니다.

#### 4) cmake/make 실행 (makefile.include 작성)

소스 최상위 디렉터리에 아래 내용으로 `makefile.include`를 작성합니다. (OpenACC + OpenMP, `-gpu=cc90,cuda12.8`)

<details>

<summary>NVHPC(CUDA)용 makefile.include</summary>

```makefile
# Precompiler options
CPP_OPTIONS = -DHOST=\"CrayEX\" -DMPI -DMPI_BLOCK=65536 -Duse_collective -DscaLAPACK -DCACHE_SIZE=4000 \
              -Davoidalloc -DMPI_INPLACE -Dvasp6 -Dtbdyn -Dfock_dblbuf -Dqd_emulate \
              -D_OPENMP -DCRAY_MPICH -DACC_OFFLOAD -DNVCUDA

CPP        = nvfortran -Mpreprocess -Mfree -Mextend -E $(CPP_OPTIONS) $*$(FUFFIX)  > $*$(SUFFIX)

GPU        ?= -gpu=cc90,cuda12.8

CC         = cc -acc -mp $(GPU)
FC         = ftn -acc -mp $(GPU) -g
FCL        = ftn -acc -mp $(GPU) -c++libs

FREE       = -Mfree

FFLAGS     = -Mbackslash -Mlarge_arrays

OFLAG      = -fast

DEBUG      = -Mfree -O0 -traceback

LLIBS       = -cudalib=cublas,cusolver,cufft -cuda

# Redefine the standard list of O1 and O2 objects
SOURCE_O1  := pade_fit.o minimax_dependence.o wave_window.o
SOURCE_O2  := pead.o

# For what used to be vasp.5.lib
CPP_LIB    = $(CPP)
FC_LIB     = $(FC)
CC_LIB     = $(CC)
CFLAGS_LIB = -O -w
FFLAGS_LIB = -O1 -Mfixed
FREE_LIB   = $(FREE)

OBJECTS_LIB= linpack_double.o

# For the parser library
CXX_PARS   = CC --no_warnings

NVROOT      =$(shell which nvfortran | awk -F /compilers/bin/nvfortran '{ print $$1 }')

# Improves performance when using NV HPC-SDK >=21.11 and CUDA >11.2
OFLAG_IN   = -fast -Mwarperf
SOURCE_IN  := nonlr.o

# Software emulation of quadruple precsion (mandatory)
QD         ?= $(NVROOT)/compilers/extras/qd
LLIBS      += -L$(QD)/lib -Wl,-rpath=$(QD)/lib -lqdmod -lqd
INCS       += -I$(QD)/include/qd

LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3 -lfftw3_omp
INCS       += -I$(FFTW_ROOT)/include

# Add -gpu=tripcount:host to compiler commands for NV HPC-SDK > 25.1
NVFORTRAN_VERSION := $(shell nvfortran --version | sed -n '2s/^nvfortran \([0-9.]*\).*/\1/p')
define greater_or_equal
$(shell printf '%s\n%s\n' '$(1)' '$(2)' | sort -V | head -n1 | grep -q '$(2)' && echo true || echo false)
endef
ifeq ($(call greater_or_equal,$(NVFORTRAN_VERSION),25.1),true)
    CC  += -gpu=tripcount:host
    FC  += -gpu=tripcount:host
endif
```

</details>

#### 5) 빌드 및 설치

```bash
$ make -j std DEPS=1
$ make -j gam DEPS=1
```

정상 완료 시 `bin/` 아래에 `vasp_std`와 `vasp_gam` 실행 파일이 생성됩니다.

### 나. 실행 방법

> * GPU 1개당 MPI rank 1개를 배치합니다 (`--ntasks-per-node`를 `--gres=gpu:` 값과 같게 설정).
> * `--cpus-per-task=72`는 각 rank가 GH200 소켓(Grace 72코어) 하나씩에 배치되도록 하기 위한 설정이며, OpenMP 스레드 수와는 관계없습니다.

#### 배치 스크립트 샘플 (`vasp_gpu.sh`)

1노드 × GPU 4개(4 MPI rank)로 `vasp_std`를 실행하는 예제입니다.

```bash
#!/bin/bash
#SBATCH --job-name=vasp_gpu
#SBATCH --comment="field=mse;appl=vasp"
#SBATCH --partition=<GPU 파티션명>
#SBATCH --nodes=1
#SBATCH --gres=gpu:4
#SBATCH --ntasks-per-node=4
#SBATCH --time=01:00:00
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

# 1. 환경 설정
module purge
module load xpmem libfabric
module load PrgEnv-nvidia
module load cudatoolkit/25.3_12.8 cray-pmi

export MPICH_GPU_SUPPORT_ENABLED=1
export FI_CXI_RX_MATCH_MODE=software
export FI_MR_CACHE_MONITOR=kdreg2

VASP_BIN=/path/to/vasp.6.5.x/bin

cd ${SLURM_SUBMIT_DIR}

# 2. 실행 (GPU당 1 rank, rank마다 GH200 소켓 1개 할당)
#srun --gpus-per-task=1 --gres-flags=allow-task-sharing --cpus-per-task=72 ${VASP_BIN}/vasp_gam
srun --gpus-per-task=1 --gres-flags=allow-task-sharing --cpus-per-task=72 ${VASP_BIN}/vasp_std
```

```bash
$ sbatch vasp_gpu.sh
```

> * 여러 노드를 사용할 경우 `--nodes`만 늘리고, `--gres=gpu:4`와 `--ntasks-per-node=4`는 그대로 둡니다.
