---
title: DevOps와 SE를 위한 리눅스 커널 이야기 정리
description: DevOps와 SE를 위한 리눅스 커널 이야기 책을 읽고 내용을 정리한 글
categories:
 - dev
tags:
 - linux
 - devops
comments: true
---


1장 - 시스템 구성 정보 확인하기
커널 정보 확인하기
uname 
- 설치되어 있는 커널의 버전 관련 정보를 확인

dmesg 
- 커널의 디버그 메시지 확인

CPU정보 확인하기
dmidecode
- 하드웨어 정보 확인하기
- bios, system, processor 주로 사용
- memory 명령어로 메모리 정보 확인

디스크 정보확인하기
df 명령어 사용
smartctl 툴 사용

네트워크 정보 확인
lspci 네트워크 카드 정보 확인
ethtool 연결 상태 확인

정리
1. dmidecode 명령을 통해 CPU, 메모리, BIOS 등의 정보를 확인할 수 있다.
2. CPU 정보는 /proc/cpuinfo 파일을 통해 확인할 수 있다.
3. free 명령을 통해 메모리의 전체 크기 및 메모리 사용 정보를 알 수 있다.
4. 시스템에 마운트된 블록 디바이스의 정보는 df 명령을 통해 확인할 수 있다.
5. 네트워크 카드 정보는 ethtool 명령을 통해 확인 그중 -g, -k, i 옵션을 많이 사용.

2장 - top을 통해 살펴보는 프로세스 정보

시스템의 상태 살피기 
top 
VIRT - task가 사용하는 virtual memory의 전체 용량
RES - 현재 task가 사용하고 있는 물리 메모리
SHR - 다른 프로세스와 공유하고 있는 shared memory의 양
Memory Commit - 시스템이 메모리를 요청했을때 가용 영역이 존재하면 메모리의 주소를 전달해준다. 하지만 이때까지는 실제로 물리 메모리가 할당된것이 아니고 실제 쓰기 작업이 시작될때 page fault가 일어난다. 

프로세스의 상태 보기
D : uninterruptible sleep, 디스크 혹은 네트워크 I/O를 대기 Run Queue가 아닌 Wait Queue에 포함
R : 실행중인 프로세스
S : sleeping 상태의 프로세스, D 상태와 차이점은 요청 리소스를 즉시 사용할 수 있는지 여부
T : traced or stopped, strace 등으로 프로세스의 시스템 콜을 추적하고 있는 상태
Z : zombie 상태의 프로세스, 부모 프로세스가 죽은 자식 프로세스를 의미

프로세스의 우선순위
PR과 NI를 통해 알 수 있다. 
실제 동작은 CPU 마다 Run Queue에 우선순위가 정해져있음. 스케줄러에 의해 선택. 
PR - 프로세스의 실제 우선순위 값
NI - nice value, 명령어를 통해 우선순위를 낮출떄 사용.

RT 스케줄러는 특정 시간안에 종료되어야 하는 중요한 프로세스, 즉 커널에서 사용하는 데몬들이 대상인 프로세스들을 관리. 

정리
1. top 명령으로 시스템의 CPU, Memory, swap의 사용량 및 상태를 알 수 있다.
2. top 명령의 결과 중 VIRT는 프로세스에 할당된 가상 메모리, RES는 물리 메모리, SHR은 다른 프로세스와 공유하는 메모리.
3. 커널은 프로세스가 요청할 때 곧바로 물리 메모리를 할당해주지 않는다. memory commit에 의해 실제 작업이 일어나는 순간 할당. 
4. vm.overcommit_memory는 커널의 memory commit 동작 방식을 변경할 수 있게 해준다. 
5. top으로 확인 가능한 프로세스의 상태중 D는 I/O대기중, R은 실제 실행중인 프로세스, S는 sleep상태, T는 tracing, Z는 Zombie 상태이다. 
6. 프로세스에는 우선순위가 존재 (PR), 우선순위를 조절하는 값도 존재 (NI)

