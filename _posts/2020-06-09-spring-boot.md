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
    * autoconfiguration 으로 등록된 bean 은 component scan으로 등록하는 bean을 덮어 쓴다.
    * ConditionalOnMissingBean - 빈이 등록 되어 있지 않을때 빈 등록함.
    * component scan -> autoConfiguration 순으로
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
  * @SpringBootApplication 은 아래 3가지 애노테이션을 포함 하고 있다.
    * @SpringBootConfiguration
    * @EnableAutoConfiguration
    * @ComponentScan
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
        ```java
          /**
          * https와 http를 사용하기 위해 connector 등록.
          *
          */
          @Bean
          public ServletWebServerFactory serverFactory() {
              TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
              tomcat.addAdditionalTomcatConnectors(createStandardConnector());
              return tomcat;
          }

          private Connector createStandardConnector() {
              Connector connector = new Connector("org.apache/.coyote.http11.Http11NioProtocol");
              connector.setPort(8080);
              return createStandardConnector();
          }
        ```
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
      * `banner.txt | gif | jpg`
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
          ```java
            // applicationStartingEvent 는 application context 생성 전에 발생하기 때문에 이렇게 등록 해줘야 한다,
            SpringApplication app = new SpringApplication(SpringBootGetStartApplication.class);
            app.addListeners(new SampleListener());

            @Component
            public class SampleListener implements ApplicationListener<ApplicationStartedEvent> {
                @Override
                public void onApplicationEvent(ApplicationStartedEvent ApplicationStartedEvent) {
                    System.out.println("Application is Startied");
                }
            }

            @Component
            public class SampleListener implements ApplicationListener<ApplicationStartingEvent> 
                @Override
                public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
                    System.out.println("Application is Starting");
                }
            }
          ```
    * WebApplication Type 설정
      * WebApplicationType
        * NONE : 서블릿과 Webflux가 없을때
        * SERVLET : Servlet
        * REACTIVE : WebFlux
      ```java
        SpringApplication app = new SpringApplication(SpringBootGetStartApplication.class);
        app.setWebApplicationType(WebApplicationType.NONE);
        app.run(args);
      ```
    * 애플리케이션 아규먼트 사용하기
      * -D는 JVM 옵션
      * --Application Arguments
      * ApplicationArguments를 빈으로 등록해주니깐 가져다 사용하면 된다.
    * 애플리케이션 실행한 뒤 뭔가 다른 작업을 하고 싶을 경우
      * ApplicationRunner(추천) 또는 CommandLineRunner
      * 순서 지정 기능 @Order
  * 외부 설정
    * properties
    * yaml
    * 환경 변수
    * 커맨드 라인 아규먼트
    * 프로퍼티 우선순위
      1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
      2. 테스트에 있는 @TestPropertySource
        * @TestPropertySource(locations = "classpath:/test.properties")
      3. @SpringBootTest 애노테이션의 properties 애트리뷰트
        * @SpringBootTest(properties = "dongchul.name=dong2")
      4. 커맨드 라인 아규먼트
      5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
      6. ServletConfig 파라미터
      7. ServletContext 파라미터
      8. java:comp/env JNDI 애트리뷰트
      9. System.getProperties() 자바 시스템 프로퍼티
      10. OS 환경 변수
      11. RandomValuePropertySource
      12. JAR 밖에 있는 특정 프로파일용 application properties
      13. JAR 안에 있는 특정 프로파일용 application properties
      14. JAR 밖에 있는 application properties
      15. JAR 안에 있는 application properties
      16. @PropertySource
      17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)
    * application.properties 우선순위
      1. file:./config/
      2. file:./
      3. classpath:/config/
      4. classpath:/
    * 랜덤 값 설정하기
      * ${random.*}
    * 플레이스 홀더
      * name = dongchul
      * fullName = ${name} lee
    * 타입 - 세이프 프로퍼티 @ConfigurationProperties
      ```java
        @Component
        @ConfigurationProperties("dongchul")
        @Validated // 프로퍼티 값을 검증 하기 위해 사용
        public class DonghculProperties {

            @NotEmpty
            private String name;

            @Size(min = 0,max = 100)
            private int age;

            private String fullName;
            ...
        }
      ```
      * 여러 프로퍼티를 묶어서 읽어올 수 있음.
      * 빈으로 등록해서 다른 빈에 주입 할 수 있음
        * @EnableConfigurtionProperties
        * @Component
        * @Bean
      * 융통성 있는 바인딩
        * context-path (kebab)
        * context_path (under score)
        * contextPath (camel)
        * CONTEXTPATH
      * 프로퍼티 타입 컨버전
        * @DurationUnit
      * 프로퍼티 값 검증
        * @Validated
        * JSR-303(@NotNull, ...)
      * 메타 정보 생성
      * @Value
        * SpEL을 사용할수 있지만 위 기능들을 전부 사용하지 못한다.
  * 프로파일
    * @Profile 애노테이션은 어디에서 사용하는가 ?
      * @Configuration
      * @Component
    * 어떤 프로파일을 활성화 할 것인가 ?
      * spring.profiles.active
    * 어떤 프로파일을 추가할 것인가 ?
      * spring.profiles.include 
    * 프로파일용 프로퍼티
      * application-{profile}.properties / yml
  * 로깅
    * 로깅 퍼사드 vs 로거
      * Commons Logging, SLF4j
      * JUL, Log4J2, LogBack
    * 스프링 5에 로거 관련 변경 사항
      * [https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/overview.html#overview-logging](https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/overview.html#overview-logging)
      * Spring-JCL
        * Commons Logging -> SLF4j or Log4j2
    * 스프링 부트 로깅
      * 기본 포멧
      * --debug (일부 핵심 라이브러리만 디버깅 모드 설정)
      * --trace (전부 다 디버깅 모드 설정)
      * 컬러 출력 : spring.output.ansi.enabled
      * 파일 출력 : logging.file 또는 logging.path
      * 로그 레벨 조정 : logging.level.패키지 = 로그레벨
    * 커스텀 로그 설정 파일 사용하기 
      * [https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html)
      * Logback : logback-spring.xml (추천, logback.xml보다 더 많은 기능을 사용할 수 있다.)
        ```xml
          <?xml version="1.0" encoding="UTF-8"?>
          <configuration>
              <include resource="org/springframework/boot/logging/logback/base.xml"/>
              <logger name="me.study" level="DEBUG"/>
          </configuration>
        ```
      * Log4J2 : log4j2-spring.xml
      * JUL (비추) : logging.properties
      * Logback extention
        * 프로파일 `<springProfile name="프로파일">`
        * Environment 프로퍼티 `<springProperty>`
    * 로거를 Log4j2로 변경하기
      * [https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging)
  * 테스트
    * 시작은 spring-boot-starter-test를 추가하는것 부터
      * test scope로 추가
    * @SpringBootTest - 통합 테스트용
      * @RunWith(SpringRunner.class)랑 같이 써야함 - (Junit5 부터는 생략 가능하다.)
      * 빈 설정 파일은 설정을 안해주나 ? 알아서 찾는다. (@SpringBootApplication)
      * WebEnvironment
        * MOCK : mock servlet environment : 내장 톰캣 구동 안함.
        * RANDOM_PORT, DEFINED_PORT : 내장 톰캣 사용 함
        * NONE : 서블릿 환경 제공 안함
    * @MockBean
      * ApplicaionContext에 들어있는 빈을 Mock으로 만든 객체로 교체 함 
      * 모든 @Test 마다 자동으로 리셋
    * 슬라이스 테스트
      * 레이어 별로 잘라서 테스트하고 싶을떄
      * @JsonTest
      * @WebMvcTest
      * @WebFluxTest
      * @DataJpaTest
      * ...
    * 테스트 유틸
      * OutputCapture (log 및 stdout 관련 테스트)
      * TestPropertyValues
      * TestRestTemplate
      * ConfigFileApplicationContextInitailizer
    
    ```java
      /**
      *
      * @SpringBootTest
      * 통합 테스트용
      *
      * @WebMvcTest(SampleContorller.class)
      * 슬라이싱 테스트용
      * 선언한 Controller 관련된 Bean 만 생성 하기 때문에
      * Service ,Repository 같은 Bean 들은 모킹 해야 한다.
      */
      @RunWith(SpringRunner.class)
      //@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK) // 기본값
      @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) // 실제로 서버가 구동됨. rest template 로 확인 해야함.
      @AutoConfigureMockMvc // mock mvc를 사용 하기 위함
      public class SampleControllerTest {
          // async client test for webflux
          // webflux의존성을 가지고 있어야 사용 가능.
          @Autowired
          WebTestClient webTestClient;

          @Autowired
          MockMvc mockMvc;

          @Autowired
          TestRestTemplate testRestTemplate;

          @MockBean //컨트롤러 테스트를 위해 모킹.
          SampleService mockSampleService;

          @Test
          public void hello() throws Exception {
              // sample service 를 mocking
              when(mockSampleService.getName()).thenReturn("dongchul");

              mockMvc.perform(get("/hello"))
                      .andExpect(status().isOk())
                      .andExpect(content().string("hello dongchul"))
                      .andDo(print());

              String result = testRestTemplate.getForObject("/hello", String.class);
              assertEquals(result, "hello dongchul");

              webTestClient.get().uri("/hello")
                      .exchange()
                      .expectStatus()
                      .isOk()
                      .expectBody(String.class)
                      .isEqualTo("hello dongchul");
          }
      }
    ```
  * Spring Boot Devtools
    * 캐시 설정을 개발 환경에 맞게 변경.
    * 클래스패스에 있는 파일이 변경 될 때 마다 자동으로 재시작.
        * 직접 껐다 켜는것(cold starts) 보다 빠르다. 
        * 릴로딩 보다는 느리다. (JRebel같은게 아님)
        * 리스타트 하고 싶지 않은 리소스는 ? spring.devtools.restart.exclude
        * 리스타트 기능 끄려면 ? spring.devtools.restart.enabled = false
    * 라이브 릴로드 ? 리스타트 했을 때 브라우저 자동 리프레시 하는 기능
        * 브라우저 플러그인 설치해야 함
        * 라이브 릴로드 서버 끄려면 ? spring.devtools.liveload.enabled = false
    * 글로벌 설정
        * ~/.spring-boot-devtools.properties
    * 리모트 애플리케이션