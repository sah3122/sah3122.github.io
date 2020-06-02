---
title: SpringSecurity-WithSecurityContextFactory 소개
description: SpringSecurity-WithSecurityContextFactory 소개
categories:
 - dev
tags:
 - Java
 - Spring
 - Security
---
> 스프링 시큐리티를 사용하여 테스트 코드를 작성하는 경우 Authentication 정보를 자동으로 담아주지 못하여 테스트 코드 작성이 힘들어지는 경우가 있다. 이런경우에 테스트코드 작성을 편하게 할 수 있는 방법을 소개 합니다.  

# SpringSecurity-WithSecurityContextFactory 소개

> 해당 기능은 백기선님의 강의(*스프링과 JPA기반 웹 애플리케이션 개발*)를 참고 하였습니다. 

샘플 프로젝트는 Spring Boot + JPA + Spring Security 로 구성되어 있습니다.


### 샘플 코드 
테스트는 서비스 및 도메인의 단위 테스트가 아닌 통합 테스트를 진행한다고 가정합니다. 

```java
@Transactional
public CreateResponse create(CreateRequest createRequest) {
    Board board = createRequest.toEntity();
    Board savedBoard = boardRepository.save(board);
    return new CreateResponse(savedBoard);
}
```
단순히 Board Entity를 저장하는 기능을 제공하는 서비스 코드 입니다.  
위 코드를 검증하기 위하여 Test 를 작성할 경우 일반적인 환경에서는 정상적으로 동작하겠지만, Security가 포함된다면 예기치 못한 이슈가 발생할 수 있습니다. 

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.csrf()
            .disable()
        .authorizeRequests()
            .mvcMatchers("/sign-up").permitAll()
        .anyRequest()
            .authenticated()
        .and()
        .formLogin()
        .and()
        .httpBasic();
}
```
Security 설정입니다.  */sign-up* 을 제외한 모든 api는 인증된 사용자만 접근이 가능하도록 설정 해둔 경우 위 코드를 검증하기 위해선 몇가지 방법이 존재할 수 있습니다. 
Spring Security Reference를 보면 아래와 같은 기능들을 제공해주고 있습니다.  
https://docs.spring.io/spring-security/site/docs/current/reference/html5

1. @WithMockUser 사용
2. @WithUserDetails 사용
3. @WithSecurityContext 사용

하나씩 살펴보면 내용이 길어지므로 레퍼런스를 확인해주시면 될것 같습니다.   
간단하게 요약하자면 Security Filter를 통과 하기 위해선 위 2가지 기능만 사용하면 통과 할 수 있지만 지금 살펴볼 기능은 3번째 기능을 사용해야 편하고 간단하게 테스트 코드를 작성할 수 있습니다. 

```java
@MappedSuperclass
@Getter
public class BaseEntity extends BaseTimeEntity {
    @Column(updatable = false)
    private Long createdBy;
    private Long lastUpdatedBy;

    @PrePersist
    public void prePersist() {
        SecurityContext context = SecurityContextHolder.getContext();
        UserAccount userAccount = (UserAccount) context.getAuthentication().getPrincipal();
        createdBy = userAccount.getAccount().getId();
        lastUpdatedBy = userAccount.getAccount().getId();
    }

    @PreUpdate
    public void preUpdate() {
        SecurityContext context = SecurityContextHolder.getContext();
        UserAccount userAccount = (UserAccount) context.getAuthentication().getPrincipal();
        lastUpdatedBy = userAccount.getAccount().getId();
    }
}
```

위 코드와 같이 Entity가 저장되거나 수정되는 경우 자동으로 Authentication에서 값 저장하는 코드가 존재한다면, 테스트코드가 동작할 때 Authentication객체가 필요하게 됩니다.  

간단하게 생각하자면, 테스트를 실행하기전 Authentication객체를 만들고, SecurityContextHolder에 담아 테스트를 실행하면 되지만, 모든 테스트 클래스에서 중복된 코드가 발생하게 될것이므로 정신건강에 좋지 않을것 같습니다. 

이러한 기능을 간편하게 애노테이션으로 정의하여 사용하는 *WithSecurityContextFactory* 를 사용해 봅시다. 

```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithAccountSecurityContextFactory.class, setupBefore = TestExecutionEvent.TEST_EXECUTION)
public @interface WithAccount {
    String value();
}
```
우선 WithSecurityContext를 정의한 애노테이션을 작성합니다. 

```java
@RequiredArgsConstructor
public class WithAccountSecurityContextFactory implements WithSecurityContextFactory<WithAccount> {

    private final AccountService accountService;

    @Override
    public SecurityContext createSecurityContext(WithAccount withAccount) {

        String username = withAccount.value();
        String email = username + "@gmail.com";
        String password = "password";

        AccountCreateRequest accountCreateRequest = new AccountCreateRequest();
        accountCreateRequest.setEmail(email);
        accountCreateRequest.setPassword(password);
        accountCreateRequest.setNickname(username);

        accountService.saveAccount(accountCreateRequest);

        UserDetails userDetails = accountService.loadUserByUsername(email);
        Authentication authentication = new UsernamePasswordAuthenticationToken(userDetails,
                userDetails.getPassword(), userDetails.getAuthorities());
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        return context;
    }
}
```
그후 WithSecurityContextFactory을 구현한 클래스를 만든 뒤, Authentication객체를 생성, SecurityContext에 담아주는것으로 구현이 끝납니다. 

사용하는 방법도 간단합니다. 
```java
@DisplayName("보드 조회 - 단건")
@WithAccount("dongchul")
@Test
void findBoard() throws Exception {
    CreateRequest createRequest = new CreateRequest();
    createRequest.setTitle("title");
    boardService.create(createRequest);

    mockMvc.perform(get(BoardApiController.BOARD_URI + "/1"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("title").exists());
}
```
위와 같이 테스트 코드 실행시 만들어둔 애노테이션을 사용하기만 하면 테스트가 정상적으로 실행됩니다. 


