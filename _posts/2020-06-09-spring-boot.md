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
* 내장 서블릿 컨테이너
  * 스프링 부트는 서버가 아니다
  * 포트 설정 가능
  * 톰캣에 컨텍스트 추가
  * 서블릿 만들기
  * 톰캣에 서블릿 추가
  * 컨텍스트에 서블릿 매핑
  * 톰캣 실행 및 대기
  * 이 모든 과정을 보다 상세히, 유연하게 설정하고 실행해주는게 바로 스프링 부트의 자동설정
    * Step 1 : ServletWebServerFactoryAutoConfiguration (서블릿 웹 서버 생성)
      * TomcatServletWebServerFactoryCustomizer (서버 커스터마이징)
    * Step 2 : DispatcherServletAutoConfiguration
      * 서블릿 만들고 등록
* 내장 웹 서버 응용 1부 : 컨테이너와 서버 포트
  * 다른 서블릿 컨테이너로 변경
    * boot-starter에 포함된 tomcat 의존성 제외
    * undertow, jetty ... 의존성 추가
  * 웹 서버 사용하지 않기
    * properties에 application-type 추가
  * 포트 변경
* 내장 웹 서버 응용 2부 : https 와 http2
  * HTTPS 설정하기
    * 키스토어 만들기
      * https://gist.github.com/keesun/f93f0b83d7232137283450e08a53c4fd
    * HTTP는 사용못함 ?
      * 기본적으로 http 커넥터는 1개라 https설정을 하게 되면 http를 사용하지 못한다.
      * Connector를 따로 등록 해 주어야 한다.
  * HTTP2
    * SSL은 기본적으로 적용되어 있어야 함
    * undertow는 별다른 설정 없이 properties로 설정 가능
    * tomcat은 8.5 버전 이하에선 설정할 것들이 많아 권장하지 않음
    * tomcat 9 & java 9 이상부터 사용 권장
  * 톰캣 HTTP2
    * JDK9와 Tomcat 9+ cncjs
    * 링크 참조
      * [링크](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-embedded-web-servers)
  * 독립적으로 실행 가능한 JAR
    * mvn package -> 독립적으로 실행 가능한 JAR 파일 하나가 생성됨
    * spring-maven-plugin이 해주는 일 (패키징)
    * 과거 "uber" jar 사용
      * 모든 클래스 (의존성 및 애플리케이션)을 하나로 압축하는 방법
      * 뭐가 어디에서 온건지 알 수 없음
        * 무슨 라이브러리를 사용한건지
      * 내용은 다르지만 이름이 같은 파일이 또 어떻게 ?
    * 스프링 부트의 전략 
      * 내장 JAR : 기본적으로 자바에는 내장 JAR를 로딩하는 표준적인 방법이 없음
      * 애플리케이션 클래스와 라이브러리 위치 구분
      * org.springframework.boot.loader.jar.JarFile을 사용해서 내장 JAR 를 읽는다.
      * org.springframework.boot.loader.Launcher를 사용해서 실행한다.

## 스프링 부트 활용
  * 스프링 부트 활용 소개
    * 스프링 부트 핵심 기능
      * SpringApplication
      * 외부 설정
      * 프로파일
      * 로깅
      * 테스트
      * Spring-Dev-Tools
    * 각종 기술 연동
      * 스프링 웹 MVC
      * 스프링 데이터
      * 스프링 시큐리티
      * REST API 클라이언트
      * ...
  * SpringApplication
    * 기본 로그 레벨 : INFO
    * FailureAnalyzer
    * 배너
      * banner.txt | gif | jpg
      * classpath 또는 spring.banner.location
      * ${spring-booot.version} 등의 변수를 사용할 수 있음
      * Banner 클래스를 구현 후 SpringApplication.setBanner() 등으로 설정 가능
    * SpringApplicationBuilder로 빌터 패턴 사용 가능.
      ```java
        //SpringApplication.run(SpringBootGetStartApplication.class, args);
        new SpringApplicationBuilder()
                .sources(SpringBootGetStartApplication.class)
                .run(args);
      ```
    * ApplicationEvent 등록
      * ApplicationContext를 만들기 전에 사용하는 리스너는 Bean을 등록할 수 없다.
        * SpringApplication.addListeners()
    * WebApplication Type 설정
    * 애플리케이션 아규먼트 사용하기
      * -D는 JVM 옵션
      * --Application Arguments
      * ApplicationArguments를 빈으로 등록해주니깐 가져다 사용하면 된다.
    * 애플리케이션 실행한 뒤 뭔가 다른 작업을 하고 싶을 경우
      * ApplicationRunner(추천) 또는 CommandLineRunner
      * 순서 지정 기능 @Order
    