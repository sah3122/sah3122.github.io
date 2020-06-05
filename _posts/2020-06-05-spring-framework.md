---
title: 스프링 프레임워크 핵심 정리
description: 백기선님 강의 스프링프레임워크 핵심 기술 정리 노트
categories:
 - dev
tags:
 - Java
 - Spring
---
> 백기선님 강의 스프링프레임워크 핵심 기술 정리 노트

작성중인 문서입니다.

# IoC 컨테이너

## 스프링 IoC 컨테이너와 빈
Inverse of Controll : 의존 관계 주입(Dependency Injection)이라고 하며 어떤 객체가 사용하는 의존객체를 직접 생성하지 않고  
외부로부터 주입 받아 사용하는 방법  
의존 관계 주입에 대한 장점을 더 알아보시려면 Open Close Principal에 대하여 공부해보시는 것을 추천 드립니다.  

* 스프링 IoC 컨테이너
  * BeanFactory : 스프링 빈 컨테이너에 접근하기 위한 최상위 인터페이스
  * 애플리케이션 컴포넌트의 중앙 저장소
  * 빈 설정소스로 부터 빈 정의, 빈을 구성하고 제공하는 역할을 한다.
* Bean이란 ?
  * 스프링 IoC 컨테이너가 관리하는 객체
  * 장점
    * 의존성 관리
    * 스코프
      * 싱글톤 : 흔히 알고 있는 싱글톤 패턴과 같이 하나만 생성
      * 프로포토 타입 : 빈이 생성될 때 마다 매번 다른 객체를 생성하는 방법
      ```java
        @Component
        @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)

        @Bean
        @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
      ```
* ApplicationContext
  * BeanFactory를 상속받고, 스프링에서 제공하는 다양한 기능을 사용할 수 있는 인터페이스
    ```java
    public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
    ```
  * 메시지 소스 처리 (i18n)
  * 이벤트 발생 기능 
  * 리소스 로딩 기능
  * ...

## ApplicationContext와 다양한 빈 설정 방법

* 스프링 IOC 컨테이너의 역할 
  * 빈 인스턴스 생성
  * 의존 관계 설정
  * 빈 제공
* ApplicationContext
  * ClassPathXmlApplicationContext : Class Path에 정의되어 있는 파일을 읽어 빈을 생성
  * AnnotationConfigApplicationContext : Annotation 기반 빈 생성
* 빈 설정
  * 빈 명세서 
  * 빈에 대한 정의를 담고 있다.
    * 이름
    * 클래스
    * 스코프
    * 생성자 (Constructor)
    * 프로퍼트 (Setter)
* 컴포넌트 스캔
  * 설정 방법
    * XML - context:component-scan
    * java - @ComponentScan
  * 특정 패키지 이하의 모든 클래스 중 @Component 애노테이션을 사용한 클래스를 빈으로 등록

## @Autowire
필요한 의존 객체의 **타입**에 해당하는 빈을 찾아 주입한다. 

* @Autowired 
  * required : 기본값이 true 이기 때문에 어플리케이션 구동 시 주입할 빈을 찾지 못하면 어플리케이션이 실행되지 않는다.

> 현재는 @Autowired를 통한 빈 주입방식 보다는 생성자를 사용한 빈 주입 방식을 사용해야한다. 

* 사용 할 수 있는 위치
  * 생성자 (스프링 4.3 부터는 생략 가능)
  * 세터, 필드
* 경우의 수
  * 해당 타입의 빈이 없는 경우 - 실패
  * 해당 타입의 빈이 한 개인 경우
  * 해당 타입의 빈이 여러 개인 경우
    * 빈 이름으로 시도
      * 같은 이름의 빈을 찾으면 해당 빈 사용
        * 같은 이름을 못 찾으면 실패
    * @Primary
    * 해당 타입의 빈을 모두 주입
    * @Qualifier (빈 이름으로 주입)
  * 동작 원리
    * BeanPostProcessor
      * 빈 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스
      * postProcessAfterInitialization, postProcessBeforeInitialization 인터페이스를 제공한다.
    * AutowiredAnnotationBeanPostProcessor
      * BeanPostProcessor를 구현한 클래스 이며 스프링이 제공하는 @Autowired와 @Value 애노테이션 그리고 JSR-330의 @Inject 애노테이션을 지원하는 애노테이션 처리기.
      * `that autowires annotated fields, setter methods, and arbitrary config methods`

## @Component와 컴포넌트 스캔
* 컴포넌트 스캔의 주요 기능
  * 스캔 위치 설정
  * 필터 : 어떤 애노테이션들을 스캔할지 또는 하지 않을지
* @Component 종류
  * @Repository
  * @Service
  * @Controller
  * @Configuration
> tmi : Repository나 Service같은 애노테이션 이름은 에릭 에반스의 DDD에서 유래되었다고 적혀있다. 
* 동작원리
  * @ComponentScan은 스캔할 패키지와 애노테이션에 대한 정보
  * 실제 스캐닝은 ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에 의해 처리 됨.

## 빈 스코프
* Scope
  * Singleton - 기본
  * ProtoType - 모든 요청에서 새로운 Bean을 생성한다.
    * Request
    * Sesseion
    * WebSocket
    * ...
  * [https://docs.spring.io/spring/docs/3.0.0.M3/reference/html/ch04s04.html](https://docs.spring.io/spring/docs/3.0.0.M3/reference/html/ch04s04.html)
* 프로토 타입 빈이 싱글톤 빈을 참조 하면 ?
  * 정상 실행
* 싱글톤 빈이 프로토 타입 빈을 참조하면 ?
  * 프로토 타입 빈이 업데이티 되지 않는다.
  * 업데이트 하려면
    * scoped-proxy
    * Object-Provider
    * Provider (표준)
  * [https://www.baeldung.com/spring-inject-prototype-bean-into-singleton](https://www.baeldung.com/spring-inject-prototype-bean-into-singleton)
* 싱글톤 객체 사용시 주의할 점
  * 상태값 공유
  * ApplicationContext 초기 구동시 인스턴스 생성

## Environment 1부 - Profile
프로파일과 프로퍼티를 다루는 인터페이스

* `ApplicationContext extends EnvironmentCapable`
  ```java
    public interface EnvironmentCapable {
        Environment getEnvironment();
    }
  ```
* 프로파일
  * 빈들의 그룹
  * Environment의 역할은 활성화 할 프로파일 확인 및 설정
* 프로파일 유즈케이스
  * 테스트 환경에서는 A라는 빈 사용, 배포 환경에서는 B라는 빈을 사용하고 싶다.
  * 서비스 환경에만 등록 하기 위한 빈 설정
* 프로파일 정의하기
  * Class
    ```java
    @Configuration 
    @Profile(“test”)

    @Component 
    @Profile(“test”)
    ```
  * Method
    ```java
    @Bean 
    @Profile(“test”)
    ```
* 프로파일 설정하기
  * -Dspring.profiles.avtive=”test,A,B,...”
  * @ActiveProfiles("test")
* 프로파일 표현식
  * `!, &, |` 와 같은 표현식으로 여러가지 경우를 계산할 수 있다. 


