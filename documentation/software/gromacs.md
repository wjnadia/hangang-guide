# GROMACS

## 가. GROMACS version

* **GROMACS 2025.4**
* 소스 다운로드 및 압축 해제

```bash
wget https://ftp.gromacs.org/gromacs/gromacs-2025.4.tar.gz
tar xzf gromacs-2025.4.tar.gz
cd gromacs-2025.4
```

## 나. 빌드

### 1. configure files & build

빌드 대상별 configure 파일을 소스 최상위 디렉터리에 두고, 별도의 build 디렉터리에서 실행합니다.

| 구분          | 컴파일러 환경           | 주요 모듈                                         | SIMD     | FFT              | 설치 경로                              |
| ----------- | ----------------- | --------------------------------------------- | -------- | ---------------- | ---------------------------------- |
| CPU (AMD)   | PrgEnv-cray (CCE) | craype-x86-turin, cray-fftw                   | AVX\_512 | cray-fftw        | `~/GROMACS_CPU/bin/cray_base`      |
| CPU (Grace) | PrgEnv-gnu        | craype-arm-grace, cray-mpich, cray-fftw       | AUTO     | cray-fftw        | `~/GROMACS_CPU/bin/gnu_grace_base` |
| GPU (GH200) | PrgEnv-gnu + CUDA | craype-accel-nvidia90, cudatoolkit/25.3\_12.8 | AUTO     | 자체 빌드(OWN\_FFTW) | `~/GROMACS_GPU/bin/gnu_base`       |

#### 1) CPU (AMD)

configure 파일: `configure_cray_base`

```bash
#!/bin/bash
# HPE Cray CCE via Cray wrappers
# Using modules:
# PrgEnv-cray/8.6.0  craype-x86-turin  cray-fftw
module purge
export LOADEDMODULES=""
module load PrgEnv-cray/8.6.0
module load craype-x86-turin
module load cray-fftw

# pip install --user "cmake==4.1.3"
export PATH=$HOME/.local/bin:$PATH

export CC=cc
export CXX=CC
export FC=ftn

cmake .. \
 -DCMAKE_INSTALL_PREFIX=$HOME/GROMACS_CPU/bin/cray_base \
 -DCMAKE_BUILD_TYPE=Release \
 -DCMAKE_PREFIX_PATH=$FFTW_ROOT \
 -DGMX_BUILD_OWN_FFTW=OFF \
 -DGMX_FFT_LIBRARY=fftw3 \
 -DGMX_MPI=ON \
 -DGMX_OPENMP=ON \
 -DGMX_CYCLE_SUBCOUNTERS=ON \
 -DGMX_GPU=OFF \
 -DGMX_SIMD=AVX_512 \
 -DGMX_USE_RDTSCP=ON \
 -DGMX_HWLOC=OFF \
 -DGMX_OPENMP_MAX_THREADS=512 \
 -DGMX_INSTALL_NBLIB_API=OFF \
 -DGMXAPI=OFF
```

빌드 및 설치

```bash
module purge
module load PrgEnv-cray/8.6.0
module load craype-x86-turin
module load cray-fftw

mkdir build_cray_base && cd build_cray_base
../configure_cray_base 2>&1 | tee config.log
make -j8 2>&1 | tee make.log
make install
```

#### 2) CPU (Grace)

configure 파일: `configure_gnu_grace_base`

```bash
#!/bin/bash
# GNU on NVIDIA Grace (Neoverse V2) - CPU only, BASE
module purge
export LOADEDMODULES=""
module load PrgEnv-gnu/8.6.0
module load craype-arm-grace
module load cray-mpich/9.0.1
module load cray-fftw
module load cmake/4.2.3

export CC=cc
export CXX=CC
export FC=ftn
# CFLAGS/CXXFLAGS 미지정 (BASE)

cmake .. \
  -DCMAKE_INSTALL_PREFIX=$HOME/GROMACS_CPU/bin/gnu_grace_base \
  -DCMAKE_C_COMPILER=cc \
  -DCMAKE_CXX_COMPILER=CC \
  -DMPI_C_COMPILER=cc \
  -DMPI_CXX_COMPILER=CC \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH=$FFTW_ROOT \
  -DGMX_BUILD_OWN_FFTW=OFF \
  -DGMX_FFT_LIBRARY=fftw3 \
  -DGMX_GPU=OFF \
  -DGMX_MPI=ON \
  -DGMX_OPENMP=ON \
  -DGMX_SIMD=AUTO \
  -DGMX_CYCLE_SUBCOUNTERS=ON \
  -DGMX_HWLOC=OFF \
  -DGMX_OPENMP_MAX_THREADS=512 \
  -DGMX_INSTALL_NBLIB_API=OFF \
  -DGMXAPI=OFF
```

