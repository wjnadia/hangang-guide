# 사용자 프로그래밍 환경

시스템 접속 후 개별 프로그래밍 환경 설정을 위해 모듈 환경을 제공합니다.

모듈 환경은 시스템에 설치된 컴파일러, 라이브러리, 응용 소프트웨어의 환경변수를 module 명령어로 간편하게 설정·변경할 수 있도록 지원하는 도구로, 여러 버전의 소프트웨어를 충돌 없이 선택하여 사용할 수 있습니다. 한강 시스템에서는 HPE Cray Programming Environment(PrgEnv)를 통해 컴파일러 환경 전체를 한 번에 전환할 수 있습니다.

## 가. 모듈 환경

### 1. 모듈 환경 기본 명령어

<table><thead><tr><th>명령어</th><th width="252">설명</th><th>예시</th></tr></thead><tbody><tr><td><code>module avail</code> (<code>av</code>)</td><td>설정 가능한 모듈 환경 리스트 출력</td><td><code>module av</code><br><code>module av cuda</code></td></tr><tr><td><code>module list</code> (<code>li</code>)</td><td>현재 설정된 모듈 환경 출력</td><td><code>module li</code></td></tr><tr><td><code>module show module_name</code></td><td>모듈 환경의 환경변수 및 설치 경로 확인</td><td><code>module show cuda/12.9</code></td></tr><tr><td><code>module load module_name</code></td><td>사용할 모듈 환경 추가</td><td><code>module load cuda/12.9</code></td></tr><tr><td><code>module unload module_name</code></td><td>설정한 모듈 환경 제거</td><td><code>module unload cuda/12.9</code></td></tr><tr><td><code>module swap module_name1 module_name2</code></td><td>모듈 환경 교체</td><td><code>module swap PrgEnv-cray PrgEnv-nvidia</code></td></tr><tr><td><code>module purge</code></td><td>현재 설정된 모든 모듈 환경 일괄 삭제</td><td><code>module purge</code></td></tr></tbody></table>

***

### 2. 모듈 설치 현황

기본적으로 제공하는 모듈 환경은 다음과 같으며 module av 명령으로 확인 가능합니다.

{% tabs %}
{% tab title="1. GPU 노드" %}
#### **1) 시스템 아키텍처**

```
craype-accel-host     craype-arm-grace      craype-hugepages2M    craype-hugepages512M  craype-network-ofi
craype-accel-nvidia90 craype-hugepages1G    craype-hugepages32M   craype-network-none   craype-network-ucx
```

#### **2) 컴파일러**

```
cuda/12.8             cuda/12.9_26.5        cudatoolkit/25.3_12.8 cudatoolkit/26.3_12.9 go/1.25.8             nvidia/25.9
cuda/12.9_25.9        cuda/13.0_25.9        cudatoolkit/25.9_12.9 cudatoolkit/26.5_12.9 go/1.26.1(default)    nvidia/26.3(default)
cuda/12.9_26.3        cuda/13.2_26.5        cudatoolkit/25.9_13.0 cudatoolkit/26.5_13.2 nvidia/25.3           nvidia/26.5
```

#### **3) 성능 분석 툴**

```
perftools             perftools-lite        perftools-lite-events perftools-lite-gpu    perftools-lite-loops  perftools-preload
```

#### **4) 프로그래밍 환경**

