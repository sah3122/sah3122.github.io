---
title: JPA Dirty Checking 사용시 주의점
description: JPA를 사용한 업데이트 연산을 믿지 말자.
categories:
 - dev
tags:
 - java
 - jpa
comments: true
---

이번 포스팅에서는 간단하지만 쉽게 놓칠 수 있는 부분을 소개해드리려고 합니다. :)

어느 날 API 모니터링 중 유난히 응답속도가 오래 걸리는 API를 발견하게 되었고, 어떤 작업을 수행하는 API인지 찾아보게 되었습니다. 

API의 기능을 간략하게 정리하자면 아래와 같은 기능을 수행하고 있었습니다. 
사용자의 읽지 않은 알림을 모두 가지고 온다. 
모두 읽음 처리한다. 

기능만 봐서는 간단한 작업들이고, 전혀 문제 될 부분들이 없어 보였지만 그동안 JPA에 익숙해짐으로써 중요한 부분들을 놓치고 있다는 것을 알게 되었습니다. 

문제의 코드를 확인해보겠습니다. 

```java
@Transactional
public ViewCountDto viewBadgeAll(UserAccount userAccount) {
    List<Notification> notifications = notificationRepository.findUnBadgeviewNotifications(userAccount.getId());
    notifications.forEach(Notification::viewBadge);
    return new ViewCountDto(notifications.size());
}
```

위 코드만 봐서는 크게 문제 될 부분이 없어 보일 수도 있습니다. 

> 만약 JPA를 잘 모르시거나 Dirty Checking이란 기능을 모르시는 분들을 위해 간략히 설명드리자면 JPA에는 영속성 컨텍스트라는 개념과, 1차 캐시라는 개념이 있습니다. 
  비즈니스 로직 실행 중 영속성 컨텍스트가 관리하는 도메인 객체의 값이 변경되고, 트랜젝션이 커밋되는 순간 Update Query가 실행되게 되는데 이러한 기능을 Dirty Checking이라고 합니다. 


영속성 컨텍스트를 중심적으로 다시 한번 위 코드를 분석해 보겠습니다. 
1. 알림 엔티티를 조회한다. (영속성 컨텍스트에 저장한다.)
2. 알림 엔티티를 각각 수정한다. 
3. 수정된 알림 엔티티들을 업데이트한다. 

어떤 문제점이 발생할지 감이 오시는 분들이 계실 것 같습니다. 
위 코드를 실행하는 경우 Notifications의 크기만큼 Update Query가 실행되고 있었습니다. 

이러한 문제점들을 바로잡기 위하여 코드를 개선하는 작업을 진행하였고 현재는 아래와 같이 동작하도록 수정하였습니다. 

```java
@Transactional
public ViewCountDto viewBadgeAll(UserAccount userAccount) {
    List<Long> notificationIds = notificationRepository.findUnBadgeviewNotifications(userAccount.getId())
                                                       .stream()
                                                       .map(Notification::getId)
                                                       .collect(toList());
    final AtomicInteger count = new AtomicInteger();
    notificationIds.stream()
                   .collect(groupingBy(notificationId -> count.getAndIncrement() / CHUNK_SIZE))
                   .values()
                   .forEach(this::bulkUpdate);
    return new ViewCountDto(notificationIds.size());
}


private void bulkUpdate(List<Long> notificationIds) {
    notificationRepository.updateNotificationsViewByIds(notificationIds);
}

@Override
public void updateNotificationsViewByIds(List<Long> ids) {
    jpaQueryFactory.update(notification)
                   .set(notification.badgeView, true)
                   .where(notification.id.in(ids))
                   .execute();
}
```

개선된 코드를 간략히 설명드리겠습니다. 
1. 읽지 않은 알림을 조회한다. 
2. 알림 리스트를 정의한 사이즈만큼의 Chunk로 분할한다. 
3. 분할된 리스트마다 Id를 추출하고 Update In 쿼리를 실행한다. 

이와 같이 개선함으로 실행되는 Update Query의 빈도의 전 / 후 차이를 살펴보면 개선 전 로직의 경우
Notification의 크기만큼 Update 쿼리가 실행된 반면 개선 후 로직의 경우는 Notification Size / 1000 만큼의 Update Query가 실행되도록 개선되었습니다. 


간략한 팁이지만 Chunk Size를 1000개로 설정한 이유로는 Mysql In query는 기본적으로 1000개까지만 입력 가능하도록 되어 있기 때문에 Chunk Size를 1000개로 설정하였습니다.
또한 Bulk Update를 실행하는 경우 영속성 컨텍스트의 관리를 받지 못하게 되므로 업데이트 연산 이후 엔티티를 가지고 후 처리가 필요한 경우에는 엔티티를 다시 조회해야 한다는 점을 반드시 알고 계셔야 합니다. :)

지금까지 JPA를 사용하며 Dirty Chekcing이라는 편리한 기능에 익숙해져 성능과는 거리가 멀어진 코드를 리팩토링하는 과정이었습니다. 
