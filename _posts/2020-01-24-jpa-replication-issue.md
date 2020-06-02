---
title: Spring JPA DB Master / Slave 적용시 주의할 점
description: JPA환경 DB Replication 설정시 이슈 정리.
categories:
 - dev
tags:
 - JPA
 - Spring
---

스프링 JPA를 사용하면서 데이터 베이스 이중화를 하는 경우에 크게 두가지 방법으로 구현한다. <br>
첫번째는 AOP 를 이용하여 쿼리 타이밍에 특정한 Datasource를 불러오는 방법. <br>
두번째는 Transaction의 Readonly 속성을 감지하여 이중화. <br>

지금 진행중인 프로젝트에서 두번째 방법으로 데이터베이스 이중화를 설정하였지만 생각하지 못한 문제점이 발생. 원인파악 및 최소한의 해결 방법을 정리.

- 문제점
  - DataSource를 여러개를 사용하기 때문에 일반적으로 스프링 JPA 설정으로 데이터소스를 만들수 없다. 커스텀한 데이터 소스들을 서버가 실행될때 등록하여 실행한다.
  - 위 방법을 사용시 스프링에서 제공해주는 기본 설정을 사용하지 못하여 @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class}) 와 같이 기본설정을 제거 하는 코드가 필요.
  - 기본설정을 제거 하여 Entity Manager룰 새롭게 등록 해주어야 했다.
  - LocalContainerEntityManagerFactoryBean 를 사용하여 EntityManager를 등록 하니 아무런 설정들이 없는상태.
  - 스프링 JPA를 사용하면 hibernate naming 전략이 snake case로 설정된다. 하지만 자동설정을 못하니 naming 전략이 camel case로 설정 되어 실행되고 있었다.
  - 문제점을 해결하기 위해 property에 naming 전략을 정의 해봤지만 정상적으로 실행되지 않았다. 구글링 결과 아래 코드를 사용하여 해결할 수 있었다.
  ```java
        entityManagerFactoryBean.setDataSource(dataSource());
        entityManagerFactoryBean.setPackagesToScan("com.my.pakage");
        entityManagerFactoryBean.setJpaVendorAdapter(vendorAdapter);

        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.physical_naming_strategy", "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
        entityManagerFactoryBean.setJpaPropertyMap(properties);
  ```

- 남은 문제
  - 스프링 자동 설정이 정확히 어떤것들을 설정해 주는지 모르니 기본적으로 사용하던 설정들중 어떤것들이 빠져 있는지 확인이 필요하다.
  - 가능하다면 스프링에서 제동해주는 자동설정을 그대로 사용하고 싶기 때문에 방법을 조금더 찾아 볼 예정. 
- 문제 해결 추가.
  - 스프링 자동 설정중 테이블 네이밍 설정이 빠져 있는것을 확인. <br>
  hibernate.implicit_naming_strategy / org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy 추가.
  - 스프링에서 제공하는 자동 설정이 어떤것들이 있는지 라이브러리에서 확인. <br>
  org.springframework.boot.autoconfigure.orm.jpa 해당 패키지 아래 jpa, hibernate autoconfiguration을 확인할 수 있다.


개발환경
* 스프링 부트 2.2.2 
* 스프링 데이터 JPA 2.2.2
