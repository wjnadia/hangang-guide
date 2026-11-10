# 스케줄러(Slurm)를 통한 작업 실행

Hangang 시스템 작업 스케쥴러는 SLURM을 사용합니다. 이 장에서는 SLURM을 통해 작업 제출하는 방법 및 관련 명령어들을 소개합니다. SLURM에 작업 제출을 위한 작업 스크립트 작성법은 작업스크립트 파일 작성 예시를 참고 부탁드립니다.

<mark style="color:red;">**※ 사용자 작업은 /scratch/$USER와 /appsdata/$USER 에서만 제출 가능합니다.**</mark>

## 가. 큐 구성

### 1. 베타서비스 기간 큐 구성

<table data-search="false"><thead><tr><th width="69.81982421875" align="center">구분</th><th width="113.96142578125" align="center">Queue Name</th><th width="167.5360107421875" align="center">Node Range</th><th width="77" align="center">Max Active Jobs</th><th width="83.17919921875" align="center">Max Running Jobs</th><th width="71.718505859375" align="center">Cores /node</th><th width="84.24462890625" align="center">WallTime</th><th width="68.006103515625" align="center">GPU /node</th><th width="101.556884765625" align="center">Mem(GiB) /node</th></tr></thead><tbody><tr><td rowspan="3" align="center">GPU</td><td align="center">normal</td><td align="center">node[0001-1250]</td><td align="center">5</td><td align="center">5</td><td align="center">288</td><td align="center">24h</td><td align="center">4</td><td align="center">849.6</td></tr><tr><td align="center">natStr</td><td align="center">node[1251-1772]</td><td align="center">5</td><td align="center">5</td><td align="center">288</td><td align="center">24h</td><td align="center">4</td><td align="center">849.6</td></tr><tr><td align="center">5thEx</td><td align="center">node[1773-1876]</td><td align="center">5</td><td align="center">5</td><td align="center">288</td><td align="center">24h</td><td align="center">4</td><td align="center">849.6</td></tr><tr><td rowspan="4" align="center">CPU</td><td align="center">normal_cpu</td><td align="center">cnode[0001-0684]</td><td align="center">5</td><td align="center">5</td><td align="center">256</td><td align="center">24h</td><td align="center">-</td><td align="center">753.4</td></tr><tr><td align="center">long_cpu</td><td align="center">cnode[0685-0764]</td><td align="center">5</td><td align="center">5</td><td align="center">256</td><td align="center">60h</td><td align="center">-</td><td align="center">753.4</td></tr><tr><td align="center">5thEx_cpu</td><td align="center">cnode[0765-0804]</td><td align="center">5</td><td align="center">5</td><td align="center">256</td><td align="center">24h</td><td align="center">-</td><td align="center">753.4</td></tr><tr><td align="center">hugemem</td><td align="center">bigsmp[01-02]</td><td align="center">2</td><td align="center">1</td><td align="center">1024</td><td align="center">24h</td><td align="center">-</td><td align="center">47,607.4</td></tr></tbody></table>

<mark style="color:red;">**※ 노드 구성/파티션은 시스템 사용량에 따라 시스템 운영 중에 조정될 수 있습니다.**</mark>

### 2. 큐 소개

<table data-search="false"><thead><tr><th width="163.396484375" align="center">큐이름</th><th width="591.6759033203125" align="center">소개</th></tr></thead><tbody><tr><td align="center">normal</td><td align="center">GPU를 활용하고자 하는 베타서비스 사용자를 위한 큐</td></tr><tr><td align="center">natStr</td><td align="center">GPU를 활용하고자 하는 혁신지원사업 및 국가전략 프로그램 사용자를 위한 큐</td></tr><tr><td align="center">5thEx</td><td align="center">GPU를 활용하고자 하는 슈퍼컴퓨터 5호기 유료 사용자를 위한 큐[무상]</td></tr><tr><td align="center">normal_cpu</td><td align="center">CPU를 활용하고자 하는 베타서비스 사용자를 위한 큐</td></tr><tr><td align="center">long_cpu</td><td align="center">CPU를 활용하고자 하는 베타서비스 사용자를 위한 큐[Walltime 60시간]</td></tr><tr><td align="center">5thEx_cpu</td><td align="center">CPU를 활용하고자 하는 슈퍼컴퓨터 5호기 유료 사용자를 위한 큐[무상]</td></tr><tr><td align="center">hugemem</td><td align="center">대용량 메모리가 필요한 베타서비스 사용자를 위한  큐</td></tr></tbody></table>

