---
title: Kotlin Study - 코틀린과 객체 지향 프로그래밍
description: 코틀린에서 객체 지향 프로그래밍을 어떤 방식으로 지원하는지 정리
categories:
 - dev
tags:
 - kotlin
comments: true
---
> Kotlin Study - 코틀린과 객체 지향 프로그래밍

`코틀린 프로그래밍 책을 공부하며 정리한 글입니다.` 

## 코틀린과 객체 지향 프로그래밍

* 모든 것은 객체다.
* 객체는 메시지를 보내고 받는 방식으로 통신한다.(객체 관점)
* 객체는 자신만의 메모리를 갖는다.(객체 관점)
* 모든 객체는 클래스의 인스턴스다.(반드시 객체여야 한다.)
* 클래스는 인스턴스를 위한 공유되는 행위를 갖는다.(프로그램 리스트에서 객체의 형태로)

> 코틀린은 위 내용을 모두 지원하며 현대적인 OOP 언어의 세 가지 기둥인 캡슐화, 상속, 다형성을 지원하고 있다.

### 클래스

* 클래스는 객체 지향 프로그래밍 언어에서 핵심적인 구성요소이다. 클래스는 타입의 행위와, 데이터를 나타낸다.
  ```kotlin
    class Deposit {
    }
  ```
* 코틀린은 자바와 다르게 같은 소스 파일 안에 여러 클래스를 정의 할 수 있다. 접근 지정자를 명시하지 않는다면 public 접근지정자가 정의된다. (java는 package-private)
* 위 예제에서는 기본 생성자를 가지고 있다. 새로운 생성자를 정의하고 싶다면 아래와 같이 할 수 있다.
  ```kotlin
    class Person constructor(val firstName: String, val lastName: String)

    fun main(args: Array<String>) {
      val person = Person("dongchul", "lee")
    }
  ```
* 위 코드에서의 constructor 키워드는 주 생성자를 의미하게 된다. 코틀린 컴파일러는 생성자 컨텍스트를 가지고 있으며 init 블록을 활용하여 주 생성자의 한 부분으로 코드를 동작 시킬 수 있다.
  ```kotlin
    class Person constructor(val firstName: String, val lastName: String?) {
      init {
        require(firstName.trim().length > 0) {"invalid argument"}
        ...
      }
    }
  ```
* 위 생성자 코드에서의 인자는 어떤 방식으로 정의 되는것인가 ? 매개변수인가 ? 그렇지 않다. 2개의 인자는 프로퍼티로 정의 된다. 자바 코드에서 예를 들면 아래와 같이 사용할 수 있게 된다.
  ```java
    Person person = new Person("dongchul", "lee");
    System.out.print(person.getFirstName());
    System.out.print(person.getLastName());
  ```
  
* 이름만 존재하고 성을 Nullable로 정의하는 생성자를 만들땐 this를 활용하여 아래와 같이 할 수 있다.
  ```kotlin
    constructor(firstName: String) : this(fristName, null)
  ```
* 생성자 인자에 접두사로 val이나 var를 반드시 붙일 필요는 없으며 getter 메소드가 필요없는 경우에는 다음과 같이 할 수 있다.
  ```kotlin
    class Person2(firstName: String, lastName: String) {
      private val name: String
      private val age: Int?

      init {
        this.name = "$firstName,$lastName"
        this.age = 10
      }

      fun getName(): String = this.name

      fun getAge(): Int? = this.age
    }
  ```
* 첫번째와는 달리 getName, getAge 두개의 메소드를 지원하게 될것이다. 
