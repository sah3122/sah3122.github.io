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
  * 스프링 웹 MVC
    * [레퍼런스](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#spring-web)
    * 스프링 부트 MVC
      * 자동 설정으로 제공하는 여러 기본 기능(앞으로 살펴볼 예정)
    * 스프링 MVC 확장
      * @Configuration + WebMvcConfigurer
    * 스프링 MVC 재정의
      * @Configuration + @EnableWebMvc
    * HttpMessageConverters
      * HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경 할 때 사용
        * {“username”:”dongchul”, “password”:”123”} <-> User
          ```java
            @PostMapping("/user")
            public User create(@RequestBody User user) {
                return user;
            }
          ```
      * 스프링 부트
        * 뷰 리졸버 설정 제공
        * HttpMessageConvertersAutoConfiguration
    * 정적 리소스 매핑 "/**"
      * 기본 리소스 위치
        * classpath:/static
        * classpath:/public
        * classpath:/resources/
        * classpath:/META-INF/resources
        * 예) "/hello.html" => static/hello.html
        * spring.mvc.static-path-pattern: 매핑 설정 변경 가능
        * spring.mvc-static-locations: 리소스 찾을 위치 변경 가능
      * Last-Modified 헤더를 보고 304 응답을 보냄
      * ResourceHttpRequestHandler가 처리
        * WebMvcConfigurer의 addResourceHandlers로 커스터마이징 가능
    * 웹 JAR 맵핑 "/webjars/*"
      * 버전 생략 하고 사용하려면 webjars-locator-core 의존성 추가
    * 웰컴 페이지
      * index.html 찾아보고 있으면 제공
      * index.템플릿 찾아보고 있으면 제공
      * 둘다 없으면 에러페이지 노출
    * 파비콘
      * favicon.ico 파일을 static 폴더 하위에 제공
      * 파비콘 만들기 [파비콘](https://favicon.io)
    * 템플릿 엔진
      * 스프링 부트가 자동설정을 지원하는 템플릿 엔진
        * FreeMarker
        * Groovy
        * Thymeleaf
        * Mustache
      * JSP를 권장하지 않는 이유
        * JAR 패키징 할 때는 동작하지 않고 WAR 패키징 해야함.
        * Undertow는 JSP를 지원하지 않음.
      * Thymeleaf 사용하기
        * https://www.thymeleaf.org
        * https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html
        * 의존성 추가 : spring-boot-starter-thymeleaf
        * 템플릿 파일 위치 : /src/main/resources/template
        * 예제 : https://github.com/thymeleaf/thymeleafexamples-stsm/blob/3.0-master/src/main/webapp/WEB-INF/templates/seedstartermng.html
    * HTML 템블릿 뷰 테스트를 보다 전문적으로 하자.
      * http://htmlunit.sourceforge.net/
      * http://htmlunit.sourceforge.net/gettingStarted.html
      * 의존성 추가
      * @Autowired WebClient  
    * ExceptionHandler
      * 스프링 @MVC 예외 처리 방법
        * @ControllerAdvice
        * @ExceptionHanlder
          ```java
            @ExceptionHandler(SampleException.class)
            public @ResponseBody AppError sampleError(SampleException e) {
                AppError appError = new AppError();
                appError.setMessage("error.app.key");
                appError.setReason("IDK");
                return appError;

            }
          ```
      * 스프링 부트가 제공하는 기본 예외 처리기
        * BasicErrorController
          * HTML 과 JSON 응답 지원
        * 커스터마이징 방법
          * ErrorController 구현
      * 커스텀 에러 페이지
        * 상태 코드 값에 따라 에러 페이지 보여주기
        * src/main/resources/static/error/
        * 404.html
        * 5xx.html
        * ErrorViewResolver 구현  
    * Spring HATEOAS
      `Hypermedia As The Engine Of Application State`
      * 서버 : 현재 리소스와 연관된 링크 정보를 클라이언트에게 제공한다.
      * 클라이언트 : 연관된 링크 정보를 바탕으로 리소스에 접근한다.
      * 연관된 링크 정보
        * Relation
        * Hypertext Reference
      * spring-boot-starter-hateoas 의존성 추가
      * https://spring.io/understanding/HATEOAS
      * https://spring.io/guides/gs/rest-hateoas/
      * https://docs.spring.io/spring-hateoas/docs/current/reference/html/
      * ObjectMapper 제공
        * 커스터마이징
          * property에 spring.jackson.* 값 변환하는 방식 추천
        * Jackson2ObjectMapperBuilder
      * LinkDiscovers 제공
        * 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을 때 사용할 수 있는 XPath 확장 클래스 
      ```java
        public class Hello extends RepresentationModel {
            private String prefix;

            private String name;
            ...
        }

        public Hello hello() {
            Hello hello = new Hello();
            hello.setName("dongchul");
            hello.setPrefix("Hey, ");

            hello.add(linkTo(SampleController.class).withSelfRel());
            return hello;
        }
      ```
    * CORS
      * SOP와 CORS
        * Single-Origin Policy
        * Cross-Origin Resource Sharing
        * Origin ?
          * URI 스키마 (http, https)
          * hostname (localhost...)
          * 포트 (8080, 18080)
      * 스프링 MVC @CrossOrigin
        * https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors
        * @Controller나 @RequestMapping에 추가 하거나
          ```java
            @CrossOrigin(origins = "http://localhost:18080")
            @GetMapping("/hello")
            public Hello hello() {
                Hello hello = new Hello();
            }
          ```
        * WebMvcConfigurer 사용해서 글로벌 설정  
          ```java
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedOrigins("htt://localhost:18080");
            }
          ```
  * 스프링 데이터
    * 인메모리 데이터베이스
      * 지원하는 인-메모리 데이터베이스
        * H2 (추천, 콘솔때문에)
        * HSQL
        * Derby
      * Spring-JDBC가 클래스패스에 있으면 자동 설정이 필요한 빈을 설정 해준다.
        * DataSource
        * JdbcTemplate
      * 인-메모리 데이터베이스 기본 연결 정보 확인 하는 방법
        * URL : "testdb"
        * username : "sa"
        * password : ""
      * H2 콘솔 사용하는 방법
        * spring-boot-devtools를 추가 하거나
        * spring.h2.console.enabled=true 추가
        * /h2-console로 접속 (path변경 가능)
    * MySQL
      * 지원하는 DBCP
        1. HikariCP (기본)
          * https://github.com/brettwooldridge/HikariCP#frequently-used
        2. Tomcat CP
        3. Commonc DBCP2
      * DBCP 설정
        * spring.datasource.hikari.*
        * spring.datasource.tomcat.*
        * spring.datasource.dbcp2.*
      * MySQL 라이센스 (GPL) 주의
        * MySQL 대신 MariaDB 사용 검토
        * 소스 코드 공개 의무 여부 확인
      * MySQL 접속시 에러
        * MySQL 5.* 최신 버전 사용할 때 문제
          * `Sat Jul 21 11:17:59 PDT 2018 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.`
        * 해결
          * jdbc:mysql:/localhost:3306/springboot?useSSL=false
        * MySQL 8.* 최신 버전 사용할 때 문제
          * com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Public Key Retrieval is not allowed
        * 해결
          * jdbc:mysql:/localhost:3306/springboot?useSSL=false&allowPublicKeyRetrieval=true
    * PostgreSQL
      * Mysql 과는 달리 라이센스 문제가 없다. 가장 추천하는 DB
    * Spring-Data-JPA 
      * ORM(Object-Relational Mapping)과 JPA (Java Persistence API)
        * 객체와 릴레이션을 맵핑할 때 발생하는 개념적 불일치를 해결하는 프레임워크
        * http://hibernate.org/what-is-an-orm
        * JPA: ORM을 위한 자바(EE) 표준
      * 스프링 데이터 JPA
        * Repository 빈 자동 생성
        * 쿼리 메소드 자동 구현
        * @EnableJpaRepositories (스프링 부트가 자동으로 설정 해줌.)
      * 스프링 데이터 JPA 사용하기
        * @Entity 클래스 만들기
        * Repository 만들기
      * 스프링 데이터 리파지토리 테스트 만들기
        * H2 DB를 테스트 의존성에 추가하기
        * @DataJpaTest (슬라이스 테스트) 작성      
          * 테스트를 실행할 때는 Inmemory DB를 사용하는것을 추천.  
          ```java
            @RunWith(SpringRunner.class)
            @DataJpaTest // 슬라이싱 테스트 DataSource, JdbcTemplate, Repository 등을 주입 받는다. 인메모리 데이터 베이스가 반드시 필요함
            public class AccountRepositoryTest {
                @Autowired
                DataSource dataSource;

                @Autowired
                JdbcTemplate jdbcTemplate;

                @Autowired
                AccountRepository accountRepository;

                ...
            }
          ```
      * JPA를 사용한 데이터베이스 초기화
        * spring.jpa.hibernate.dll.auto 
          * create
          * create-drop
          * update
          * validate
        * spring.jpa.generate-dll=true로 설정 해줘야 한다.
      * SQL 스크립트를 사용한 데이터베이스 초기화
        * schema.sql 또는 schema-${platform}.sql
        * data.sql 또는 data-${platform}.sql
        * ${platform} 값은 spring.datasource.platform으로 설정 가능. 
      * 데이터베이스 마이그레이션 툴
        * Flyway 와 Liquibase가 대표적, Flyway 사용
        * https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup
        * 의존성 추가
          * org.flywaydb:flyway-core
        * 마이그레이션 디렉토리
          * /resource/db/migration 또는 db/migration/{vendor}
          * spring.flyway.location으로 변경 가능
        * 마이그레이션 파일 이름
          * 한번 적용이 된 파일은 절대 수정하면 안되고 새로운 파일을 만들어서 버전을 올려야 한다.
          * V숫자__이름.sql
          * V는 꼭 대문자
          * 숫자는 순차적으로 (타임스템프 권장)
          * 숫자와 이름 사이에 언더바 두개
          * 이름은 가능한 서술적으로
      * Redis
        * 캐시, 메시지 브로커, 키/벨류 스토어 등으로 사용 가능
        * 의존성 추가
          * spring-boot-starter-data-redis
        * 스프링 데이터 Redis
          * https://projects.spring.io/spring-data-redis/
          * StringRedisTemplate 또는 RedisTemplate
          * extends CrudRepository
        * Redis 주요 커맨드
          * https://redis.io/commands
          * keys *
          * get {key}
          * hgetall {key}
          * hget {key} {column}
        * 커스터마이징
          * spring.redis.*
      * MongoDB
        * 스프링 데이터 몽고 DB
          * Mongotemplate
          * MongoRepository
          * 내장형 MongoDB (테스트용)
            * de.flapdoodle.embed:de.flapdoodle.embed.mongo
          * @DataMongoTest
      * Neo4J
        * Neo4j는 노드간의 연관 관계를 영속화 하는데 유리한 그래프 데이터베이스
        * 스프링 데이터 Neo4J
          * Neo4jTemplate (Deprecated)
          * SessionFactory
          * Neo4jRepository
    * 스프링 시큐리티
      * 웹 시큐리티
      * 메소드 시큐리티
      * 다양한 인증 방법 지원
        * LDAP, 폼 인증, Basic 인증, OAuth, ...
      * 스프링 부트 시큐리티 자동 설정
        * SecurityAutoConfiguration 
        * UserDetailServiceAutoConfiguration
        * spring-boot-starter-secutiry
          * 스프링 시큐리티 5.* 의존성 추가
        * 모든 요청에 인증이 필요함
        * 기본 사용자 생성
          * Username : user
          * Password : 애플리케이션을 실행 할 때 마다 랜덤 값 생성 (콘솔 출력)
          * spring.security.user.name
          * spring.security.user.password
        * 인증 관련 각종 이벤트 발생
          * DefaultAuthenticationEventPublisher 빈 등록
          * 다양한 인증 에러 핸들러 등록 가능
      * UserDetailsService 구현
        * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication-userdetailsservice
        * UserDetails 를 구현한 기본 구현체(User)를 Spring Security에서 제공해준다.
        ```java
          @Service
          public class SecutiryAccountService implements UserDetailsService {

              @Autowired
              private PasswordEncoder passwordEncoder;

              @Autowired
              private SecurityAccountRepository securityAccountRepository;

              public SecurityAccount createAccount(String username, String password) {
                  SecurityAccount securityAccount = new SecurityAccount();
                  securityAccount.setUsername(username);
                  securityAccount.setPassword(passwordEncoder.encode(password));
                  return securityAccountRepository.save(securityAccount);
              }

              @Override
              public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                  Optional<SecurityAccount> byUserName = securityAccountRepository.findByUserName(username);
                  SecurityAccount securityAccount = byUserName.orElseThrow(() -> new UsernameNotFoundException(username));

                  return new User(securityAccount.getUsername(), securityAccount.getPassword(), authorites()); // UserDetails 를 구현한 기본 구현체를 Spring Security에서 제공해준다.
              }

              private Collection<? extends GrantedAuthority> authorites() {
                  return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
              }
          }
        ```
      * PasswordEncoder 설정 및 사용
        * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#core-services-password-encoding
        ```java
          // Noop : 사용해서는 안되는 방법. 아무 인코딩 / 디코딩을 하지 않는 방법
          @Bean
          public PasswordEncoder passwordEncoder() {
              //return NoOpPasswordEncoder.getInstance();
              return PasswordEncoderFactories.createDelegatingPasswordEncoder();
          }
        ```
  * RestTemplate 과 WebClient
    * RestTemplate
      * Blocking I/O 기반의 Synchronous API
      * RestTemplateAutoConfiguration
      * 프로젝트에 spring-web 모듈이 있다면 RestTemplateBuilder를 빈으로 등록해준다.
      * https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#rest-client-access
      * 기본적으로 java.net.HttpURLConnection 사용
      * 커스터마이징
        * 로컬 커스터마이징
        * 글로벌 커스터마이징
          * RestTemplateCustomizer
          * 빈 재정의 
    * WebClient 
      * Non-Blocking I/O 기반의 Asynchronous API
      * WebClientAutoConfiguration
      * 프로젝트에 spring-webflux 모듈이 있다면 WebClient.Builder를 빈으로 등록해준다.
      * https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client
      * 기본적으로 Reactor Netty의 HTTP 클라이언트 사용
      * 커스터마이징
        * 로컬 커스터마이징
        * 글로벌 커스터마이징
          * WebClientCustomizer
          * 빈 재정의
      ```java
        @Component
        public class RestRunner implements ApplicationRunner {

            @Autowired
            RestTemplateBuilder restTemplateBuilder;

            @Autowired
            WebClient.Builder builder;

            @Override
            public void run(ApplicationArguments args) throws Exception {

                RestTemplate restTemplate = restTemplateBuilder.build();
                WebClient webClient = builder.build();

                StopWatch stopWatch = new StopWatch();
                stopWatch.start();

                String helloResult = restTemplate.getForObject("http://localhost:8080/hello", String.class);
                System.out.println(helloResult);

                String worldResult = restTemplate.getForObject("http://localhost:8080/world", String.class);
                System.out.println(worldResult);

                Mono<String> helloMono = webClient.get().uri("http://localhost:8080/hello")
                        .retrieve()
                        .bodyToMono(String.class);
                //subscribe 시 실제로 요청을 보내게 되며 non blocking 으로 동작 한다.
                helloMono.subscribe(s -> {
                    System.out.println(s);
                    if(stopWatch.isRunning())
                        stopWatch.stop();

                    System.out.println(stopWatch.prettyPrint());
                    stopWatch.start();
                });

                Mono<String> worldMono = webClient.get().uri("http://localhost:8080/world")
                        .retrieve()
                        .bodyToMono(String.class);
                worldMono.subscribe(s -> {
                    System.out.println(s);
                    if(stopWatch.isRunning())
                        stopWatch.stop();

                    System.out.println(stopWatch.prettyPrint());
                    stopWatch.start();
                });

                stopWatch.stop();
                System.out.println(stopWatch.prettyPrint());
            }
        }
      ```
  * 스프링 부트 운영
    * 스프링 부트 Actuator 
      * 의존성
        * spring-boot-starter-actuator
      * 애플리케이션의 각종 정보를 확인 할 수 있는 Endpoints
        * 다양한 Endpoints 제공
        * JMX 또는 HTTP를 통해 접근 가능 함
        * shutdown을 제외한 모든 Endpoint는 기본적으로 활성화 상태
        * 활성화 옵션 조정
          * management.endpoints.enabled-by-default=false
          * management.endpoint.info.enabled=true
      * JConsole 사용하기
        * https://docs.oracle.com/javase/tutorial/jmx/mbeans/
        * https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html
      * VisualVM 사용하기
        * https://visualvm.github.io/download.html
        * java 10이상은 포함되어 있지 않다.
        * jconsole보다 확인하기 편하다
      * HTTP 사용하기
        * /acturator
        * health와 info를 제외한 대부분의 Endpoint가 기본적으로 비공개 상태
        * 공개 옵션 조정
          * management.endpoints.web.expose.include=*
          * management.endpoints.web.expose.exclude=env,beans
      * 스프링 부트 어드민
        * https://github.com/codecentric/spring-boot-admin 스프링 부트 Actuator UI 제공 
        * 어드민 서버 설정
          * @EnableAdminServer
          * spring-boot-admin-starter-server 의존성 필요
        * 클라이언트 설정
          * spring-boot-admin-starter-client 의존성 필요
          * spring.boot.admin.client.url=http://localhost:8080(어드민 서버 주소)
          * management.endpoints.web.exposure.include=*
        * 운영중에도 로그 레벨 변경 가능
        * 스프링 시큐리티를 반드시 설정 해야 한다.