```
PrgEnv-cray/8.7.0(default)                cray-libsci/26.03.0(default)              cray-python/3.12.12
PrgEnv-gnu/8.7.0(default)                 cray-libsci_acc/25.09.2(default)          cray-stat/4.12.6(default)
PrgEnv-nvidia/8.7.0(default)              cray-libsci_acc/26.03.0                   cray-stat/4.12.8
apprentice3/3.2.1                         cray-mpich/8.1.33                         cray-zmqnet/1.3.2(default)
atp/3.15.7(default)                       cray-mpich/9.0.1                          craype/2.7.35(default)
cce/20.0.0                                cray-mpich/9.1.0(default)                 craype/2.7.36
cce/21.0.0(default)                       cray-mpich-abi/8.1.33                     craypkg-gen/1.3.36(default)
cpe/25.09(default)                        cray-mpich-abi/9.0.1                      craypkg-gen/1.3.37
cpe/26.03                                 cray-mpich-abi/9.1.0(default)             gcc-native/12
cray-R/4.4.0(default)                     cray-mpixlate/1.0.9                       gcc-native/13
cray-ccdb/5.0.7(default)                  cray-mrnet/5.1.6(default)                 gcc-native/14
cray-cti/2.20.0(default)                  cray-netcdf/4.9.2.1(default)              gdb4hpc/4.16.5(default)
cray-cti/2.21.0                           cray-netcdf/4.9.2.3                       gdb4hpc/4.16.6
cray-dsmml/0.3.1(default)                 cray-netcdf-hdf5parallel/4.9.2.1(default) papi/7.2.0.2(default)
cray-dyninst/12.3.6                       cray-netcdf-hdf5parallel/4.9.2.3          papi/7.3.0.1
cray-fftw/3.3.10.11(default)              cray-openshmemx/11.8.0(default)           perftools-base/25.09.0
cray-hdf5/1.14.3.7                        cray-parallel-netcdf/1.12.3.19            perftools-base/26.03.0(default)
cray-hdf5/1.14.3.9(default)               cray-parallel-netcdf/1.12.3.21(default)   sanitizers4hpc/1.1.6(default)
cray-hdf5-parallel/1.14.3.7               cray-pmi/6.1.16                           valgrind4hpc/2.13.6(default)
cray-hdf5-parallel/1.14.3.9(default)      cray-pmi/6.1.17(default)
cray-libsci/25.09.0                       cray-python/3.11.7(default)
```

#### **5) Cray 모듈**

```
libfabric/2.3.1(default)        xpmem/1.0.1-1.5_1_gfb6998056825
```

#### **6) 라이브러리**

```
aws-ofi-nccl/1.17.3                 cudss/0.8.0.10_cuda12               nccl/2.27.7                         nccl/2.30.7-nvidia(default)
aws-ofi-nccl/1.18.0                 cuquantum/26.01.0.4_cuda12(default) nccl/2.27.7-nvidia                  nvshmem/3.6.5(default)
aws-ofi-nccl/1.19.0                 cuquantum/26.06.0.17_cuda12         nccl/2.28.9                         openmpi/5.0.10(default)
aws-ofi-nccl/1.20.0(default)        fftw/2.1.5(default)                 nccl/2.28.9-nvidia                  petsc/3.24.5(default)
boost/1.72.0(default)               hdf4/4.2.16-2(default)              nccl/2.29.7                         pio/2.6.9(default)
boost/1.90.0                        hdf5/2.1.0                          nccl/2.29.7-nvidia
cudss/0.7.1.4_cuda12(default)       libxc/7.0.0(default)                nccl/2.30.7
```

#### **7) 상용소프트웨어**

```
fluent/2026_R1
```

#### **8) 기타**

```
OMB/7.5.2(default)             ferret/7.6.0(default)          nccl-tests/2.18.5              singularity/4.5.0
OMB/7.5.2-openmpi              git/2.53.0(default)            nccl-tests/2.19.2              singularity/4.5.0.bak
apptainer/1.4.5(default)       gnu-parallel/20260222(default) ncl/6.6.2(default)             tmux/3.6a(default)
cdo/2.6.0(default)             grads/2.2.1(default)           nco/5.3.6(default)             wgrib2/3.8.0(default)
cmake/4.2.3(default)           htop/3.4.1(default)            ncview/2.1.11(default)
cmake/4.4.3                    miniforge/26.1.1-3(default)    nvtop/3.3.2(default)
emacs/31.1(default)            nccl-tests/2.18.2(default)     singularity/3.8.7(default)
```
{% endtab %}

{% tab title="2.CPU 노드" %}
#### **1) 시스템 아키텍처**

```
craype-accel-amd-gfx90a  craype-accel-nvidia70    craype-hugepages1G       craype-network-ucx       craype-x86-milan-x       craype-x86-turin
craype-accel-amd-gfx908  craype-accel-nvidia80    craype-hugepages2M       craype-x86-cascadelake   craype-x86-rome
craype-accel-amd-gfx940  craype-accel-nvidia90    craype-hugepages32M      craype-x86-emr           craype-x86-skylake
craype-accel-amd-gfx942  craype-accel-nvidia-b100 craype-hugepages512M     craype-x86-genoa         craype-x86-spr
craype-accel-host        craype-accel-nvidia-b200 craype-network-none      craype-x86-icelake       craype-x86-spr-hbm
craype-accel-intel-max   craype-arm-grace         craype-network-ofi       craype-x86-milan         craype-x86-trento
```

