---
title: 스프링 부트 정리
description: 백기선님 강의 스프링 부트 개념과 활용 정리 노트
categories:
 - dev
tags:
 - Java
 - Spring
---
> 백기선님 강의 스프링 부트 개념과 활용 정리 노트
강의 내용을 다시한번 복습하기 위하여 정리합니다.

작성중인 문서입니다.

스프링 부트란 독립적이며, 프로덕션 레벨의 스프링 기반 어플리케이션을 간편하게 만들수 있게 도와주는 툴입니다. 

## 스프링 부트 원리

* 의존성 관리 이해
  * 스프링 부트에서는 자동으로 의존성을 관리 해준다.
  * 일반적으로 spring-boot-stater-parent에 정의되어 있는 라이브러리는 버전을 명시하지 않아도 현재 사용하는 스프링 부트 버전에 맞게 자동으로 버전을 찾아 적용한다.
  * parent에 등록되지 않은 라이브러리를 사용할 때는 버전을 명시해줘야 한다.
  * 특정 버전을 사용하고 싶을땐 버전을 명시한게 우선순위를 가지게 된다.
* 의존성 관리 응용
  * 의존성 추가
    * 스프링 부트가 관리 해주지않는 라이브러리는 버전을 명시해주는것이 좋다.
  * 의존성 변경
    * 스프링 부트가 관리해주는 버전도 변경 할 수 있다.
* 자동 설정 이해
  * @EnableAutoConfiguration (@SpringBootApplication 안에 포함되어 있다.)
  * Spring Bean은 사실 두 단계로 나눠서 생성된다.
    * 1단계 : @ComponentScan
    * 2단계 : @EnableAutoConfiguration
  * @ComponentScan
    * @Component
    * @Configuration @Repository @Service @Controller @RestController
  * EnableAutoConfiguration
    * spring.factories
      * org.springframework.boot.autoconfigure.EnableAutoConfiguration
    * @Configuration
    * @ConditionalOnXxxYyyZzz
