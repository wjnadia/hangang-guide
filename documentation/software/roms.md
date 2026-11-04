# ROMS

## 가. ROMS version

* **ROMS v3.9 (svn r1054)**
* 소스 다운로드 (다운로드 권한을 받으려면 [myroms.org](https://www.myroms.org/) 포럼에 문의해야 할 수 있습니다.)

```bash
export ROMS_HOME=/scratch/${USER}/ROMS/ROMS3.9      # 설치할 경로로 변경
svn checkout -r 1054 https://www.myroms.org/svn/src/trunk ${ROMS_HOME}
```

## 나. 설치 환경

```bash
module purge
module load PrgEnv-gnu
module load craype-x86-turin
module load cray-mpich
module load cray-netcdf-hdf5parallel
module load cray-parallel-netcdf
```

## 다. 설치 방법

참고 자료: [ROMS Wiki - build\_roms](https://www.myroms.org/wiki/build_roms)

### 1. build script 준비

sample build script를 복사한 뒤 Hangang 환경에 맞게 수정합니다.

```bash
cd ${ROMS_HOME}
cp ROMS/Bin/build_roms.sh ./build_roms_hangang_gnu.sh
```

`build_roms_hangang_gnu.sh`에서 반드시 확인할 항목은 다음과 같습니다.

| 항목                 | 내용                              |
| ------------------ | ------------------------------- |
| module             | 2장의 설치 환경과 같은 모듈을 로드            |
| `MY_ROOT_DIR`      | 작업 최상위 디렉터리                     |
| `ROMS_ROOT_DIR`    | ROMS 소스 경로 (`${ROMS_HOME}`)     |
| `ROMS_APPLICATION` | 실행할 application 이름              |
| `FORT`             | Fortran 컴파일러 (`ftn` wrapper 사용) |
| `FFLAGS`           | 컴파일 최적화 옵션                      |

### 2. 빌드

```bash
chmod +x build_roms_hangang_gnu.sh
./build_roms_hangang_gnu.sh -j 8 2>&1 | tee build.log
```

### 3. 빌드 확인

로그에 별도의 성공 메시지가 없으므로 아래와 같이 확인합니다.

```bash
# 오류 개수 확인: 0 이면 오류 없음
grep "^Error" build.log | wc -l

# 실행 파일(romsM) 생성 확인
ls -al bin_gnu/
```

## 라. 실행 방법

### 1. tiling 설정

input file(`*.in`)의 tiling 값을 실행에 사용할 MPI 태스크 수에 맞게 수정합니다. `NtileI × NtileJ`가 전체 MPI 태스크 수와 같아야 합니다.

| 노드 수 | MPI 태스크 수 | NtileI × NtileJ (예) |
| ---- | --------- | ------------------- |
| 1    | 256       | 16 × 16             |
| 4    | 1024      | 32 × 32             |

### 2. job script (CPU, 4노드 1024 코어 예시)

```bash
#!/bin/bash
#SBATCH --job-name=ROMS3.9_HANGANG_MAIN_N04_1024P
#SBATCH --partition=normal_cpu
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=256
#SBATCH --ntasks=1024
#SBATCH --cpus-per-task=1
#SBATCH --time=00:10:00
#SBATCH --output=%x_%j.out
#SBATCH --error=%x_%j.out
#SBATCH --comment="field=ees;appl=roms"
#SBATCH --exclusive

set -euo pipefail

echo "===================================================="
echo " ROMS NWP BMT test on Hangang"
echo " Nodes      : ${SLURM_JOB_NUM_NODES}"
echo " Total tasks: ${SLURM_NTASKS}"
echo " Job ID     : ${SLURM_JOB_ID}"
echo " Start time : $(date)"
echo "===================================================="

module purge
module load PrgEnv-gnu
module load gcc-native/14
module load craype-x86-turin
module load cray-mpich
module load cray-hdf5-parallel
module load cray-netcdf-hdf5parallel
module load cray-parallel-netcdf

module list

ROMS_EXE=/scratch/${USER}/ROMS/ROMS3.9/bin_gnu/romsM

RUN_PATH=/scratch/${USER}/ROMS/BMT_TEST/ROMS_OUT_LOGS/Hanagang_Main/gnuo3znver5_1024P_4NODE
ROMS_IN=${RUN_PATH}/roms_N04.in          # 사용할 input file로 변경

cd "${RUN_PATH}"
mkdir -p ncout
rm -f rst_*.nc                           # 이전 실행의 restart 파일 삭제 (필요한 경우)

echo "실행파일: ${ROMS_EXE}"
echo "입력파일: ${ROMS_IN}"
echo "MPI 태스크 수: ${SLURM_NTASKS}  (${ROMS_IN} 의 NtileI x NtileJ 와 일치해야 함)"

time -p srun --unbuffered "${ROMS_EXE}" "${ROMS_IN}"

echo "===================================================="
echo " End time : $(date)"
echo "===================================================="
```

* 작업 제출: `sbatch <job script>`
* 노드 수를 바꿀 때는 `--nodes`, `--ntasks`, input file의 tiling, `ROMS_IN`을 함께 수정합니다.
* 2일 이상 실행이 필요한 경우 `--partition=long_cpu`를 사용합니다.