빌드 및 설치

```bash
mkdir build_gnu_grace_base && cd build_gnu_grace_base
../configure_gnu_grace_base 2>&1 | tee config.log
make -j32 2>&1 | tee make.log
make install
```

#### 3) GPU (GH200)

configure 파일: `configure_gnu_gpu_base`

```bash
#!/bin/bash
module purge
export LOADEDMODULES=""
module load PrgEnv-gnu
module load craype-arm-grace
module load craype-accel-nvidia90
module load cudatoolkit/25.3_12.8
module load cray-mpich/9.0.1
module load cmake/4.2.3

export CC=cc
export CXX=CC
export FC=ftn
# CFLAGS/CXXFLAGS 미지정 (BASE)

cmake .. \
 -DCMAKE_INSTALL_PREFIX=$HOME/GROMACS_GPU/bin/gnu_base \
 -DCMAKE_C_COMPILER=cc \
 -DCMAKE_CXX_COMPILER=CC \
 -DCMAKE_CUDA_HOST_COMPILER=$(which g++-14) \
 -DCMAKE_CUDA_FLAGS="-I${MPICH_DIR}/include" \
 -DMPI_C_COMPILER=cc \
 -DMPI_CXX_COMPILER=CC \
 -DGMX_GPU=CUDA \
 -DCMAKE_CUDA_ARCHITECTURES=90 \
 -DGMX_MPI=ON \
 -DGMX_OPENMP=ON \
 -DGMX_SIMD=AUTO \
 -DGMX_BUILD_OWN_FFTW=ON \
 -DGMX_FFT_LIBRARY=fftw3 \
 -DGMX_INSTALL_NBLIB_API=OFF \
 -DCMAKE_BUILD_TYPE=Release
```

빌드 및 설치

```bash
mkdir build_gnu_gpu_base && cd build_gnu_gpu_base
../configure_gnu_gpu_base 2>&1 | tee config.log
make -j32 2>&1 | tee make.log && make install
```

### 2. 설치 확인

설치가 끝나면 `<INSTALL_PREFIX>/bin` 아래에 실행 파일 `gmx_mpi`와 환경 설정 스크립트 `GMXRC`가 생성됩니다. (예: CPU-AMD)

```bash
$ ls ~/GROMACS_CPU/bin/cray_base/bin/
total 292
./ ../ demux.pl gmx-completion.bash gmx-completion-gmx_mpi.bash gmx_mpi GMXRC GMXRC.bash GMXRC.csh GMXRC.zsh xplor2gmx.pl
```

## 다. 실행 방법

### 1. 파티션 정보

| 노드 종류                | 노드명   | 파티션          | 최대 실행 시간 |
| -------------------- | ----- | ------------ | -------- |
| CPU (AMD EPYC Turin) | cnode | `normal_cpu` | 1일       |
| CPU (AMD EPYC Turin) | cnode | `long_cpu`   | 2일 12시간  |
| GPU (GH200)          | node  | `normal`     | 1일       |

* 작업 제출: `sbatch <job script>`
* 빌드나 step 수를 바꿔 실행할 때: `sbatch --export=ALL,BUILD=cray_base,NSTEPS=50000 <job script>`

### 2. CPU (AMD)

