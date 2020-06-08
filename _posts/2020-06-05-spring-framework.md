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
강의 내용을 다시한번 복습하기 위하여 정리합니다.


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

## Environment 2부 - Property
* 프로퍼티란 ?
  * 다양한 방법으로 정의할 수 있는 설정값
  * [Environment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Environment.html)의 역할은 프로퍼티 소스 설정 및 값 가져오기 
* 프로퍼티 우선순위
  * StandardServletEnvironment의 우선순위
    * ServletConfig 매개변수
    * ServletContext 매개변수
    * JNDI (java:comp/env/)
    * JVM 시스템 프로퍼티 (-Dkey=”value”)
    * JVM 시스템 환경 변수 (운영 체제 환경 변수)
* @PropertySource
  * Environment를 통해 프로퍼티 추가하는 방법
* 스프링 부트의 외부 설정 참고
  * 기본 프로퍼티 소스 지원 (application.properties)
  * 프로파일 까지 고려한 계층형 프로퍼티 우선순위 제공

## MessageSource
다양한 언어 지원기능을 제공하는 인터페이스

* `ApplicationContext extends MessageSource`
    ```java
      public interface MessageSource {
        @Nullable
	      String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
        ...
      }
    ```
* 스프링 부트를 사용한다면 별다른 설정 필요없이 message.properties를 사용할 수 있음.
  * messages.properties
  * messages_ko_kr.properties
* Reloading 기능을 제공하는 MessageSource
  ```java
    @Bean
    public MessageSource messageSource() {
      var messageSource = new ReloadableResourceBundleMessageSource();
      messageSource.setBasename("classpath:/messages");
      messageSource.setDefaultEncoding("UTF-8");
      messageSource.setCacheSeconds(3);
      return messageSource;
    }
  ```  