#### **2) 컴파일러**

```
aocc/5.0.0            aocc/5.2.0(default)   go/1.26.1(default)    intel/2024.2          intel/2026.1
aocc/5.1.0            go/1.25.8             intel/2023.2.4        intel/2025.3(default)
```

#### **3) 성능 분석 툴**

```
perftools             perftools-lite        perftools-lite-events perftools-lite-gpu    perftools-lite-loops  perftools-preload
```

#### **4) 프로그래밍 환경**

```
PrgEnv-aocc/8.6.0                           cray-hdf5-parallel/1.14.3.7                 cray-parallel-netcdf/1.12.3.21(default)
PrgEnv-aocc/8.7.0(default)                  cray-hdf5-parallel/1.14.3.9(default)        cray-pmi/6.1.16
PrgEnv-cray/8.6.0                           cray-libsci/25.09.0                         cray-pmi/6.1.17(default)
PrgEnv-cray/8.7.0(default)                  cray-libsci/26.03.0(default)                cray-python/3.11.7
PrgEnv-gnu/8.6.0                            cray-libsci_acc/25.09.2                     cray-python/3.12.12(default)
PrgEnv-gnu/8.7.0(default)                   cray-libsci_acc/26.03.0(default)            cray-stat/4.12.6
PrgEnv-intel/8.6.0                          cray-mpich/8.1.33                           cray-stat/4.12.8(default)
PrgEnv-intel/8.7.0(default)                 cray-mpich/9.0.1                            cray-zmqnet/1.3.2(default)
apprentice3/3.2.1                           cray-mpich/9.1.0(default)                   craype/2.7.35
atp/3.15.7(default)                         cray-mpich-abi/8.1.33                       craype/2.7.36(default)
cce/20.0.0                                  cray-mpich-abi/9.0.1                        craypkg-gen/1.3.36
cce/21.0.0(default)                         cray-mpich-abi/9.1.0(default)               craypkg-gen/1.3.37(default)
cpe/25.09                                   cray-mpich-abi-pre-intel-5.0/8.1.33         gcc-native/12
cpe/26.03(default)                          cray-mpich-abi-pre-intel-5.0/9.0.1          gcc-native/13
cray-R/4.4.0(default)                       cray-mpich-abi-pre-intel-5.0/9.1.0(default) gcc-native/14(default)
cray-ccdb/5.0.7(default)                    cray-mpixlate/1.0.9(default)                gdb4hpc/4.16.5
cray-cti/2.20.0                             cray-mrnet/5.1.6(default)                   gdb4hpc/4.16.6(default)
cray-cti/2.21.0(default)                    cray-netcdf/4.9.2.1                         papi/7.2.0.2
cray-dsmml/0.3.1(default)                   cray-netcdf/4.9.2.3(default)                papi/7.3.0.1(default)
cray-dyninst/12.3.6(default)                cray-netcdf-hdf5parallel/4.9.2.1            perftools-base/25.09.0
cray-fftw/3.3.10.11(default)                cray-netcdf-hdf5parallel/4.9.2.3(default)   perftools-base/26.03.0(default)
cray-hdf5/1.14.3.7                          cray-openshmemx/11.8.0(default)             sanitizers4hpc/1.1.6(default)
cray-hdf5/1.14.3.9(default)                 cray-parallel-netcdf/1.12.3.19              valgrind4hpc/2.13.6(default)
```

#### **5) Cray 모듈**

```
libfabric/2.3.1(default)      xpmem/1.0.1-1.4_gf67ddfb318d4
```

#### **6) 라이브러리**

```
boost/1.72.0(default)   fftw/2.1.5(default)     hdf5/2.1.0              libxc/7.0.0(default)    petsc/3.24.5(default)
boost/1.90.0            hdf4/4.2.16-2(default)  impi/2021.17(default)   openmpi/5.0.10(default) pio/2.6.9(default)
```

#### **7) 상용소프트웨어**