3장 Load Average와 시스템 부하

Load Average의 정의
- 프로세스의 상태 중 R과 D 상태에 있는 프로세스 개수의 1분, 5분, 15분 마다의 평균값. 
- Load Average가 높으면 프로세스가 많이 실행되거나 I/O를 처리하기 위해 대기하고 있다는것을 의미한다. 
- 프로세스의 수를 수치로 나타낸것이기 떄문에 곧이 곧대로 받아들이기 보단 현재 CPU Core의 수에 따라 의미가 달라진다. 
  - 프로세스 2개 실행, CPU 1개인 경우 LoadAverage는 2지만 하나의 프로세스는 항상 대기, 반면 프로세스 2개, CPU 2개인 경우 동시에 프로세스가 실행되는 구조이다. 

Load Average 계산 과정
uptime 명령어를 사용하여 현재 값을 확인할 수 있다. 
![uptime](/assets/images/post/linux-kernel/linux-kernel01.png)

실제 연산과정을 확인하려면 커널의 Load Average계산 함수를 따라가면된다. 
복잡하지는 않지만 정리하자면 CPU의 Run Queue에 있는 nr_running 프로세스의 개수와 nr_uninterruptible 프로세스의 개수를 세어 1, 5, 15분 마나의 평균을 계산하는 구조이다. 

CPU Bound vs I/O Bound
Load Average가 높다는 것은 단순히 CPU를 사용하려는 프로세스가 많다는것을 의미하는것이 아니고, I/O에 병목이 생겨 대기 프로세스가 많다고도 볼 수 있다. 
Load Average로만으로는 시스템의 상태를 정확히 진단할 수 없다.

vmstat으로 부하의 정체 확인하기
Load Average값은 시스템에 부하가 있다는것을 알려주지만, 구체적인 정보는 알 수없다. 
vmstat으로 시스템의 상태를 확인해볼수 있다. 
- r : 실행되기를 기다리거나 현재 실행중 프로세스
- b : I/O를 위해 대기열에 있는 프로세스

Load Average가 시스템에 끼치는 영향
우리가 돌리는 프로세스가 어떤 시스템 자원을 더 많이 쓰는지에 따라(I/O or CPU) 시스템에 영향끼치는 정도가 다르다. 
ex) IO 부하가 많은 프로세스가 실행되면 그렇지 않은 프로세스가 CPU 경합을 이기기 쉽고, CPU 부하가 많은 경우 경합에서 이기기 힘들다.

Case Study - OS 버전과 Load Average 
커널은 완벽하지 않기 떄문에 OS 버전에 따라 버그 또는 이슈가 존재할 수 있다. 
그렇기 때문에 버전 차이로 인한 성능 이슈를 항상 염두해두어야 한다. 
/proc/sched_debug : proc파일 시스템에 있는 파일, 각 CPU의 Run Queue 상태와 스케줄링 정보 확인

정리 
1. Load Average는 실행 중 혹은 실행 대기 중이거나 I/O 작업 등을 위해 대기 큐에 있는 프로세스들의 수를 기반으로 만들어진 값. 
2. Load Average 자체의 절대적인 높은과 낮음은 없으며, 현재 시스템에 장착되어 있는 CPU 코어를 기반으로 한 상대적인 값으로 해석. 
3. 커널에도 버그가 존재, Load Average 값을 절대적으로 신뢰해선 안됨.
4. vmstat 툴로 시스템의 부하를 측정 가능. r, b 열을 눈여겨 봐야함. 
5. /proc/sched_debug는 vm_stat 툴을 통해 확인할 수 있는 것보다 더 자세한 정보 제공. 

4장 free 명령이 숨기고 있는 것들

메모리 사용량 확인하기
free 명령은 전체 메모리 용량과, 사용 중인 용량, buffers와 cached로 명명되는 캐싱 영역의 용량을 확인할때 사용한다.

