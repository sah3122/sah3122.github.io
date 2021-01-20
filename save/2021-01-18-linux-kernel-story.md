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

