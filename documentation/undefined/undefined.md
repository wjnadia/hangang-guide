# 사용자간 데이터 공유 방법

**`setfacl`**(Set File Access Control Lists)은 **특정 사용자나 그룹에게 개별적으로 파일/디렉터리 권한을 부여**할 때 사용하는 명령어로 사용자간 데이터를 공유할 때 사용할 수 있습니다.

### 1. 기본사용법 <a href="#id-1" id="id-1"></a>

```
setfacl [옵션] [설정내용] [파일/디렉터리명]
```

옵션

* **`-m`** (modify)**:** 권한을 추가하거나 수정 (가장 많이 사용)
* **`-x`** (remove)**:** 특정 사용자나 그룹의 접근 권한 삭제
* **`-b`** (remove all)**:** 설정된 모든 접근 권한을 제거&#x20;
* **`-R`** (recursive)**:** 하위 디렉터리와 파일까지 재귀적으 적용

설정내용

* **사용자** u:\[사용자명]:\[권한]
* **그룹**       g:\[그룹명]:\[권한]



### 2. 사용예시 <a href="#id-2" id="id-2"></a>

#### &#x20; **가. 특정 사용자(USER)에게 권한 부여**

```
형식: u:[사용자명]:[권한]            # 권한: r(읽기)w(쓰기)x(실행)
```

```
# user01에게 test.txt 파일의 읽기(r) 및 쓰기(w) 권한 부여
setfacl -m u:user01:rw test.txt​

# user02에게 project 디렉터리의 읽기, 쓰기, 실행 권한을 하위까지 모두 부여
setfacl -R -m u:user02:rwx project/
```

getfacl 명령어를 사용해 부여된 권한을 확인할 수 있습니다.

```
getfacl test.txt
```

#### &#x20; **나. 특정 그룹에게 권한 부여**

```
형식: g:[그룹명]:[권한]              # 권한: r(읽기)w(쓰기)x(실행)
```

```
# team01 그룹에게 sample.sh 파일의 읽기(r) 및 실행(x) 권한 부여
setfacl -m g:team01:rx sample.sh
```

#### &#x20; **다. user01 사용자가 /scratch/user01 디렉토리를 user02사용자에게 권한 부여**

```
1. user01 계정으로 로그인
2. cd /scratch
3. setfacl -m u:user02:rx user01    # rwx로 설정하면 읽기/쓰기/실행
```

#### &#x20; **라. user01 사용자가 /scratch/user01/shared 디렉토리를 user02사용자에게 권한 부여**

```
1. user01 계정으로 로그인
2. cd /scratch                      # /scratch 경로로 이동
3. setfacl -m u:user02:x user01     # 하위 shared디렉토리 이외는 접근하지 못하도록 상위디렉토리에 x만 부여
4. cd user01                        # 현재경로: /scratch/user01
5. setfacl -m u:user02:rwx shared   # user02에게 shared 디렉토리에 읽기/쓰기/실행 권한 부여
```



### 3. 권한 삭제 방법 <a href="#id-3" id="id-3"></a>

```
# test.txt에서 user02에게 주었던 권한만 타겟팅하여 삭제
setfacl -x u:user02 test.txt​

# test.txt에 설정된 '모든' 권한을 초기화 (기본 권한 상태로 복구)
setfacl -b test.txt
```
