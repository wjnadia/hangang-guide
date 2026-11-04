# OpenFOAM

OpenFOAM은 오픈소스 CFD(전산유체역학) 소프트웨어입니다. 이 문서는 KISTI 6호기(한강) **CPU 파티션**과 \*\*GPU 파티션(GH200 Grace CPU)\*\*에서 OpenFOAM을 설치하고 실행하는 방법을 설명합니다.

> * OpenFOAM은 **v1912 버전부터** 아래 방법으로 컴파일할 수 있습니다.
> * OpenFOAM은 OpenMP/thread를 지원하지 않으므로 **MPI 단독**으로 실행합니다.
> * 6호기 기본 환경에서 추가로 설치해야 하는 필수 라이브러리는 없습니다.

***

## 1. CPU 파티션

### 설치 환경

| 항목       | 내용                                   |
| -------- | ------------------------------------ |
| 시스템      | 6호기 한강 CPU 파티션                       |
| OS       | SUSE Linux Enterprise Server 15 SP6  |
| CPU      | AMD EPYC 9745 128-Core Processor × 2 |
| 메모리      | 768 GB (노드당)                         |
| 컴파일러     | GCC 14.2.0                           |
| MPI      | Cray MPICH 9.1.0                     |
| OpenFOAM | v1912 이후 버전                          |

### 가. 설치 방법

#### 1) 소스 다운로드

OpenCFD에서 배포하는 `OpenFOAM-v<YYMM>`과 `ThirdParty-v<YYMM>` 소스 파일을 다운로드한 후, **같은 디렉터리 아래에** 두 디렉터리가 함께 존재하도록 압축을 해제합니다. (`<YYMM>`은 2자리 연도 + 2자리 월, 예: `v2512`)

```bash
$ tar xzf OpenFOAM-v<YYMM>.tgz
$ tar xzf ThirdParty-v<YYMM>.tgz
$ ls
OpenFOAM-v<YYMM>  ThirdParty-v<YYMM>
```

> **(선택) METIS 사용 시** 기본 도메인 분할 방식인 scotch 외에 metis를 사용하려면, metis 5.1.0 소스를 아래 경로에 압축 해제합니다. `ThirdParty-v<YYMM>/sources/metis/metis-5.1.0`

#### 2) 모듈 설정

```bash
$ module purge
$ module load xpmem libfabric craype-x86-turin
$ module load PrgEnv-gnu
```

#### 3) build 디렉터리 생성 및 설치 경로 설정

OpenFOAM은 별도의 build 디렉터리를 만들지 않고 \*\*소스 디렉터리 안에서 바로 빌드(in-source build)\*\*하며, 압축을 해제한 `OpenFOAM-v<YYMM>` 디렉터리가 곧 설치 경로가 됩니다. 이후 작업은 모두 이 디렉터리에서 진행합니다.

```bash
$ cd OpenFOAM-v<YYMM>
```

#### 4) cmake/make 실행 (컴파일 옵션 및 빌드 환경 설정)

OpenFOAM은 cmake 대신 자체 빌드 도구인 **wmake**를 사용합니다. 컴파일 옵션은 `wmake/rules` 아래의 규칙 파일로 지정하고, 빌드 환경은 `etc/bashrc`로 설정합니다.

**(선택) `-march=znver5` 컴파일 옵션 적용**

```bash
$ cp wmake/rules/linux64Gcc/c++OptKNL wmake/rules/linux64Gcc/c++OptZnver5
$ sed -i 's/march=knl/march=znver5/g' wmake/rules/linux64Gcc/c++OptZnver5
```

**빌드 환경 설정**

```bash
$ source etc/bashrc WM_MPLIB=CRAY-MPICH WM_COMPILE_OPTION=OptZnver5
```

> `-march=znver5`를 사용하지 않을 경우 `WM_COMPILE_OPTION=OptZnver5`를 생략합니다 (기본 `-O3` 적용).

#### 5) 빌드 및 설치

```bash
$ ./Allwmake -j $(nproc) > build.log
```

**설치 확인**

정상적으로 컴파일이 완료되면 `build.log` 마지막에 아래와 같이 `bin`과 `lib`에 생성된 파일 수가 출력됩니다. 비정상 종료된 경우에는 이 메시지가 출력되지 않습니다. (OpenFOAM v2512 기준 예시)

```
========================================
  OpenFOAM-v2512
  Gcc system compiler
  linux64GccDPInt32OptZnver5, with CRAY-MPICH cray-mpich
  api = 2512
  patch = 0
  bin = 287 entries
  lib = 155 entries
```

### 나. 실행 방법