## 나. SLURM 스케줄러에서 사용하는 기본 6가지 명령어

|    명령어    |             설명            |
| :-------: | :-----------------------: |
|  `sinfo`  |     파티션(큐) 및 노드 정보 확인     |
|  `sbatch` |     작업 스크립트 제출 (배치 모드)    |
|  `salloc` |  작업 생성 및 명령 실행 (인터랙티브 모드) |
|  `squeue` |        제출 작업 상태 확인        |
|   `srun`  | 작업 실행 또는 제출 (`mpirun` 대체) |
| `scancel` |         제출된 작업 삭제         |

***

※ sinfo --help 명령어를 이용하여 sinfo의 옵션을 확인하실 수 있습니다.

## 다. AI/HPC 분야(field) 및 프로그램(appl) 정보 입력

<mark style="color:red;">※</mark> <mark style="color:red;">**Hangang 시스템의 사용자 활용 현황을 보다 정확하게 분석하고, AI/HPC 응용 분야별 자원 이용 통계 및 운영 정책 수립에 활용하기 위해  아래와 같이 SBATCH 옵션을 통한 응용 분야(field)와 사용 프로그램 (appl) 정보 작성을 의무화 합니다. 즉, 사용하는 분야와 어플리케이션에 맞게 SBATCH의 --comment 옵션을 다음 형식으로 반드시**</mark> <mark style="color:red;">**기입한 후 작업을 제출해야 합니다.**</mark>

```vim
#SBATCH --comment="field=<분야>;appl=<프로그램>"  
```

<mark style="color:red;">**※ 분야(field)와 프로그램(appl)에는 허용되는 값만 입력할 수 있으며 허용되는 값은 아래 분야별 SBATCH 옵션 이름표, 프로그램별 SBATCH 옵션 이름표를 참조하거나 로그인 노드에서 아래와 같은 명령어를 이용해 확인 가능합니다.**</mark>

```bash
$ showappl
```

<mark style="color:red;">**※ 어플리케이션 구분을 추가는 주기적으로 수집된 사용자 요구에 맞추어 진행됩니다. 추가를 원하시면**</mark> [<mark style="color:red;">**consult@ksc.re.kr로**</mark>](mailto:consult@ksc.re.kr%EB%A1%9C) <mark style="color:red;">**해당 어플리케이션에 대한 추가 요청을 해주시기 바랍니다.**</mark>

### **\[AI/HPC 분야(field) 리스트]**