```
abaqus/2024          abaqus/2025          abaqus/2026(default) cfx/2026_R1          fluent/2026_R1       gaussian/g16
```

#### **8) 기타**

```
OMB/7.5.2(default)             emacs/31.1(default)            htop/3.4.1(default)            singularity/3.8.7(default)
OMB/7.5.2-openmpi              ferret/7.6.0(default)          miniforge/26.1.1-3(default)    singularity/4.5.0
cdo/2.6.0(default)             git/2.53.0(default)            ncl/6.6.2(default)             singularity/4.5.0.bak
cmake/4.2.3(default)           gnu-parallel/20260222(default) nco/5.3.6(default)             tmux/3.6a(default)
cmake/4.4.3                    grads/2.2.1(default)           ncview/2.1.11(default)         wgrib2/3.8.0(default)
```
{% endtab %}
{% endtabs %}

* ANSYS 사용이 가능한 사용자 그룹은 대학교, 산업체(중소기업), 출연연구소에 한정합니다.
  * **사용 가능 사용자 그룹이 아니거나 사용 신청을 하지 않은 타 기관 사용자에 의한 ANSYS 사용은 ANSYS 사에 의해 법적 제재를 당할 수 있음에 유의 바랍니다.**
* Abaqus 사용이 가능한 사용자 그룹은 대학교, 산업체(중소기업), 출연연구소에 한정합니다.
  * **사용 가능 사용자 그룹이 아니거나 사용 신청을 하지 않은 타 기관 사용자에 의한 Abaqus 사용은 다쏘시스템코리아에 의해 법적 제재를 당할 수 있음에 유의 바랍니다.**
