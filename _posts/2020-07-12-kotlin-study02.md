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

### 접근 레벨
* internal : internal은 모듈 내 어디에서든 새로운 클래스 인스턴스르 생성할 수 있음을 의미
* private : private으로 설정한 클래스는 이를 정의한 파일 스코프 내부에서만 접근 가능
* protected : protected 접근 레벨은 오직 서브 클래스에서만 사용할 수 있다. 파일 레벨의 타입을 선언하는 경우 protected를 사용할 수 없다.

### 중첩 클래스
```kotlin
class OuterClass {
  class NestedClass {

  }
}
```
* 중첩 클래스에서도 접근 레벨을 설정할 수 있다. 중첩 클래스를 private로 설정하면 NestedClass는 OuterClass 스코프 내부에서만 생성 가능.
* 자바에서는 정적 클래스와 비 정적 클래스, 이렇게 두 가지 형태의 중첩 클래스를 지원한다.
  static 키워드를 사용해 선언한 중첩 클래스는 정적 중첩 클래스라 부르고 비 정정으로 선언한 클래스는 내부 클래스라고 부른다. 중첩 클래스는 해당 클래스를 둘러싼 클래스의 멤버로 간주한다.
  ```kotlin
    class Outer {
      static class StaticNestedClass {}
      class Inner {}
    }
  ```
* 코틀린에는 자바에서의 this 보다 더욱 강력한 this@label이란 표현식이 존재한다. 레이블 구조를 사용하면 this를 사용해 바깥 스코프를 참조 할 수 있게된다.
  ```kotlin
    class A {
      private val somefield: Int = 1
      inner class B {
        private val somefield: Int = 1
        fun foo(s: String) {
          println(this.somefield)
          println(this@B.somefield)
          println(this@A.somefield)
        }
      }
    }
  ```
### 열거형 클래스
* 열거형은 클래스의 구체적인 타입으로, 주어진 enum 타입 변수는 미리 정의된 상수로 제한된다.
```kotlin
  enum class Day {
    MONDAY, TUESDAY. WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
  }
```

### 정적 메소드와 컴패니언 오브젝트
* 코틀린은 자바와는 달리 클래스를 위한 정적 메소드를 지원하지 않는다. 
* 코틀린에서 정적 메소드는 클래스 레벨이 아닌 패키지 레벨에 정의하는것이 바람직하다.
```kotlin 
fun showFirst(input: String) {
  if (input.isEmpty()) throw IllegalArgumentException()
  return input.first()
}
```
* 위 코드를 실행하게 되면 코틀린 컴파일러는 새로운 final 클래스를 만들게 되고 정의한 함수가 추가되어 있는것을 확인할 수있다.
* 코틀린은 스칼라에서 발견한 싱글톤 개념을 가지고 왔다. 
  ```kotlin
    object Singleton {
      private var count = 0
      fun doSomeThing(): Unit {
        println(++count)
      }
    }
  ```
* 코틀린에서도 자바에서 호출하는 것 처럼 정적 메소드를 호출할 수 있는 방법이 있다. 이를 위해선 객체를 클래스 안에 위치 시킨 다음, 이를 컴패니언 오브젝트로 지정해야 한다.
  ```kotlin
    // factory pattern 
    interface StudentFactory {
      fun create(name: String): Student
    }

    class Student private constructor(val name: String) {
      companion object : StudentFactory {
        override fun create(name: String) : Student {
          return Student(name)
        }
      }
    }
  ```
* 새로운 인스턴스를 만들기 위해선 `Student.create("name")` 을 선언해야 한다. `Student.create는 Student.Companion.create 와 같은 코드를 작성하는 축약형이다.`

### 인터페이스
* 인터페이스는 계약에 불과하며, 연관된 기능의 집합에 대한 정의를 가지고 있다. 자바 8 버전과 마찬가지로, 코틀린 인터페이스도 추상 메소드를 선언하는것 뿐만 아니라 메소드 구현체를 가질 수 있다.
  추상 클래스와는 달리, 인터페이스는 상태를 가질수는 없으나 프로퍼티를 가질수는 있다.
  ```kotlin
    interface Document {
      val version: Long
      val size: Long

      val name: String
      get() = "NoNmae"
      
      fun save(input: InputStream)
      fun load(stream: OutputStream)
      fun getDescription(): String { return "Document $name has $size" }
    }
  ```
* 위 인터페이스는 프로퍼티 3개와 메소드 3개를 정의하고 있다.

### 상속
* 상속은 객체지향 프로그래밍의 핵심이다. 상속은 기존 클래스를 재활용 또는 확장하여 행위를 수정한 새로운 클래스를 생성하게 해준다. 
  기존 클래스를 슈퍼 클래스(또는 부모 클래스)라 부르고, 생성된 새로운 클래스를 파생 클래스 라고 부른다.
  ```kotlin
    enum class CardType {
      VISA, MASTERCARD, AMEX
    }
    
    open class Payment(val amount: BigDecimal) 
    class CardPayment(amount: BigDecimal, val number: String, val expiryDate: DateTime, val type: CardType) : Payment(amount)
  ```
* 위 코드를 살펴보면 Payment 클래스는 open이라는 키워드로 정의하였다. open 키워드를 통하여 Payment클래스는 상속이 가능하다고 나타내고, 코틀린 설계자들은 기본적으로 클래스는 상속에 닫혀있다고 설계하였다.
* CardPayment를 보면 `: Payment` 를 사용하였는데, `이는 Payment를 확장한 CardPayment` 로 해석할 수 있다.
* 앞서 코드에서는 CardPayment 클래스는 주 생성자를 가지고 있다. 그러므로 Payment(amount)처럼 부모 클래스의 생성자를 호출 하였다. 만약 주 생성자가 없는 클래스에서는 어떻게 상속을 구현할까 ?
  ```kotlin 
    class ChequePayment : Payment {
      constructor(amount: BigDeciaml, name: String) : super(amount) {
        ...
      }
    }
  ```
* 주 생성자를 생성하지 않았기 때문에 두 번째 생성자 정의에서 부모 생성자를 호출 하고 있다.
* 하나의 클래스는 상속을 하나만 할 수 있지만 인터페이스는 여러개 구현 할 수 있다. 
  ```kotlin
    interface Drivable {
      fun drive()
    }
    
    interface Sailable {
      fun saill()
    }

    class AmphibiousCar(val name: String) : Drivable, Sailable {
      override fun drive() {
        println("")
      }

      override fun saill() {
        println("")
      }
    }
  ```