<table data-search="false"><thead><tr><th width="103.91145833333331" align="center">구분</th><th width="163.3203125" align="center">분야(field)</th><th>분야 설명</th></tr></thead><tbody><tr><td rowspan="13" align="center">AI</td><td align="center"><strong>agenticai</strong></td><td>Agentic AI</td></tr><tr><td align="center"><strong>robotics</strong></td><td>Physical AI &#x26; Robotics</td></tr><tr><td align="center"><strong>aim</strong></td><td>Autonomous Driving &#x26; AI Mobility</td></tr><tr><td align="center"><strong>nlp</strong></td><td>Natural Language Processing, Large Language Models</td></tr><tr><td align="center"><strong>cv</strong></td><td>Computer Vision &#x26; Spatial Intelligence</td></tr><tr><td align="center"><strong>speech</strong></td><td>Speech, Audio AI</td></tr><tr><td align="center"><strong>recsys</strong></td><td>Recommendation System &#x26; Decision Making</td></tr><tr><td align="center"><strong>multimodal</strong></td><td>Multimodal, Generative AI, World Model, Media Synthesis</td></tr><tr><td align="center"><strong>sciai</strong></td><td>AI for Science &#x26; Graph AI</td></tr><tr><td align="center"><strong>efficientai</strong></td><td>Efficient &#x26; Scalable AI Systems, Quantization, Edge AI</td></tr><tr><td align="center"><strong>healthai</strong></td><td>Healthcare AI</td></tr><tr><td align="center"><strong>finai</strong></td><td>Finance AI</td></tr><tr><td align="center"><strong>misc</strong></td><td>Miscellaneous</td></tr><tr><td rowspan="23" align="center">HPC</td><td align="center"><strong>math</strong></td><td>Mathematics</td></tr><tr><td align="center"><strong>phys</strong></td><td>Physics</td></tr><tr><td align="center"><strong>chem</strong></td><td>Chemistry</td></tr><tr><td align="center"><strong>aac</strong></td><td>Astrophysics and Cosmology</td></tr><tr><td align="center"><strong>ees</strong></td><td>Earth and Environmental Science</td></tr><tr><td align="center"><strong>bio</strong></td><td>Bio/life science</td></tr><tr><td align="center"><strong>med</strong></td><td>Medical/Health science</td></tr><tr><td align="center"><strong>bns</strong></td><td>Brain and Neurosciences</td></tr><tr><td align="center"><strong>agf</strong></td><td>Agriculture, Fishery and Food</td></tr><tr><td align="center"><strong>qst</strong></td><td>Quantum Science and Technology</td></tr><tr><td align="center"><strong>mse</strong></td><td>Material Science and engineering</td></tr><tr><td align="center"><strong>ere</strong></td><td>Energy and Resources engineering</td></tr><tr><td align="center"><strong>mae</strong></td><td>Mechanical and Aerospace engineering</td></tr><tr><td align="center"><strong>ne</strong></td><td>Nuclear engineering</td></tr><tr><td align="center"><strong>che</strong></td><td>Chemical Engineering</td></tr><tr><td align="center"><strong>eee</strong></td><td>Electrical and electronic engineering</td></tr><tr><td align="center"><strong>ict</strong></td><td>Information/Communication Technology</td></tr><tr><td align="center"><strong>cse</strong></td><td>Computer science and engineering</td></tr><tr><td align="center"><strong>cte</strong></td><td>Civil and transportation engineering</td></tr><tr><td align="center"><strong>ss</strong></td><td>Social Science</td></tr><tr><td align="center"><strong>caa</strong></td><td>Culture and Arts</td></tr><tr><td align="center"><strong>hum</strong></td><td>Humanities</td></tr><tr><td align="center"><strong>misc</strong></td><td>Miscellaneous</td></tr></tbody></table>

### **\[AI/HPC 프로그램(appl) 리스트]**

