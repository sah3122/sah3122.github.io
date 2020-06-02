---
title: Checked Exception과 UnCheckedException의 차이
description: Checked Exception과 UnCheckedException의 차이 정리
categories:
 - dev
tags:
 - Java
---
> Checked Exception과 UnCheckedException의 차이점 정리 및 실험

# Checked Exception 과 UnCheckedException의 차이
![image](https://1.bp.blogspot.com/-oTNPzMsGKJs/VVjYdQCGKXI/AAAAAAAAAH4/Wh9uLNtsZos/s1600/overview%2Bof%2Bexception.png)


* Checked Exeption 
	* 어플리케이션에서 반드시 예외 처리를 해야 한다. 하지 않을시 Runtime Error 가 발생
	* Transaction Rollback이 되지 않는다. 
	* 대표적으로 IO,SQLException이 있다.
	* try-catch 로 예외를 처리 하거나 상위 메소드로 예외 처리로직을 위임 할 수 있다.

* UnCheckedException
	* 어플리케이션에서 예외처리를 강제 하지 않는 에러.
	* Transaction Rollback 처리
	* 대표적으로 Runtime Exception, NullPointer, IllegalArgumentException 등이 존재 한다.
	* 명시적 예외 처리 로직이 필요하지 않는 에러

* 두가지 Exception을 나누는 기준은 위 표와 같이 Runtime Exception의 상속 여부 이다.
	* RuntimeException 을 상속받는 Exception 모두가 Unchecked Exception이라고 할 수 있다.
* 간단한 예시를 통하여 Unchecked Exception 의 Rollback을 확인해보자.

```java
@Transactional
public void insertWithUnCheckedException() {
    User user = new User("name");
    userRepository.save(user);

    throw new RuntimeException();
}

```

위와 같이 Service 로직에서 데이터를 저장 후 Unchecked Exception 을 던지게 되면 어떤 일이 일어날지 간단한 테스트 실행.

```java
@SpringBootTest
@Commit
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void uncheckedException() {
        assertThatExceptionOfType(RuntimeException.class).isThrownBy(() -> userService.insertWithUnCheckedException());
    }
}
결과 로그
2020-03-24 18:39:02.118  INFO 83948 --- [           main] p6spy                                    : #1585042742118 | took 4ms | statement | connection 4| url jdbc:h2:mem:testdb
2020-03-24 18:39:02.134  INFO 83948 --- [           main] p6spy                                    : #1585042742134 | took 0ms | rollback | connection 4| url jdbc:h2:mem:testdb
```
* 위 로그를 보시면 UncheckedException 이 발생하면 rollback 이 정상적으로 일어난 것을 확인 할 수 있습니다.

* CheckedException이 발생 하였을때 정말 Rollback이 되지 않을까 ?   

마찬가지로 간단한 테스트를 실행해보자.

```java
@Transactional
public User insertWithCheckedException() {
    User user = new User("name");
    User save = userRepository.save(user);
    try {
        throw new Exception();
    } catch (Exception e) {
        log.error("throw exception");
    }
    return save;
}
```

User를 저장하고 강제로 Exception을 던져보았습니다. 

```java
@SpringBootTest
@Commit
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void checkedException() {
        User user = userService.insertWithCheckedException();
        assertThat(user.getId()).isNotNull();
    }
}
결과 로그
2020-03-24 18:41:41.851 DEBUG 83958 --- [           main] org.hibernate.SQL                        : 
    call next value for hibernate_sequence
2020-03-24 18:41:41.861  INFO 83958 --- [           main] p6spy                                    : #1585042901861 | took 4ms | statement | connection 4| url jdbc:h2:mem:testdb
call next value for hibernate_sequence
call next value for hibernate_sequence;
2020-03-24 18:41:41.876 ERROR 83958 --- [           main] m.s.q.exceptiontest.service.UserService  : throw exception
2020-03-24 18:41:41.885 DEBUG 83958 --- [           main] org.hibernate.SQL                        : 
    /* insert me.study.exceptiontest.domain.User
        */ insert 
        into
            user
            (name, id) 
        values
            (?, ?)
2020-03-24 18:41:41.889 TRACE 83958 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [name]
2020-03-24 18:41:41.889 TRACE 83958 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [BIGINT] - [1]
2020-03-24 18:41:41.890  INFO 83958 --- [           main] p6spy                                    : #1585042901890 | took 0ms | statement | connection 4| url jdbc:h2:mem:testdb
/* insert me.study.exceptiontest.domain.User */ insert into user (name, id) values (?, ?)
/* insert me.study.exceptiontest.domain.User */ insert into user (name, id) values ('name', 1);
2020-03-24 18:41:41.893  INFO 83958 --- [           main] p6spy                                    : #1585042901893 | took 0ms | commit | connection 4| url jdbc:h2:mem:testdb
;
```
* 위 로그를 보시면 정상적으로 CheckedException이 발생 하였지만 `m.s.q.exceptiontest.service.UserService  : throw exception`  User 데이터가 저장되는것을 확인 할 수 있습니다.
* CheckedException 이 발생 될 때 정상적으로 Rollback을 하기 위한 여러가지 방법이 있지만 저는 명시적인 UncheckedExecption을 던져주는 방식으로 Rollback이 되는지 실행 해보겠습니다. 

```java
@Transactional
public void insertWithCheckedException() {
    User user = new User("name");
    userRepository.save(user);
    try {
        throw new Exception();
    } catch (Exception e) {
        log.error("throw exception");
        throw new RuntimeException();
    }
}
```

위 코드와 같이 CheckedException 이 던져질 경우 catch에서 명시적인 예외를 다시 던져주는 방식으로 실행 해 보겠습니다. 

```java
@SpringBootTest
@Commit
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    void checkedException() {
        assertThatExceptionOfType(RuntimeException.class).isThrownBy(() -> userService.insertWithCheckedException());
    }
}

결과 로그
2020-03-24 18:48:18.409 ERROR 83989 --- [           main] m.s.q.exceptiontest.service.UserService  : throw exception
2020-03-24 18:48:18.409  INFO 83989 --- [           main] p6spy                                    : #1585043298409 | took 0ms | rollback | connection 4| url jdbc:h2:mem:testdb

;
```

위 로그와 같이 명시적인 에러를 던져줌으로서 정상적으로 Rollback이 되는것을 확인 할 수 있습니다.

> Exception의 종류에 따라 트랜젝션 여부가 다르다는것을 알고 있었지만 실제로 어떻게 처리하는지, 정말 Rollback이 안되는지 등에 대한 실행을 통한 결과 검증 과정을 정리 하였습니다.   
> 지금까지 Exception 처리시 별다른 처리를 하지 않고 단순 로그만 남기는 경우가 많이 있었는데 이번 테스트를 통하여 정확한 예외 처리 방법에 대한 학습을 할 수 있었습니다. :)

참고
* https://cheese10yun.github.io/checked-exception/
* https://1.bp.blogspot.com/ 