```bash
#!/bin/bash
##########################################
# GROMACS 2025.4 BMT - Hangang CPU node (AMD EPYC Turin, CPU only)
# STMV (1,066,628 atoms, 2fs time-step)
#   TIP3P / CHARMM27, 1.2 nm cut-offs, h-bond constraints,
#   0.15 nm PME grid spacing, NVT ensemble
##########################################
#SBATCH -J GROMACS_STMV_N01_C256_OMP01
#SBATCH -p normal_cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=256       # MPI rank 수/노드
#SBATCH --cpus-per-task=1           # OpenMP thread 수/rank
#SBATCH --exclusive
#SBATCH -o %x.%j.log
#SBATCH -e %x.%j.log
#SBATCH --time=01:00:00
#SBATCH --comment="field=mse;appl=gromacs"

ulimit -s unlimited

# build 종류: cray_base | gnu_base | gnu_opt | aocc_base | aocc_opt | ...
BUILD=${BUILD:-cray_base}

module purge
export LOADEDMODULES=""
module load PrgEnv-cray/8.6.0
module load craype-x86-turin
module load cray-fftw
export GMX_INSTALL=$HOME/GROMACS_CPU/bin/${BUILD}
export PATH=${GMX_INSTALL}/bin:$PATH
gmxBin="${GMX_INSTALL}/bin/gmx_mpi"
WorkloadPath=$HOME/GROMACS_heterogeneous_parallelization_benchmark_info_and_systems_JCP/stmv

NSTEPS=${NSTEPS:-50000}             # BMT용

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export OMP_PLACES=cores
export OMP_PROC_BIND=close
export GMX_MAXBACKUP=-1             # 반복 실행 시 #md.log.1# 백업 파일 생성 방지

NRANKS=$(( SLURM_NNODES * SLURM_NTASKS_PER_NODE ))
TAG=${BUILD}_N${SLURM_NNODES}_R${SLURM_NTASKS_PER_NODE}_T${OMP_NUM_THREADS}

cd ${SLURM_SUBMIT_DIR}

time -p srun --cpu-bind=cores \
  ${gmxBin} mdrun \
  -notunepme -ntomp ${OMP_NUM_THREADS} -pin on -pinstride 1 \
  -pme cpu -nb cpu -bonded cpu -update cpu -dlb yes \
  -v -resethway -nsteps ${NSTEPS} \
  -s ${WorkloadPath}/topol.tpr -noconfout \
  -g md_${TAG}_${SLURM_JOB_ID}.log

grep -E "Performance:" md_${TAG}_${SLURM_JOB_ID}.log
```

* 2일 이상 실행이 필요한 경우 `-p long_cpu`를 사용합니다.

### 3. GPU (GH200)

```bash
#!/bin/bash
##########################################
# GROMACS 2025.4 BMT - Hangang GH200, STMV (1,066,628 atoms)
# Full GPU offloading, 1 rank * 72 OpenMP threads  (1 rank = 1 GPU)
##########################################
#SBATCH -J GMX_STMV_GPU
#SBATCH -p normal
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=72
#SBATCH --gpus-per-node=4
#SBATCH --exclusive
#SBATCH -o %x.%j.log
#SBATCH -e %x.%j.log
#SBATCH --time=01:00:00
#SBATCH --comment="field=mse;appl=gromacs"

# 실행 시 변경 가능한 변수
BUILD=${BUILD:-gnu_base}     # gnu_base | gnu_opt | ...
NSTEPS=${NSTEPS:-500000}
EXTRA=${EXTRA:-}             # 추가 mdrun 옵션 (예: "-nstlist 300")

module purge
module load PrgEnv-gnu craype-arm-grace craype-accel-nvidia90
module load cudatoolkit/25.3_12.8 cray-mpich/9.0.1

GMX=$HOME/GROMACS_GPU/bin/${BUILD}/bin/gmx_mpi
TPR=$HOME/GROMACS_heterogeneous_parallelization_benchmark_info_and_systems_JCP/stmv/topol.tpr

ulimit -s unlimited
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
export GMX_MAXBACKUP=-1
# Direct GPU communication
export MPICH_GPU_SUPPORT_ENABLED=1
export GMX_FORCE_GPU_AWARE_MPI=1
export GMX_ENABLE_DIRECT_GPU_COMM=1

# 다중 rank: PME 전용 rank 1개 + 동적 로드밸런싱
NRANKS=${SLURM_NTASKS}
MULTI=""
[ ${NRANKS} -gt 1 ] && MULTI="-npme 1 -dlb yes"

LOG=md_${BUILD}_N${SLURM_NNODES}_G${NRANKS}_${SLURM_JOB_ID}.log

cd ${SLURM_SUBMIT_DIR}
echo "BUILD=${BUILD} NODES=${SLURM_NNODES} RANKS=${NRANKS} OMP=${OMP_NUM_THREADS} NSTEPS=${NSTEPS} EXTRA=${EXTRA}"

# 바인딩은 GROMACS(-pin on)가 담당 → srun은 바인딩하지 않음
# Full GPU offloading, PME 튜닝 ON (-notunepme 없음)
srun --cpu-bind=none \
  ${GMX} mdrun -s ${TPR} -g ${LOG} \
    -nb gpu -pme gpu -bonded gpu -update gpu ${MULTI} \
    -ntomp ${OMP_NUM_THREADS} -pin on \
    -resethway -nsteps ${NSTEPS} -noconfout -cpt -1 -v ${EXTRA}

grep "Performance:" ${LOG}
grep -iE "optimal|cut-off.*grid" ${LOG} | tail -3     # PME 튜닝 최종 결과
```

### 4. 결과 확인

성능은 로그 파일 끝의 `Performance:` 줄에서 확인합니다. 단위는 ns/day와 hour/ns입니다.

```bash
grep "Performance:" md_*.log
```