* Gaussian은 helpdesk 계정담당자(account@ksc.re.kr)를 통해 사용 권한 신청 후 사용 가능합니다.
  * **GaussView**는 [**MyKSC**](https://portal.my-6th.ksc.re.kr/) 웹 포털의 [**VNC 앱**](https://app.gitbook.com/s/jtnnlutdgkoEEgUQdY8k/myksc/undefined-1/undefined-1/vnc-linux)에서 사용할 수 있습니다.&#x20;
* MATLAB은 **사용자(소속기관)가 라이선스를 보유해야 합니다**. 사용법은 [**MATLAB via VNC**](https://app.gitbook.com/s/jtnnlutdgkoEEgUQdY8k/myksc/undefined-1/undefined-1/matlab-via-vnc) 참고 바랍니다.

## 나. 컴파일러 사용법 <a href="#id-1" id="id-1"></a>

### 1. 컴파일러 및 MPI 환경설정(modules) <a href="#id-1.-mpi-modules" id="id-1.-mpi-modules"></a>

{% tabs %}
{% tab title="GPU 노드" %}
#### **1) 지원 컴파일러**

* GPU 노드에서는 GNU/Cray/Nvidia 컴파일러를 지원합니다.

| 컴파일러   | 지원 버전                           | 모듈                                                          |
| ------ | ------------------------------- | ----------------------------------------------------------- |
| GNU    | 12.3.0, 13.3.1, 14.2.0(default) | gcc-native/12, gcc-native/13, gcc-native/14(default)        |
| Cray   | 20.0, 21.0(default)             | cce/20.0.0, cce/21.0.0(default)                             |
| NVIDIA | 25.3, 25.9, 26.3(default), 26.5 | nvidia/25.3, nvidia/25.9, nvidia/26.3(default), nvidia/26.5 |

#### **2) 컴파일 환경 설정**

컴파일 환경은 PrgEnv-xxx 모듈로 설정합니다. MPI는 cray-mpich가 기본 및 권장 설정입니다.

&#x20;   **a. GNU 컴파일러 환경설정**

```bash
$ module load PrgEnv-gnu
$ module li
Currently Loaded Modulefiles:
  1) gcc-native/14         3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-gnu/8.7.0
```

버전 변경 (gcc v14.2를 v13.3.1로 변경)

```bash
$ module swap gcc-native/14 gcc-native/13
$ module li
Currently Loaded Modulefiles:
  1) gcc-native/13         3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-gnu/8.7.0
```

&#x20;   **b. Cray 컴파일러 환경설정**

```bash
$ module load PrgEnv-cray
$ module li
Currently Loaded Modulefiles:
  1) cce/21.0.0            3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-cray/8.7.0
```

버전 변경 (cce v21.0을 v20.0으로 변경)

```bash
$ module swap cce/21.0.0 cce/20.0.0
$ module li
Currently Loaded Modulefiles:
  1) cce/20.0.0            3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-cray/8.7.0
```

&#x20;   **c. NVIDIA 컴파일러 환경설정**

```bash
$ module load PrgEnv-nvidia
$ module li
Currently Loaded Modulefiles:
  1) nvidia/26.3           3) cray-dsmml/0.3.1      5) cray-libsci/26.03.0   7) libfabric/2.3.1
  2) craype/2.7.36         4) cray-mpich/9.1.0      6) PrgEnv-nvidia/8.7.0   8) craype-network-ofi
```

버전 변경 (NVHPC v26.3을 v26.5로 변경)

```bash
$ module swap nvidia/26.3 nvidia/26.5
$ module li
Currently Loaded Modulefiles:
  1) nvidia/26.5           3) cray-dsmml/0.3.1      5) cray-libsci/26.03.0   7) libfabric/2.3.1
  2) craype/2.7.36         4) cray-mpich/9.1.0      6) PrgEnv-nvidia/8.7.0   8) craype-network-ofi
```

#### **3) Compiler wrapper**

Cray PE 환경에서는 컴파일을 위해 wrapper 명령(`ftn`/`cc`/`CC`)을 사용합니다.

Wrapper 명령은 현재 로드된 PrgEnv-xxx 모듈에 따라 실제 컴파일러(Cray, GNU, NVIDIA 등)를 자동으로 호출하며, 함께 로드된 라이브러리 모듈(cray-mpich, cray-libsci, cray-fftw 등)의 헤더 경로와 링크 옵션도 자동으로 추가합니다. 따라서 사용자는 컴파일러 종류와 관계없이 동일한 명령을 사용할 수 있고, -I, -L, -l 옵션을 직접 지정하지 않아도 됩니다. MPI 코드 컴파일도 wrapper 명령을 그대로 사용합니다.

| Wrapper |                                 설명                                 |
| :-----: | :----------------------------------------------------------------: |
|  `ftn`  | Fortran 코드 컴파일 명령이고 Nvidia 환경이라면 `nvfortran` or `mpif90` 대신 사용합니다. |
|   `cc`  |       C 코드 컴파일 명령이고 Nvidia 환경이라면 `nvcc` or `mpicc` 대신 사용합니다.       |
|   `CC`  |      C++ 코드 컴파일 명령이고 Nvidia 환경이라면 `nvc++` or `mpiCC` 대신 사용합니다.     |

다음은 PrgEnv-nvidia 모듈만 올린 후 wrapper 명령 ftn, cc, CC에서 사용하는 NVIDIA 컴파일러 버전을 확인한 것입니다.

```bash
$ module load PrgEnv-nvidia
$ ftn --version
nvfortran 25.5-0 linuxarm64 target on aarch64 Linux -tp neoverse-v2
NVIDIA Compilers and Tools
Copyright (c) 2025, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.

$ cc --version
nvc 25.5-0 linuxarm64 target on aarch64 Linux -tp neoverse-v2
NVIDIA Compilers and Tools
Copyright (c) 2025, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.

$ CC --version
nvc++ 25.5-0 linuxarm64 target on aarch64 Linux -tp neoverse-v2
NVIDIA Compilers and Tools
Copyright (c) 2025, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.
```

#### 4) CUDA 버전 변경

GPU 노드에서는 NVIDIA HPC SDK(nvidia 모듈) 버전과 CUDA(cuda 모듈) 버전을 함께 변경하여 원하는 CUDA 버전을 사용할 수 있습니다. CUDA 버전은 사용하는 NVIDIA HPC SDK 버전과 호환되는 조합으로 설정해야 합니다.

**사용 가능한 조합**

| NVIDIA HPC SDK   | CUDA 모듈         | CUDA 버전 |
| ---------------- | --------------- | ------- |
| nvidia/25.3      | cuda/12.8       | 12.8    |
| nvidia/26.3 (기본) | cuda/12.9\_26.3 | 12.9    |
| nvidia/26.5      | cuda/13.2\_26.5 | 13.2    |

**설정 순서**

1. module purge로 기존 모듈 환경을 초기화합니다.
2. craype-arm-grace,PrgEnv-nvidia 모듈을 로드합니다.
3. 필요한 경우 module swap으로 nvidia 모듈 버전을 변경합니다.
4. 해당 버전에 맞는 cuda 모듈을 로드합니다.
5. craype-accel-nvidia90 모듈을 로드합니다. (반드시 cuda 모듈 다음에 로드합니다.)

예제 1) CUDA 12.8 사용 (NVIDIA HPC SDK 25.3)

```bash
$ module purge
$ module load PrgEnv-nvidia
$ module load craype-arm-grace
$ module swap nvidia/26.3 nvidia/25.3
$ module swap cray-mpich/9.1.0 cray-mpich/9.0.1  #Cuda 12.x는 cray-mpich/9.0.1과 호환
$ module load cuda/12.8
$ module load cray-pmi
$ module load craype-accel-nvidia90
export LD_LIBRARY_PATH=/opt/cray/pe/mpich/9.0.1/gtl/lib/:$LD_LIBRARY_PATH
```

예제 2) CUDA 12.9 사용 (기본 NVIDIA HPC SDK 26.3)