> 5호기와의 가장 큰 차이는 MPI 실행 명령이 `mpirun`이 아니라 \*\*`srun`\*\*이라는 점입니다. 기존에 `runParallel`이나 `mpirun`으로 실행하던 부분은 `srun`으로 바꾸고, 관련 명령행도 `srun`에 맞게 수정해야 합니다.
>
> CPU 파티션(AMD CPU)에서는 노드의 \*\*모든 코어(256 코어)\*\*를 사용했을 때 가장 좋은 성능을 보였습니다.

#### 배치 스크립트 샘플 (`openfoam_cpu.sh`)

2노드 × 256코어 = 512 MPI rank로 simpleFoam 케이스를 실행하는 예제입니다. 케이스 디렉터리(`0/`, `constant/`, `system/`)에서 제출합니다.

```bash
#!/bin/bash
#SBATCH --job-name=openfoam_cpu
#SBATCH --partition=<CPU 파티션명>
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=256       # CPU 파티션: 노드의 전체 코어 사용 권장
#SBATCH --comment="field=mae;appl=openfoam"
#SBATCH --time=01:00:00
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

# 1. 환경 설정
module purge
module load xpmem libfabric craype-x86-turin
module load PrgEnv-gnu
source /path/to/OpenFOAM-v<YYMM>/etc/bashrc WM_MPLIB=CRAY-MPICH WM_COMPILE_OPTION=OptZnver5

cd ${SLURM_SUBMIT_DIR}

# 2. 도메인 분할 (분할 수 = 전체 MPI rank 수)
foamDictionary -entry numberOfSubdomains -set ${SLURM_NTASKS} system/decomposeParDict
decomposePar -force > log.decomposePar 2>&1

# 3. 병렬 실행 (mpirun / runParallel 대신 srun 사용)
srun simpleFoam -parallel > log.simpleFoam 2>&1

# 4. 결과 병합 (필요 시)
reconstructPar -latestTime > log.reconstructPar 2>&1
```

```bash
$ sbatch openfoam_cpu.sh
```

> * 솔버(`simpleFoam`)는 케이스에 맞게 변경합니다 (`pimpleFoam`, `interFoam` 등).
> * `-march=znver5` 없이 빌드한 경우 `source` 줄에서 `WM_COMPILE_OPTION=OptZnver5`를 빼야 합니다.

#### 벤치마크(occDrivAerStaticMesh) 실행 예제