<table data-search="false"><thead><tr><th width="83.203125" align="center">구분</th><th>프로그램명(appl)</th><th>프로그램명(appl)</th><th>프로그램명(appl)</th></tr></thead><tbody><tr><td rowspan="5" align="center">AI</td><td><strong>pytorch</strong></td><td><strong>jax</strong></td><td><strong>triton</strong></td></tr><tr><td><strong>pytorch-ddp</strong></td><td><strong>tensorflow</strong></td><td><strong>jupyter</strong></td></tr><tr><td><strong>pytorch-fsdp</strong></td><td><strong>tensorrt</strong></td><td><strong>r</strong></td></tr><tr><td><strong>deepspeed</strong></td><td><strong>ollama</strong></td><td><strong>misc</strong></td></tr><tr><td><strong>megatron-lm</strong></td><td><strong>vllm</strong></td><td></td></tr><tr><td rowspan="18" align="center">HPC</td><td><strong>abaqus</strong></td><td><strong>grims</strong></td><td><strong>q-chem</strong></td></tr><tr><td><strong>amber</strong></td><td><strong>gromacs</strong></td><td><strong>qmcpack</strong></td></tr><tr><td><strong>ansys</strong></td><td><strong>kflow</strong></td><td><strong>quantum_espresso</strong></td></tr><tr><td><strong>berkeleygw</strong></td><td><strong>lammps</strong></td><td><strong>r</strong></td></tr><tr><td><strong>bwa</strong></td><td><strong>ls-dyna</strong></td><td><strong>ramses</strong></td></tr><tr><td><strong>castep</strong></td><td><strong>materials_studio</strong></td><td><strong>raspa</strong></td></tr><tr><td><strong>cesm</strong></td><td><strong>matlab</strong></td><td><strong>roms</strong></td></tr><tr><td><strong>cgyro</strong></td><td><strong>mfdn</strong></td><td><strong>rosetta</strong></td></tr><tr><td><strong>charmm</strong></td><td><strong>mom</strong></td><td><strong>siesta</strong></td></tr><tr><td><strong>cp2k</strong></td><td><strong>mpas</strong></td><td><strong>star-ccm</strong></td></tr><tr><td><strong>fhi_aims</strong></td><td><strong>msc_one</strong></td><td><strong>su2</strong></td></tr><tr><td><strong>fvcom</strong></td><td><strong>namd</strong></td><td><strong>vasp</strong></td></tr><tr><td><strong>gadget</strong></td><td><strong>nw_chem</strong></td><td><strong>wannier90</strong></td></tr><tr><td><strong>gatk</strong></td><td><strong>openfoam</strong></td><td><strong>wien2k</strong></td></tr><tr><td><strong>gaussian</strong></td><td><strong>openmm</strong></td><td><strong>wrf</strong></td></tr><tr><td><strong>geant4</strong></td><td><strong>openmx</strong></td><td><strong>xtb</strong></td></tr><tr><td><strong>gotpm</strong></td><td><strong>orca</strong></td><td><strong>in_house</strong></td></tr><tr><td><strong>gpaw</strong></td><td><strong>pyfr</strong></td><td><strong>misc</strong></td></tr></tbody></table>

## 라. 작업 실행

{% tabs %}
{% tab title="1.GPU 노드" %}
#### 1) 작업 스크립트(`job_gpu.sh`) 작성 예

```bash
#!/bin/bash
#SBATCH --time=00:10:00          # Set the maximum wall-clock time to 10 minutes (HH:MM:SS).
#SBATCH --partition=normal       # Submit the job to the GPU partition/queue.
#SBATCH --nodes=2                # Request 2 compute node.
#SBATCH --ntasks-per-node=4      # Launch 4 MPI tasks (ranks) on each node.
#SBATCH --cpus-per-task=8        # Allocate 8 CPU cores to each task (e.g., for OpenMP threads).
#SBATCH --gres=gpu:4             # Request 4 GPUs per node as a generic resource.
#SBATCH --exclusive              # Allocate the node(s) exclusively; do not share with other jobs.
#SBATCH --comment="field=[field];appl=[appl]" # Set field & application, please run 'showappl' command on the login node`
#SBATCH -J 8GPU_job              # Set the slurm job name to "8GPU_job"

module purge
module load craype-arm-grace
module load PrgEnv-nvidia
module swap nvidia/26.5           # for nvidia 26.5 instead of default version
module load cuda/13.2_26.5        # for cuda v13.2
module load craype-accel-nvidia90

