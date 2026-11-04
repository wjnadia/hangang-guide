# LAMMPS

## 가. 설치 방법

{% tabs %}
{% tab title="GPU 활용" %}
**1. 소스 다운로드**

```bash
git clone https://github.com/lammps/lammps.git
cd lammps   # 다운 받은 소스 경로로 이동
git checkout stable_30Sep2026
```

**2. 모듈 설정 (NVIDIA 컴파일러 + CUDA 12.8)**

```bash
module purge
module load PrgEnv-nvidia
module load craype-arm-grace
module swap nvidia/26.3 nvidia/25.3
module swap cray-mpich/9.1.0 cray-mpich/9.0.1   # CUDA 12.x는 cray-mpich/9.0.1 과 호환
module load cray-pmi
module load cuda/12.8
module load craype-accel-nvidia90
export LD_LIBRARY_PATH=/opt/cray/pe/mpich/9.0.1/gtl/lib/:$LD_LIBRARY_PATH
```

**3. build 디렉터리 생성 및 설치 경로 설정**

```bash
mkdir build    # 이 경로에서 컴파일 진행됨, 재 빌드시 build 내용 삭제 후 진행해야 함
cd build
INSTALL_PATH=<설치할 경로>
NVPL_HOME=/apps/NVIDIA/hpc_sdk/Linux_aarch64/25.3/math_libs/nvpl   # CUDA 12.8
```

**4. cmake 실행**

```bash
GCC_TC=/opt/cray/pe/gcc-native/12

# nvcc probes the host compiler without CMAKE_CXX_FLAGS, so --gcc-toolchain
# must be baked into the compiler command itself. Generate a CC wrapper.
cat > $PWD/CC-gcc <<EOF
#!/bin/bash
exec $(which CC) --gcc-toolchain=${GCC_TC} "\$@"
EOF
chmod +x $PWD/CC-gcc

cmake ../cmake \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_PATH \
  -DCMAKE_C_COMPILER=cc \
  -DCMAKE_CXX_COMPILER=$PWD/CC-gcc \
  -DCMAKE_CXX_FLAGS="-O3 -mcpu=neoverse-v2 -Mfprelaxed" \
  -DMPI_CXX_FOUND=TRUE \
  -DMPI_C_FOUND=TRUE \
  -DCMAKE_PREFIX_PATH=$NVPL_HOME \
  -DFFT=NVPL \
  -DBUILD_MPI=yes \
  -DPKG_OPT=yes \
  -DPKG_OPENMP=yes \
  -DPKG_KSPACE=yes \
  -DPKG_MOLECULE=yes \
  -DPKG_RIGID=yes \
  -DPKG_GRANULAR=yes \
  -DPKG_MANYBODY=yes \
  -DPKG_GPU=yes \
  -DGPU_API=cuda \
  -DGPU_ARCH=sm_90 \
  -DGPU_PREC=mixed \
  -DPKG_KOKKOS=yes \
  -DFFT_KOKKOS=CUFFT \
  -DKokkos_ENABLE_CUDA=on \
  -DKokkos_ENABLE_OPENMP=on \
  -DKokkos_ARCH_ARMV9_GRACE=on \
  -DKokkos_ARCH_HOPPER90=on \
  -DKokkos_CUDA_COMPILER=nvcc \
  -DCUDA_HOST_COMPILER=$(which CC)
```

**5. 빌드 및 설치**

```bash
make -j $(nproc)
make install
```
{% endtab %}

{% tab title="CPU 활용" %}
**1. 소스 다운로드**

```bash
git clone https://github.com/lammps/lammps.git
cd lammps   # 다운 받은 소스 경로로 이동
git checkout stable_30Sep2026
```

**2. 모듈 설정 (GNU 컴파일러)**

```bash
module purge
module load craype-x86-turin
module load PrgEnv-gnu
module load cray-fftw
```

**3. build 디렉터리 생성 및 설치 경로 설정**

```bash
mkdir build    # 이 경로에서 컴파일 진행됨, 재 빌드시 build 내용 삭제 후 진행해야 함
cd build
INSTALL_PATH=<설치할 경로>
```

