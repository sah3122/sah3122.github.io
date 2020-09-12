---
title: 스프링 클라우드 서비스 이해 - Zuul Proxy
description: Spring Cloud Zuul
categories:
 - dev
 - spring
 - cloud
tags:
 - java
comments: true
---
> Spring Cloud Zuul에 대해서 간단히 알아보자.

작성중인 글입니다.

# Spring Cloud Zuul 이란 ?

Netflix에서 개발한 MSA 환경에서 Routing 과 Filtering 을 위한 서비스 라고 이해 하면 될것 같다.  
Zuul을 사용하면 아래와 같은 기능들을 설정할 수 있다고 합니다.
* Authentication
* Insights
* Stress Testing
* Canary Testing
* Dynamic Routing
* Service Migration
* Load Shedding
* Security
* Static Response handling
* Active/Active traffic management


# 개발환경 설정
기본적으로 아래와 같은 디펜던시를 추가.
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

ZuulProxy를 사용한다고 선언 해주어야 한다.
```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulproxyApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulproxyApplication.class, args);
    }

}
```

# Zuul Proxy 설정
dean-service 경로로 들어오는 요청에 대해서 지정한 api 서버로의 routing 을 설정해보자.
```yaml
zuul:
  routes:
    dean-service:
      path: /dean-service/**
      url: http://localhost:8081/dean-service
```
path와 같은 경우 /* 와 같이 특정 url 에 한정해서 설정하는 것 보단 /** 와 같이 계층적으로 모든 api가 매칭 될 수 있도록 설정하는것을 권장하고 있다.  
위 설정을 진행하고 미리 만들어둔 서버로 요청이 잘 가는지 테스트 해보자.
![Zuul 예제](/assets/images/post/spring-cloud-zuul/zuul-example1.png)
정상적으로 routing이 되는것을 확인할 수 있습니다 :) 

# Provider 설정
Response에 대한 응답 처리를 하고 싶을때 사용. 공식 문서에 나와 있는 예제는 Timeout, Internal Server Error 가 발생하였을때 Custom 한 응답을 내려 줄 수 있도록 설정하고 있습니다. 
```java
@Component
public class DeanServiceProvider implements FallbackProvider {
    @Override
    public String getRoute() {
        return "dean-service";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        if (cause instanceof HystrixTimeoutException) {
            return response(HttpStatus.GATEWAY_TIMEOUT);
        } else {
            return response(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    private ClientHttpResponse response(final HttpStatus status) {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return status;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return status.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return status.getReasonPhrase();
            }

            @Override
            public void close() {
            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```
