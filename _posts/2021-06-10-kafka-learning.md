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
  * kafka-console-producer --topic stylerId --bootstrap-server localhost:9092 --property "parse.key=true" --property "key.separator=:"
    * parse.key를 true로 두면 레코드 전송시 메시지 키 추가 가능
    * separator : 메시지 키와 값을 구분하는 구분자 선언, default는 tab이다.

* 컨슈머 실행
  * kafka-console-consumer --topic stylerId --from-beginning --bootstrap-server localhost:9092 --property print.key=true --property key.separator="-" --group hello-group
    * 메시지 키를 확인하기 위해 print.key를 true로 설정
    * 메시지 키 값을 구분하기 위해 separator 설정
    * group 옵션을 통해 신규 컨슈머 그룹 생성

## 정리
* Producer로 전달되는 레코드 값은 UTF-8 기반 Byte로 변환 되며 ByteArraySerializer로만 직렬화 된다. 즉 String이 아닌 다른 값으로 직렬화 하여 전송하지 못한다.  
  다른 타입으로 직렬화 하여 전송하고 싶다면 커스텀 프로듀서를 개발해야 한다. 
* key-value 쌍의 레코드 중 key 가 같은 경우는 동일한 파티션으로 전송된다. 
* 커밋이란 컨슈머가 특정 레코드까지 처리를 안료 하였다고 레코드의 오프셋 번호를 카프카 브로커에 저장하는 것이다. 커밋 정보는 __consumer_offsets 이름의 내부 토픽에 저장된다. 
* 토픽의 데이터를 가져가는 방법은 모든 파티션으로 부터 동일한 중요도로 가져간다. 따라서 토픽에 넣은 데이터의 순서를 보장하고 싶다면 파티션 1개로 구성된 토픽을 만드는것이다. 