```bash
$ module purge
$ module load PrgEnv-nvidia
$ module load craype-arm-grace
$ module load cuda/12.9_26.3
$ module swap cray-mpich/9.1.0 cray-mpich/9.0.1  #Cuda 12.x는 cray-mpich/9.0.1과 호환
$ module load cray-pmi
$ module load craype-accel-nvidia90
export LD_LIBRARY_PATH=/opt/cray/pe/mpich/9.0.1/gtl/lib/:$LD_LIBRARY_PATH
```

예제 3) CUDA 13.2 사용 (NVIDIA HPC SDK 26.5)

```bash
$ module purge
$ module load PrgEnv-nvidia
$ module load craype-arm-grace
$ module swap nvidia/26.3 nvidia/26.5
$ module load cuda/13.2_26.5
$ module load cray-pmi
$ module load craype-accel-nvidia90
export LD_LIBRARY_PATH=$CUDA_HOME/compat/lib:$LD_LIBRARY_PATH  #cuda 13.x 호환 라이브러리 로드
```
{% endtab %}

{% tab title="CPU 노드" %}
#### **1) 지원 컴파일러**

CPU 노드에서는 GNU/Cray/AOCC/Intel 컴파일러를 지원합니다.

| 컴파일러  | 지원 버전                                        | 모듈                                                                |
| ----- | -------------------------------------------- | ----------------------------------------------------------------- |
| GNU   | 12.3.0, 13.3.1, 14.2.0(default)              | gcc-native/12, gcc-native/13, gcc-native/14(default)              |
| Cray  | 20.0, 21.0(default)                          | cce/20.0.0, cce/21.0.0(default)                                   |
| AOCC  | 5.0, 5.1, 5.2(default)                       | aocc/5.0.0, aocc/5.1.0, aocc/5.2.0(default)                       |
| Intel | 2021.10, 2024.2.1, 2025.3.2(default), 2026.1 | intel/2023.2.4, intel/2024.2, intel/2025.3(default), intel/2026.1 |

Intel 2021.10 버전은 intel/2023.2.4 모듈로 제공됩니다.

#### **2) 컴파일 환경 설정**

컴파일 환경은 PrgEnv-xxx 모듈로 설정합니다. MPI는 cray-mpich가 기본 및 권장 설정입니다.

&#x20;   **a. GNU 컴파일러 환경설정**

```bash
$ module load PrgEnv-gnu
$ module li
Currently Loaded Modulefiles:
  1) gcc-native/14         3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-gnu/8.7.0
```

버전 변경 (gcc v14.2를 v13.3.1로 변경)

```bash
$ module swap gcc-native/14 gcc-native/13
$ module li
Currently Loaded Modulefiles:
  1) gcc-native/13         3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-gnu/8.7.0
```

&#x20;   **b. Cray 컴파일러 환경설정**

```bash
$ module load PrgEnv-cray
$ module li
Currently Loaded Modulefiles:
  1) cce/21.0.0            3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-cray/8.7.0
```

버전 변경 (cce v21.0을 v20.0으로 변경)