1. 아래 저장소를 다운로드합니다. [https://develop.openfoam.com/committees/hpc/-/tree/develop/incompressible/simpleFoam/occDrivAerStaticMesh](https://develop.openfoam.com/committees/hpc/-/tree/develop/incompressible/simpleFoam/occDrivAerStaticMesh)
2. _Instructions → Obtaining mesh file&#x73;_&#xC5D0;서 원하는 메쉬(65M / 110M / 236M)를 받아 `constant/` 디렉터리 아래에 압축 해제합니다.
3. `system/include/caseDefinition` 수정
   * `nCores`: 전체 코어(MPI rank) 수 지정
   * `decompositionMethod`: `scotch`로 변경 권장 (기본값 `hierarchical`을 유지할 경우 `nCores`에 맞게 `nHierarchical`의 3차원 분할 값도 정확히 수정해야 함)
4. `Allrun` 파일 수정
   *   SBATCH 옵션 (원본 → 수정)

       ```bash
       # 원본
       #SBATCH --ntasks=512
       #SBATCH --ntasks-per-node=128
       #SBATCH --partition=arm

       # 수정 (--ntasks 값은 nCores와 동일해야 함)
       #SBATCH --ntasks=<nCores>
       #SBATCH --partition=<CPU 파티션명>  
       ```
   *   MPI 실행 명령

       ```bash
       # 원본
       parEx="mpirun -np ${nProcs} ${OMPI_FLAGS}"
       # 수정
       parEx="srun"
       ```
5.  작업 제출

    ```bash
    $ sbatch Allrun
    ```

***

## 2. GPU 파티션 (GH200 Grace CPU)

### 설치 환경

| 항목       | 내용                                       |
| -------- | ---------------------------------------- |
| 시스템      | 6호기 한강 GPU 파티션                           |
| OS       | SUSE Linux Enterprise Server 15 SP6      |
| CPU      | NVIDIA GH200 Grace 72-Core Processor × 4 |
| 메모리      | 480 GB (노드당)                             |
| 컴파일러     | GCC 14.2.0                               |
| MPI      | Cray MPICH 9.1.0                         |
| OpenFOAM | v1912 이후 버전                              |

### 가. 설치 방법

#### 1) 소스 다운로드

CPU 파티션과 동일하게 `OpenFOAM-v<YYMM>`과 `ThirdParty-v<YYMM>`을 같은 디렉터리 아래에 압축 해제합니다.

```bash
$ tar xzf OpenFOAM-v<YYMM>.tgz
$ tar xzf ThirdParty-v<YYMM>.tgz
```

> **(선택) METIS 사용 시** metis 5.1.0 소스를 `ThirdParty-v<YYMM>/sources/metis/metis-5.1.0`에 압축 해제합니다.

#### 2) 모듈 설정

```bash
$ module purge
$ module load xpmem libfabric craype-arm-grace
$ module load PrgEnv-gnu
```

#### 3) build 디렉터리 생성 및 설치 경로 설정

별도 build 디렉터리 없이 소스 디렉터리에서 빌드하며, `OpenFOAM-v<YYMM>` 디렉터리가 설치 경로가 됩니다.

```bash
$ cd OpenFOAM-v<YYMM>
```

#### 4) cmake/make 실행 (컴파일 옵션 및 빌드 환경 설정)

**(선택) `-mcpu=grace` 컴파일 옵션 적용**

```bash
$ cp wmake/rules/linuxARM64Gcc/c++Opt wmake/rules/linuxARM64Gcc/c++OptGrace
$ sed -i 's/-O3/-O3 -mcpu=grace/g' wmake/rules/linuxARM64Gcc/c++OptGrace
```

**빌드 환경 설정**

```bash
$ source etc/bashrc WM_MPLIB=CRAY-MPICH WM_COMPILE_OPTION=OptGrace
```

> `-mcpu=grace`를 사용하지 않을 경우 `WM_COMPILE_OPTION=OptGrace`를 생략합니다.

#### 5) 빌드 및 설치

```bash
$ ./Allwmake -j $(nproc) > build.log
```

정상 완료 시 `build.log` 마지막에 OpenFOAM 버전, 컴파일 옵션, `bin`/`lib` 생성 파일 수가 출력됩니다. (CPU 파티션의 설치 확인 예시 참고)

### 나. 실행 방법

> GH200 노드에서는 노드의 288 코어를 모두 사용하는 것보다 \*\*노드당 256 코어(GH200 소켓당 72 코어 중 64 코어)\*\*만 사용하는 것이 노드 확장성 측면에서 더 좋은 성능을 보였습니다.
>
> MPI 실행은 CPU 파티션과 동일하게 `mpirun` 대신 \*\*`srun`\*\*을 사용합니다.

#### 배치 스크립트 샘플 (`openfoam_gpu.sh`)

2노드 × 256코어(GH200 소켓당 64코어) = 512 MPI rank로 실행하는 예제입니다. OpenFOAM은 Grace CPU에서만 실행됩니다.

```bash
#!/bin/bash
#SBATCH --job-name=openfoam_gh200
#SBATCH --partition=<GPU 파티션명>
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=256       # 288코어 전체보다 256코어(소켓당 64코어) 사용 권장
#SBATCH --comment="field=mae;appl=openfoam"
#SBATCH --time=01:00:00
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.err

# 1. 환경 설정
module purge
module load xpmem libfabric craype-arm-grace
module load PrgEnv-gnu
source /path/to/OpenFOAM-v<YYMM>/etc/bashrc WM_MPLIB=CRAY-MPICH WM_COMPILE_OPTION=OptGrace

cd ${SLURM_SUBMIT_DIR}

# 2. 도메인 분할 (분할 수 = 전체 MPI rank 수)
foamDictionary -entry numberOfSubdomains -set ${SLURM_NTASKS} system/decomposeParDict
decomposePar -force > log.decomposePar 2>&1

# 3. 병렬 실행 (mpirun / runParallel 대신 srun 사용)
srun simpleFoam -parallel > log.simpleFoam 2>&1

# 4. 결과 병합 (필요 시)
reconstructPar -latestTime > log.reconstructPar 2>&1
```

```bash
$ sbatch openfoam_gpu.sh
```

> * 소켓당 64코어씩 고르게 배치되도록 하려면 `srun --distribution=block:cyclic` 같은 배치 옵션을 시스템에서 확인 후 사용합니다.
> * 메모리를 CPU 코어 수에 비례해 할당하는 시스템이라면, 256코어만 요청할 때 노드 메모리 전체가 할당되지 않을 수 있습니다. 메모리가 부족하면 할당 정책을 확인해야 합니다.
> * `-mcpu=grace` 없이 빌드한 경우 `source` 줄에서 `WM_COMPILE_OPTION=OptGrace`를 빼야 합니다.

#### 벤치마크(occDrivAerStaticMesh) 실행 예제

CPU 파티션의 벤치마크 실행 절차와 동일하며, `Allrun`의 SBATCH 옵션만 GPU 파티션에 맞게 수정합니다.

```bash
#SBATCH --ntasks=<nCores>
#SBATCH --ntasks-per-node=256
#SBATCH --partition=<GPU 파티션명>
```

```bash
$ sbatch Allrun
```
