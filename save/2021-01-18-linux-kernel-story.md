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


1장 
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