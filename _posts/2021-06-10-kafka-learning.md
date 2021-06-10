---
title: 카프카 학습 내용 정리
description: 아파치 카프카 애플리케이션 프로그래밍 with 자바 책 정리 및 카프카 내용 정리
categories:
 - dev
tags:
 - kafka
comments: true
---


## 명령어

* 주키퍼 실행 
  * zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties

* 카프카 실행
  * kafka-server-start /usr/local/etc/kafka/server.properties

* 토픽 생성
  * kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 —partitions 3 —config retention.ms=172800000 --topic mytopic2  
    replication-factor : 파티션 복제본 갯수 (최대 설정값은 브로커의 갯수이다.)  
    Partitions  : 파티션 갯수 (최소 1개, default config/server.properties > num.partitions 값)  
    retention.ms : 데이터 유지 기간 

* 토픽 조회
  * kafka-topics --bootstrap-server localhost:9092 --list
 
* 토픽 상세 조회
  * kafka-topics --bootstrap-server localhost:9092 --describe --topic mytopic

* 토픽 옵션 수정
* 파티션 갯수 수정시 kafka-topic 이용, 리텐션 기간 변경시 kafka-config 이용  
  * kafka-topics --bootstrap-server localhost:9092 --topic mytopic --alter --partitions 2  
    파티션의 갯수를 2개로 변경. (늘릴수는 있지만 줄일순 없음)
  * kafka-configs --bootstrap-server localhost:9092 --entity-type topics --entity-name mytopic --alter --add-config retention.ms=86400000
    리텐션 기간 변경

* 프로듀서 실생
  * kafka-console-producer --topic stylerId --bootstrap-server localhost:9092

* 컨슈머 실행
  * kafka-console-consumer --topic stylerId --from-beginning --bootstrap-server localhost:9092