## ApplicationEventPublisher
이벤트 프로그래밍에 필요한 인터페이스 제공 [옵저버패턴](https://johngrib.github.io/wiki/observer-pattern/) 구현체

* `ApplicationContext extends ApplicationEventPublisher`
  ```java
    @FunctionalInterface
    public interface ApplicationEventPublisher {
      default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
      }
      void publishEvent(Object event);
    }
  ```
* 이벤트 만들기
  * ApplicationEvent 상속
  * 스프링 4.2 부터는 이 클래스를 상속받지 않아도 이벤트로 사용할 수 있다.
* 이벤트 처리하는 방법
  * ApplicationListener<이벤트> 구현한 클래스 만들어서 빈으로 등록하기.
  * 스프링 4.2 부터는 @EventListener를 사용해서 빈의 메소드에 사용할 수 있다.
  * 기본적으로는 synchronized.
  * 순서를 정하고 싶다면 @Order와 함께 사용.
  * 비동기적으로 실행하고 싶다면 @Async와 함께 사용.
* 스프링이 제공하는 기본 이벤트
  * ContextRefreshedEvent: ApplicationContext를 초기화 했더나 리프래시 했을 때 발생.
  * ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작 신호를 받은 시점에 발생.
  * ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지 신호를 받은 시점에 발생.
  * ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에 발생.
  * RequestHandledEvent: HTTP 요청을 처리했을 때 발생.  

```java
  @Component
  public class AppRunner implements ApplicationRunner {

      @Autowired
      ApplicationEventPublisher eventPublisher; // 이벤트를 넘겨주기 위한 객체

      @Override
      public void run(ApplicationArguments args) throws Exception {
          eventPublisher.publishEvent(new MyEvent(this, 100));

      }
  }

  @Component
  public class MyEventHandler {
    @EventListener  
    @Async
    public void handle(MyEvent myEvent) {
        System.out.println(Thread.currentThread().toString());
        System.out.println("event" + myEvent.getData());
    }
  }
```

[사용법](https://www.baeldung.com/spring-events)

## ResourceLoader
리소스를 읽어오는 기능을 제공하는 인터페이스
* `ApplicationContext extedns ResoruceLoader`
  ```java
    public interface ResourceLoader {
        String CLASSPATH_URL_PREFIX = "classpath:";

        Resource getResource(String var1);

        @Nullable
        ClassLoader getClassLoader();
    }
  ```
* 리소스 읽어오기
  * 파일 시스템에서 읽어오기
  * 클래스패스에서 읽어오기
  * URL로 읽어오기
  * 상대/절대 경로로 읽어오기
  ```java
  @Component
  public class AppRunner implements ApplicationRunner {

      @Autowired
      ApplicationContext applicationContext;

      @Override
      public void run(ApplicationArguments args) throws Exception {
          System.out.println(applicationContext.getClass());

          Resource resource = applicationContext.getResource("classpath:test.txt");
          System.out.println(resource.getClass());
          System.out.println(resource.exists());
          System.out.println(resource.getDescription());
          System.out.println(Files.readString(Path.of(resource.getURI())));
      }
  }
  ```

## Resource 추상화
* 특징
  * java.net.URL을 추상화 한것
  * 스프링 내부에서 많이 사용하는 인터페이스
* 추상화 한 이유
  * 클래스 패스 기준으로 리소스 읽어오는 기능 부재
  * ServletContext를 기준으로 상대 경로를 읽어오는 기능 부재
  * 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수 있지만 구현이 복잡하고 편의성 메소드가 부족하다.
* [Resource](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html)
  * 상속 받은 인터페이스
  * 주요 메소드
    * getInputStream()
    * exitst()
    * isOpen()
    * getDescription(): 전체 경로 포함한 파일 이름 또는 실제 URL
* 구현체
  * UrlResource : 기본으로 지원하는 프로토콜 http, https, ftp, file, jar
  * ClassPathResource : 지원하는 접두어 classpath:
  * FileSystemResource 
  * ServletContextResource : 웹 어플리케이션 루트에서 상대 경로로 리소스를 찾는다.
  * ...
* 리소스 읽어오기
  * Resource의 타입은 locaion 문자열과 ApplicationContext의 타입에 따라 결정 된다.
    * ClassPathXmlApplicationContext -> ClassPathResource
    * FileSystemXmlApplicationContext -> FileSystemResource
    * WebApplicationContext -> ServletContextResource
  * ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+ classpath:)중 하나를 사용할 수 있다.
    * classpath:me/whiteship/config.xml -> ClassPathResource
    * file:///some/resource/path/config.xml -> FileSystemResourc

## Validation 추상화
애플리케이션에서 사용하는 객체 검증용 인터페이스

* 특징
  * 어떠한 계층과도 관계가 없다. -> 모든 계층(웹, 서비스, 데이터)에서 사용할 수 있다.
  * 구현체 중 하나로, JSR-303(Bean Validation 1.0)과 JSR-349(Bean Validation 1.1)을 지원한다.
* 인터페이스
  * boolean support(Class clazz) : 어떤 타입의 객체를 검증할 때 사용할 것인지 결정
  * void validate(Object obj, Errors e) : 실제 검증 로직을 이 안에서 구현
    * 구현 시 ValidationUtils를 사용하며 편리
* 스프링 부트 2.0.5 이상 버전을 사용할 때
  * LocalValidatorFactoryBean 빈으로 자동 등록
  * JSR-380(Bean Validation 2.0.1) 구현체로 hibernate-validator 사용.
  * https://beanvalidation.org/

```java
public class Event {

    Integer id;

    @NotEmpty
    String title;

    @Min(value = 0)
    Integer limit;

    @Email
    String email;
}

public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.equals(aClass);
    }

    @Override
    public void validate(Object o, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "notempty", "Empty title is not allow"); // errorcode는 messagesource에서 가지고 온다.
    }
}
```

> 참고 : 스프링 부트 2.3 버전 부터 spring-boot-starter-web에서 validtaion이 분리됨.
spring-boot-starter-validation을 추가해줘야 한다.

## 데이터 바인딩 추상화 : PropertyEditor
[DataBinder](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DataBinder.html)

* 기술적인 관점 
  * 프로퍼티 값을 타겟 객체에 설정하는 기능
* 사용자 관점
  * 사용자 입력값을 애플리케이션 도메인 모델에 동적으로 변환해 넣어주는 기능
  * 입력값 대부분은 "문자열"이지만 그 값을 객체가 가지고 있는, int, String, long, Boolean 심지어 사용자 도메인으로 변환하여 넣어준다.
* PropertyEditor 
  * 스프링 3.0 이전까지 DataBinder가 변환 작업시 사용하던 인터페이스
  * thread - safe하지 않다.
  * Object와 String 간의 변환만 할수 있어 사용범위가 제한적.

```java
public class EventPropertyEditor extends PropertyEditorSupport {
  @Override
  public String getAsText() {
    return ((Event)getValue()).getTitle();
  }
  @Override
  public void setAsText(String text) throws IllegalArgumentException {
    int id = Integer.parseInt(text);
    Event event = new Event();
    event.setId(id);
    setValue(event);
  }
}
```

## 데이터 바인딩 추상화 : Converter와 Formmater

* [Converter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/converter/Converter.html)
  * S 타입을 T 타입으로 변환할 수 있는 일반적인 변환기
  * 상태값이 없다 -> Thread Safe하다
  * ConverterRegistry에 등록하여 사용
  ```java
    @RestController
    public class EventController {

        @GetMapping("/event/{event}")
        public String getEvent(@PathVariable Event event) {
            return event.getId().toString();
        }
    }

    @Component
    public static class StringToEventConverter implements Converter<String, Event> {
        @Override
        public Event convert(String s) {
            return new Event(Integer.parseInt(s));
        }
    }
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new StringToEventConverter());
        }
        
    }
  ```
* [Formatter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/format/Formatter.html)
  * PropertyEditor 대체제
  * Object와 String 간의 변환을 담당한다.
  * 문자열을 Locale에 따라 다국화하는 기능도 제공한다. (optional)
  * FormatterRegistry에 등록해서 사용
  ```java
    //thread safe 하여 bean 등록 가능 및 다른 bean 주입 가능
    //data binding 관련해서 사용할 경우 event formatter를 사용하는것을 추천.
    @Component
    public class EventFomatter implements Formatter<Event> {

        @Autowired
        MessageSource messageSource;

        @Override
        public Event parse(String s, Locale locale) throws ParseException {
            return new Event(Integer.parseInt(s));
        }

        @Override
        public String print(Event event, Locale locale) {
            return event.getId().toString();
        }
    }

    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addFormatter(new EventFomatter());
        }
    }
  ```
* [ConversionService](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/ConversionService.html)
  * 실제 변환 작업은 이 인터페이스를 통해서 thread-safe 하게 사용할 수 있음.
  * 스프링 MVC, 빈 (value) 설정, SpEL에서 사용한다.
  * DefaultFormattingConversionService
    * FormatterRegistry
    * ConversionService
    * 여러 기본 컴버터와 포매터 등록 해 줌.
* 스프링 부트
  * 웹 애플리케이션인 경우에 DefaultFormattingConversionSerivce를 상속하여 만든 WebConversionService를 빈으로 등록해 준다.  
    `public class WebConversionService extends DefaultFormattingConversionService`
  * Formatter와 Converter 빈을 찾아 자동으로 등록해 준다.

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ConversionService conversionService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(conversionService); // 등록된 converter를 확인 하는 방법.
        System.out.println(conversionService.getClass().toString());
    }
}
```

## SpEL(Spring Expression Language)
* [SpEL](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions) 이란 ?
  * 객체 그래프를 조회 및 조작하는 기능 제공
  * 메소드 호출 지원, 문자열 템플릿 기능도 제공
  * SpEL은 모든 스프링 프로젝트 전반에서 사용하기 위한 EL로 만들어졌다.
  * 스프링 3.0 부터 지원
* SpEL 구성
  * ExpressionParser parser = new SpelExpressionParser()
  * StandardEvaluationContext context = new StandardEvaluationContext(bean)
  * Expression expression = parser.parseExpression("SpEL 표현식")
  * String value = expression.getValue(context, String class)

* 문법
  * #{"표현식"}
  * ${"프로퍼티"}
  * 표현식은 프로퍼티를 가질 수 있지만 반대는 안된다.
    * #{${my.data} + 1}
  * [레퍼런스](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions-language-ref)
* 사용처
  * @Value 애노테이션 [참고](https://www.baeldung.com/spring-value-annotation)
  * @ConditionalOnExpression 애노테이션
  * Spring Security
    * Method Security, @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter
    * XML 인터셉터 URL 설정
    * ...
  * 스프링 데이터
    * @Query 애노테이션
  * ...  
```java
    @Value("#{1 + 1}")
    int value;

    @Value("#{'hello ' + 'world'}")
    String greeting;

    @Value("#{1 eq 1}")
    boolean trueOrFalse;

    @Value("${my.value}")
    int myValue;

    @Value("#{${my.value} eq 100}")
    boolean isMyValue100;

    @Value("#{sample.data}")
    int sampleData;