Buffers와 cached 영역
커널은 블록 디바이스라고 부르는 디스크로부터 데이터를 읽거나, 저장한다. 디스크에 읽기, 쓰기 작업은 매우 느리고 리로스를 많이 사용하기 때문에
디스크에 대한 요청을 빠르게 처리하기 위하여 메모리의 일부를 디스크 요청에 대한 캐싱역역으로 할당하여 사용한다. 이때 사용되는 캐싱영역을 
buffers, cached라고 부른다.
PageCache는 파일의 내용을 저장하는 캐시, Buffer Cache는 파일 시스템의 메타 데이터를 담고있는 캐시라고 정리할 수 있다. 
각각이 free 명령에서 나타내는 cached, buffers 영역이다. 
서버 동작 초기엔 가용영역과 사용영역만 사용하다 가용 / 캐시 / 사용영역을 사용하게 됨. 
시간이 더 지나고, 메모리의 사용이 많아지면 가용영역이 점차 줄어지다, 더이상 줄이지 못할때 캐시 영역을 반환하여 사용하게 됨. 
이런 과정이 반복되다 더이상 반환하지 못하게 되는 순간 Swap이라는 영역을 사용하고, 성능의 저하가 발생한다. 

/proc/meminfo 읽기
free 명령어는 메모리에 대한 통계를 볼수 있지만 시스템의 어느 부분에서 메모리가 사용되는지는 알수 없다. 
조금더 자세한 정보를 알기 위해 /proc/meminfo를 사용한다.

SwapCached영역은 swap으로 빠진 메모리영역중 다시 메모리로 돌아온 영역을 의미. (Swap영역이 다시 메모리로 이동될때 swap영역을 곧바로 삭제하지는 않는다. (다시 사용할 수 있기 때문))
Active(anon) : Page Cache 영역을 제외한 영역, 비교적 최근에 할당된 영역
Inactive(anon) : 참조된지 오래되어 swap으로 이동될 수 있는 영역
Active(file) : 커널의 I/O 성능향상을 위해 사용하는 영역, buffers와 cached 영역이 여기 속함. 
Inactive(file) : 참조된지 오래되어 swap으로 이동될 수 있는 영역
Dirty : I/O 성능 향상을 위해 커널이 캐시 목적으로 사용하는 영역중 쓰기 작업이 이루어지고 실제 블록 디바이스의 블록에 씌어져야할 영역. 

active 메모리는 시간이 지나서 inactive로 이동하는것이 아니고, 가용 메모리가 부족한 상황에서 LRU 알고리즘에 인해 이동한다. 

slab 메모리 영역
slab : 커널이 직접적으로 사용하는 메모리 영역. dentry cache(디렉토리 계층관계 캐싱), inode cache(파일의 inode 정보) 등 커널이 사용하는 메모리가 포함
Sreclaimable: slab 영역 중 재사용가능한 영역. 캐시용도로 사용하는 메모리들이 주로 포함된다. 메모리가 부족한 경우 해제되어 프로세스에 할당될 영역
SUnreclaim: slab 영역중 재사용될수 없는 영역. 커널이 현재 사용하고 있는 영역이다. 

정리
free명령어와 /proc/meminfo를 통해 시스템 메모리의 전반적인 상태확인 방법을 알아보았다. 
1. free - buffers는 파일 시스템의 메타 데이터 등을 저장하는 블록 디바이스의 블록을 위한 캐시이다. 
2. free - cached는 I/O 작업의 효율성을 위해 파일의 내용을 저장하는 영역이다. 
3. buffers와 cached 영역은 시스템의 효율성의 위하여 커널에서 사용하는 용도이며, 프로세스가 필요로 할때는 해당 영역을 반환하게 된다. 
4. /proc/meminfo에서 보이는 anon영역은 프로세스가 사용하는 영역, file은 i/o 성능 향상을 위해 커널이 캐싱용도로 사용하는 영역이다. 
5. anon과 file영역은 active, inactive라 불리는 LRU Listfmf xhdgo rhksflehlsek. 
6. Slab 영역은 커널이 사용하는 캐싱영역을 의미한다. 