```bash
$ module swap cce/21.0.0 cce/20.0.0
$ module li
Currently Loaded Modulefiles:
  1) cce/20.0.0            3) cray-dsmml/0.3.1      5) craype-network-ofi    7) cray-libsci/26.03.0
  2) craype/2.7.36         4) libfabric/2.3.1       6) cray-mpich/9.1.0      8) PrgEnv-cray/8.7.0
```

&#x20;   **c. AOCC 컴파일러 환경설정**

```bash
$ module load PrgEnv-aocc
$ module li
Currently Loaded Modulefiles:
  1) aocc/5.2.0            3) cray-mpich/9.1.0      5) PrgEnv-aocc/8.7.0     7) craype-network-ofi
  2) craype/2.7.36         4) cray-libsci/26.03.0   6) libfabric/2.3.1
```

버전 변경 (aocc v5.2를 v5.1로 변경)

```bash
$ module swap aocc/5.2.0 aocc/5.1.0
$ module li
Currently Loaded Modulefiles:
  1) aocc/5.1.0            3) cray-mpich/9.1.0      5) PrgEnv-aocc/8.7.0     7) craype-network-ofi
  2) craype/2.7.36         4) cray-libsci/26.03.0   6) libfabric/2.3.1
```

&#x20;   **d. Intel 컴파일러 환경설정**

```bash
$ module load PrgEnv-intel
$ module li
Currently Loaded Modulefiles:
  1) intel/2025.3          3) cray-mpich/9.1.0      5) PrgEnv-intel/8.7.0    7) craype-network-ofi
  2) craype/2.7.36         4) cray-libsci/26.03.0   6) libfabric/2.3.1
```

버전 변경 (v2025.3.2를 v2026.1로 변경)

```bash
$ module swap intel/2025.3 intel/2026.1
$ module li
Currently Loaded Modulefiles:
  1) intel/2026.1          3) cray-mpich/9.1.0      5) PrgEnv-intel/8.7.0    7) craype-network-ofi
  2) craype/2.7.36         4) cray-libsci/26.03.0   6) libfabric/2.3.1
```

#### **3) Compiler wrapper**

Cray PE 환경에서는 컴파일을 위해 wrapper 명령(ftn/cc/CC)을 사용합니다.

Wrapper 명령은 현재 로드된 PrgEnv-xxx 모듈에 따라 실제 컴파일러(Cray, GNU, NVIDIA 등)를 자동으로 호출하며, 함께 로드된 라이브러리 모듈(cray-mpich, cray-libsci, cray-fftw 등)의 헤더 경로와 링크 옵션도 자동으로 추가합니다. 따라서 사용자는 컴파일러 종류와 관계없이 동일한 명령을 사용할 수 있고, -I, -L, -l 옵션을 직접 지정하지 않아도 됩니다. MPI 코드 컴파일도 wrapper 명령을 그대로 사용합니다.

| Wrapper | 설명                                                             |
| ------- | -------------------------------------------------------------- |
| `ftn`   | Fortran 코드 컴파일 명령이고 GNU 환경이라면 `gfortran` or `mpif90` 대신 사용합니다. |
| `cc`    | C 코드 컴파일 명령이고 GNU 환경이라면 `gcc` or `mpicc` 대신 사용합니다.             |
| `CC`    | C++ 코드 컴파일 명령이고 GNU 환경이라면 `g++` or `mpiCC` 대신 사용합니다.           |

다음은 PrgEnv-gnu 모듈만 올린 후 wrapper 명령 ftn, cc, CC에서 사용하는 GCC 컴파일러 버전을 확인한 것입니다.