```

## 스프링 AOP
Aspect - Oriented Programming (AOP)는 OOP를 보완하는 수단으로, 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법

* AOP 주요 개념
  * Aspect : 흩어진 관심사
  * Target : Ascpect를 적용하는 매체 (class, method)
  * Advice : 실직적인 부가 기능을 실행하는 구현체
  * Join Point와 Pointcut : advice를 실행할 위치, 포지션, 실행 전후 처리 등을 설정하는 기능
* AOP 구현체
  * 자바
    * AspectJ
    * 스프링 AOP
* AOP 적용 방법
  * 컴파일
  * 로드 타임
  * 런타임

## 프록시 기반 AOP

* 스프링 AOP 특징
  * **프록시 기반의 AOP**구현체
  * **스프링 빈에만 AOP를 적용**할 수 있다.
  * 모든 AOP기능을 제공하는 것이 목적이 아니라, 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는것이 목적.
* [프록시 패턴](https://jdm.kr/blog/235)
  * 기존 코드의 변경없이 접근제어 또는 부가 기능을 추가할 수 있는 대리자 패턴
* 문제점
  * 매번 프록시 클래스를 작성해야 하는가 ?
  * 여러 클래스, 여러 메소드에 적용하려면 ?
* 스프링 AOP
  * 스프링 IoC컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용하여 여러 복잡한 문제 해결
  * 동적 프록시 : 동적으로 프록시 객체를 생성하는 방법
    * 자바가 제공하는 방법은 인터페이스 기반 프록시 생성
    * CGlib는 클래스 기반 프록시도 지원
  * 스프링 IoC : 기존 빈을 대체하는 동적 프록시 빈을 만들어 등록 시켜준다.
    * 클라이언트 코드의 변경이 없다.
    * `public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware`

## @AOP
애노테이션 기반의 스프링 @AOP

* Aspect 정의
  * @Aspect
  * 빈으로 등록해야 하니깐 @Component 추가
* 포인트 컷 정의
  * @Pointcut(표현식)
  * 주요 표현식
    * execution
    * @annotaion
    * bean
  * 포인트 컷 조합
    * `&&, ||, !`
* 어드바이스 정의
  * @Before
  * @AfterReturning
  * @AfterThrowing
  * @Around

```java
    //Annotation 사용시 Retentionpolicy를 class 이상으로 적용해야 한다.Class파일까지 남아있어야 하기 때문에
    // 기본값 class
    @Retention(RetentionPolicy.CLASS)
    @Target(ElementType.METHOD)
    @Documented
    public @interface PerfLogging {}

    @PerfLogging
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("createEvent");
    }

    @Around("@annotation(PerfLogging)")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
        long begin = System.currentTimeMillis();
        Object proceed = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return proceed;
    }

    @Before("bean(simpleEventService)")
    public void hello() {
        System.out.println("Hello");
    }
```