**4. cmake 실행**

```bash
cmake ../cmake \
  -DCMAKE_INSTALL_PREFIX=$INSTALL_PATH \
  -DCMAKE_C_COMPILER=cc \
  -DCMAKE_CXX_COMPILER=CC \
  -DCMAKE_Fortran_COMPILER=ftn \
  -DCMAKE_CXX_FLAGS="-O3 -march=native -ffp-contract=fast -flto=auto -funroll-loops" \
  -DFFT=FFTW3 \
  -DFFTW3_INCLUDE_DIR=$FFTW_ROOT/include \
  -DFFTW3_LIBRARY=$FFTW_ROOT/lib/libfftw3.so \
  -DBUILD_MPI=yes \
  -DPKG_OPT=yes \
  -DPKG_OPENMP=yes \
  -DPKG_KSPACE=yes \
  -DPKG_MOLECULE=yes \
  -DPKG_RIGID=yes \
  -DPKG_GRANULAR=yes \
  -DPKG_MANYBODY=yes \
  -DPKG_KOKKOS=yes \
  -DKokkos_ENABLE_OPENMP=yes \
  -DKokkos_ARCH_ZEN5=on
```

**5. 빌드 및 설치**

```bash
make -j $(nproc)
make install
```
{% endtab %}
{% endtabs %}

## 나. 실행방법

{% tabs %}
{% tab title="GPU 활용" %}
### **1 . 실행 (8 Node, 4 GPU/node, 64 Procs/node, 2 Threads/proc)**

```bash
#!/bin/bash
#SBATCH --time=00:10:00        # Set the maximum wall-clock time to 10 minutes.
#SBATCH --partition=normal        # Submit the job to the GPU partition/queue.
#SBATCH --exclusive
#SBATCH --nodes=8              # Request 8 compute nodes.
#SBATCH --ntasks-per-node=64   # Run 64 MPI tasks on each node (512 tasks total).
#SBATCH --cpus-per-task=2      # 2 OpenMP threads per MPI task.
#SBATCH --gres=gpu:4           # 4 GPUs per node.
#SBATCH -J GPU_8N              # Set the Slurm job name to "GPU_8N".
#SBATCH -o log32g_%j.out       # Write stdout to file log32g_<jobid>.out.
#SBATCH --comment="field=mse;appl=lammps"

#cd /path/to/work
LAMMPS_DIR=$INSTALL_PATH
export LD_LIBRARY_PATH=$LAMMPS_DIR/lib:$LD_LIBRARY_PATH

NPROCS=$((SLURM_NNODES*SLURM_NTASKS_PER_NODE))

export MPICH_GPU_SUPPORT_ENABLED=1
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export OMP_PROC_BIND=spread
export OMP_PLACES=threads

srun $OPTS -n $NPROCS --cpu-bind=cores $LAMMPS_DIR/bin/lmp -sf gpu -pk gpu 4 omp $SLURM_CPUS_PER_TASK -var x 4 -var y 4 -var z 4 -in ./in.rhodo.scaled
```
{% endtab %}

{% tab title="CPU 활용" %}
## **1. 실행 (1 Node, 128 Procs/node, 2 Threads/proc)**

```bash
#!/bin/bash
#SBATCH --time=00:10:00
#SBATCH --partition=normal_cpu
#SBATCH --exclusive
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=2
#SBATCH --comment="field=mse;appl=lammps"

LAMMPS_DIR=$INSTALL_PATH
export LD_LIBRARY_PATH=$LAMMPS_DIR/lib:$LD_LIBRARY_PATH

export OMP_PLACES=threads
export OMP_PROC_BIND=spread
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun --cpus-per-task=$SLURM_CPUS_PER_TASK --distribution=block:block --cpu-bind=cores --mem-bind=local $LAMMPS_DIR/bin/lmp_mpi -sf omp -pk omp 0 -var x 1 -var y 1 -var z 1 -in ./in.rhodo.scaled
```
{% endtab %}
{% endtabs %}