```bash
$ module load PrgEnv-gnu
$ ftn --version
GNU Fortran (SUSE Linux) 14.2.0
Copyright (C) 2024 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ cc --version
gcc-14 (SUSE Linux) 14.2.0
Copyright (C) 2024 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ CC --version
g++-14 (SUSE Linux) 14.2.0
Copyright (C) 2024 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
{% endtab %}
{% endtabs %}



### 2. 컴파일 방법 <a href="#id-1.-mpi-modules" id="id-1.-mpi-modules"></a>

{% tabs %}
{% tab title="GPU 노드" %}
#### 1) 모듈 설정 (예: NVIDIA 컴파일러 +cuda 12.9 사용)

```bash
$ module purge
$ module load PrgEnv-nvidia
$ module load craype-arm-grace
$ module load cuda/12.9_26.3
$ module swap cray-mpich/9.1.0 cray-mpich/9.0.1  #Cuda 12.x는 cray-mpich/9.0.1과 호환
$ module load cray-pmi
$ module load craype-accel-nvidia90
export LD_LIBRARY_PATH=/opt/cray/pe/mpich/9.0.1/gtl/lib/:$LD_LIBRARY_PATH
```

#### **2) 컴파일 및 링크**

* 순차 또는 MPI 코드

```bash
$ ftn -O3 mycode.f90 -o mycode.x
$ cc  -O3 mycode.c   -o mycode.x
$ CC  -O3 mycode.cpp -o mycode.x
```

* CUDA + MPI 코드

```bash
$ nvcc -O3 -arch=sm_90 -c kernel.cu -o kernel.o
$ CC   -O3 -c main.cpp -o main.o
$ CC   -O3 main.o kernel.o -cuda -o mycode.x
```

* CUDA Fortran + MPI 코드

```bash
 $ ftn -O3 -cuda -gpu=cc90 mycode.cuf -o mycode.x
```

* OpenACC + MPI 코드

```bash
$ cc  -O3 -acc -gpu=cc90 -Minfo=accel my_acc_code.c   -o acc.x
$ ftn -O3 -acc -gpu=cc90 -Minfo=accel my_acc_code.f90 -o acc.x
```

* OpenMP GPU 오프로딩 + MPI 코드

```bash
$ cc  -O3 -mp=gpu -gpu=cc90 -Minfo=mp omp_gpu.c   -o omp_gpu.x
$ ftn -O3 -mp=gpu -gpu=cc90 -Minfo=mp omp_gpu.f90 -o omp_gpu.x
```
{% endtab %}

{% tab title="CPU 노드" %}
#### **1) 모듈 설정 (예: GNU 컴파일러 사용)**

```bash
$ module load PrgEnv-gnu
$ module load craype-x86-turin
```

#### **2) 컴파일 및 링크**

* 순차 코드

```bash
$ ftn -O3 ser_code.f90 -o ser_code.x
$ cc  -O3 ser_code.c   -o ser_code.x
$ CC  -O3 ser_code.cpp -o ser_code.x
```

* MPI 코드

```bash
$ ftn -O3 mpi_code.f90 -o mpi_code.x
$ cc  -O3 mpi_code.c   -o mpi_code.x
$ CC  -O3 mpi_code.cpp -o mpi_code.x
```

* OpenMP 코드

```bash
$ ftn -O3 -fopenmp omp_code.f90 -o omp_code.x
$ cc  -O3 -fopenmp omp_code.c   -o omp_code.x
$ CC  -O3 -fopenmp omp_code.cpp -o omp_code.x
```

* OpenMP + MPI 하이브리드 코드

```bash
$ ftn -O3 -fopenmp hybrid.f90 -o hybrid.x
$ cc  -O3 -fopenmp hybrid.c   -o hybrid.x
$ CC  -O3 -fopenmp hybrid.cpp -o hybrid.x
```
{% endtab %}
{% endtabs %}

## 다. PrgEnv별 OpenMP 옵션

| PrgEnv        | 컴파일러                        | OpenMP (CPU) | OpenMP (GPU 오프로딩)                 |
| ------------- | --------------------------- | ------------ | --------------------------------- |
| PrgEnv-cray   | CCE                         | -fopenmp     | -fopenmp (+craype-accel-nvidia90) |
| PrgEnv-gnu    | GCC                         | -fopenmp     | 권장하지 않음                           |
| PrgEnv-aocc   | AOCC                        | -fopenmp     | 지원하지 않음 (CPU 노드 전용)               |
| PrgEnv-intel  | Intel oneAPI (icx/icpx/ifx) | -qopenmp     | 지원하지 않음 (CPU 노드 전용)               |
| PrgEnv-nvidia | NVHPC                       | -mp          | -mp=gpu                           |

AOCC와 Intel 컴파일러는 CPU 노드에서만 제공되며, NVIDIA GPU 대상 OpenMP 오프로딩은 지원하지 않습니다. Intel 컴파일러는 -fiopenmp도 사용할 수 있습니다.