srun -N 2 -n 8 ./my_gpu_code      # You should use srun instead of mpirun!
```

#### 2) 작업 제출 및 확인

```bash
$ sbatch job_gpu.sh   # job submit
$ squeue              # check the status of the job
```
{% endtab %}

{% tab title="2.CPU 노드" %}
#### 1) 작업 스크립트(`job_cpu.sh`) 작성 예

```bash
#!/bin/bash
#SBATCH --time=00:10:00          # Set the maximum wall-clock time to 10 minutes (HH:MM:SS).
#SBATCH --partition=normal_cpu   # Submit the job to the CPU partition/queue.
#SBATCH --nodes=4                # Request 4 compute node.
#SBATCH --ntasks-per-node=16     # Launch 16 MPI tasks (ranks) on each node.
#SBATCH --cpus-per-task=8        # Allocate 8 CPU cores to each task (e.g., for OpenMP threads).
#SBATCH --exclusive              # Allocate the node(s) exclusively; do not share with other jobs.
#SBATCH --comment="field=[field];appl=[appl]" # Set field & application, please run 'showappl' command on the login node`
#SBATCH -J 4node_cpu_job         # Set the slurm job name to "4node_cpu_job"

module purge
module load craype-x86-turin
module load PrgEnv-gnu

srun -N 4 -n 64 ./my_mpi_code    # You should use srun instead of mpirun!
```

#### 2) 작업 제출 및 확인

<pre class="language-bash"><code class="lang-bash"><strong>$ sbatch job_cpu.sh   # job submit
</strong>$ squeue              # check the status of the job
</code></pre>


{% endtab %}
{% endtabs %}

## 마. 인터랙티브 작업 수행

{% tabs %}
{% tab title="1. GPU노드" %}
#### 1) 기본 형식

```
salloc -p [파티션 정보] -N [노드 수] --exclusive -t [HH:MM:SS] --comment="field=[분야];appl=[응용프로그램]" -J [작업 이름]
```

#### 2) 사용예: GPU노드 1개, GPU 1장

```
salloc -p [GPU 파티션] -N 1 --gres=gpu:1 -t 01:00:00 --comment="field=[분야];appl=[응용프로그램]" -J interactive_gpu
```
{% endtab %}

{% tab title="2. CPU노드" %}
#### 1) 기본 형식

```
salloc -p [파티션 정보] -N [노드 수] --exclusive -t [HH:MM:SS] \       --comment="field=[분야];appl=[응용프로그램]" -J [작업 이름]
```

#### 2)  사용예: **CPU 노드 1개, 2시간 독점 사용**

```
salloc -p [CPU 파티션] -N 1 --exclusive -t 02:00:00 \       --comment="field=misc;appl=misc" -J interactive_cpu
```
{% endtab %}
{% endtabs %}



## 바. sbatch 옵션 설명

| 옵션                      | 설명                            | 예시                                 |
| ----------------------- | ----------------------------- | ---------------------------------- |
| `-p`, `--partition`     | 사용할 파티션                       | `-p [파티션 정보]`                      |
| `-N`, `--nodes`         | 할당받을 노드 수                     | `-N 2`                             |
| `-n`, `--ntasks`        | 전체 MPI 프로세스(task) 수           | `-n 144`                           |
| `--ntasks-per-node`     | 노드당 task 수                    | `--ntasks-per-node=72`             |
| `-c`, `--cpus-per-task` | task당 CPU 코어 수 (OpenMP 스레드용)  | `-c 4`                             |
| `--mem`                 | 노드당 메모리                       | `--mem=100G`                       |
| `--exclusive`           | 노드를 다른 작업과 공유하지 않고 독점 사용      | `--exclusive`                      |
| `-t`, `--time`          | 최대 사용 시간 (파티션 최대값 이하)         | `-t 02:00:00`                      |
| `-J`, `--job-name`      | 작업 이름 (`squeue`에 표시됨)         | `-J interactive_cpu`               |
| `--comment`             | 분야/응용 정보 (**필수**)             | `--comment="field=misc;appl=misc"` |
| `--gres=gpu:[N]`        | 노드당 GPU 수 (GPU 파티션)           | `--gres=gpu:4`                     |
| `--gpus-per-node`       | 노드당 GPU 수 (`--gres` 대신 사용 가능) | `--gpus-per-node=4`                |
| `-w`, `--nodelist`      | 특정 노드 지정                      | `-w [노드명]`                         |
| `-x`, `--exclude`       | 특정 노드 제외                      | `-x [노드명]`                         |
