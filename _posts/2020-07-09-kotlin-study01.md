---
title: Kotlin Study - 기본기
description: 자바와 코틀린의 차이점, 기본기 정리
categories:
 - dev
tags:
 - kotlin
comments: true
---
> Kotlin Study - 코틀린 기본기

`코틀린 프로그래밍 책을 공부하며 정리한 글입니다.` 

## 코틀린 기본기

### var와 val

* 코틀린에는 변수를 선언하기 위한 두가지 키워드가 존재.
  * var
    ```kotlin
      // 생성과 동시에 초기화
      var name = "kotlin"
      
      // 생성 후 초기화
      var name: String
      name = "kotlin"
      
      // var로 선언한 변수는 값을 변경할 수 있다.
      var name = "dobby"
      name = "dongchul"
    ```
  * val
    * 읽기 전용 변수를 선언하기 위해선 val 키워드 사용, 자바에서 final 변수를 선언하는것과 같다.
    * 생성과 동시에 초기화를 해줘야한다.
    * 읽기 전용 변수는 인스턴스 자신이 불변이 되는것을 의미하지 않음. 재할당을 막는 키워드 (java도 동일하다.)
    ```kotlin
    val name = "kotlin"
    ```

### 타입 추론
* 위 예제에서 변수를 초기화 시 타입을 포함하지 않을것을 확인할 수 있다. 이는 변수를 선언하는 과정에서 타입 추론을 통하여 적절한 타입을 찾아주기 때문이다.
  이같은 기능은 java에서도 11 버전이상 지원 된다.
* 값과 변수만 타입추론을 사용하지 않고 타입추론은 함수 시그니처로 매개변수 타입을 추론하는 클로저에서도 사용한다. 
  ```kotlin
    fun plusOne(x: Int) = x + 1
  ```
* 때로는 :을 이용하여 명시적으로 선언하는것이 도움이 된다.
  ```kotlin
    val explicitType: Number = 12.3
  ```

### 기본 타입
* 자바와 코틀린 간의 큰 차이점은 코틀린은 모든 것이 객체라는 것이다. 다시 말하여 코틀린에서는 자바에서 지원하는 원시타입 (boolean, int)이 존재하지 않는다.
  코틀린은 원시타입을 모두 객체로 승격시킴으로써 언어에서 래퍼 객체의 필요성을 전부 제거 했다.
* 코틀린 컴파일러는 성능을 이유로 기본 타입을 가능하면 JVM 원시 타입으로 다시 매핑하려 한다. 때로는 값이 박싱되어야 하는데(Nullable, Generic) 이러한 타입은 객체를 요구하는 곳에 자동으로 위치하게 된다.
  > 박싱된 각기 다른 두 값은 같은 인스턴스를 사용하지 않기 때문에, 박싱된 값에 대하여는 참조 동등성을 보장하지 않는다.
* 숫자

| 타입 | 길이 |
|:---|:---|
|Long|64|
|Int|32|
|Short|16|
|Byte|8|
|Double|64|
|Float|32|

```kotlin
  // 정수 기본값
  val int = 123
  val long = 123456L
  // 부동소수점 기본값
  val double = 12,34
  val float = 12.34F
  val hexadeciaml = 0xAB
  val binary = 0b010101
```

* 코틀린은 자동으로 숫자를 확장하지 않기 때문에 반드시 명시적인 확장이 필요하다.
  ```kotlin
    val int = 123
    val long = int.toLong()
  ```

### Boolean
* 불리언은 좀더 표중화 되어 있고, 부저으 논리곱, 논리합 연산자를 지원.

### 문자 
* char 는 단일 문자를 나타낸다. 문자 리터럴은 'A', 'Z'처럼 단일 따옴표 사용.

### 문자열
* 문자열은 자바와 마찬가지로 불변이다. 문자열 리터럴은 이중 또는 삼중 따옴표를 사용하여 생성한다.
* 이중 따옴표를 사용하면 이스케이프된 문자열을 생성
  ```kotlin
    val string = "string with \n new line"
  ```
* 삼중 따옴표는 원시 문자열을 생성, 원시 문자열에서는 이스케이프가 필요하지 않고, 모든 문자를 포함한다.
  ```kotlin
    var rawString = """
    raw string is super useful for strings that span many lines"""
  ```
* 또한 문자열은 for 루프에서 사용가능한 이터레이터 함수를 제공

### 배열
* 코틀린에서는 라이브러리 함수인 arrayOf()를 사용하여 배열을 생성할 수 있다.
  ```kotlin
    var array = arrayOf(1, 2, 3)
  ```
* 다른 방법으론 초기 크기와 각 요소를 생성하는 데 사용되는 함수로 부터 배열을 생성할 수 있다. 
  ```kotlin
    var perfectSquares = Array(10) {k -> k * k}
    // 0, 1, 4, 9 ...
  ```
* 자바와는 다르게 코틀린은 언어차원에서 배열이 특별하진 않다, 배열은 일반적인 컬렉션 클래스일 뿐이다. 
  배열의 인스턴스는 이터레이터와 get, set함수를 지원하며 괄호 문법도 지원한다. 
* 코틀린 JVM에선 원시타입으로 표현될 박싱타입을 피하고자 원시타입에 특화된 배열 클래스를 제공한다. 
  * ByteArray, CharArray, ShortArray, IntArray, LongArray, BooleanArray, FloatArray, DoubleArray 지원

### 패키지
* 패키지는 네임스페이스 단위로 코드를 나눌 수 있게 해준다. 
  ```kotlin 
    package me.study.kt
    class Foo
    fun bar(): String = "bar"
  ```
* 패키지명은 클래스, 객체, 인터페이스 또는 함수에 대한 **정규화된 이름**을 제공하기 위하여 사용
* 위 예제에서 Foo클래스는 me.study.kt.Foo라는 정규화된 이름을 가지며, 최상위 함수인 bar는 me.study.kt.bar라는 정규화된 이름을 가진다.

### 임포트명 변경
* 다른 패키지에서 동일한 이름의 클래스를 사용한다면 alias를 설정할 수 있다.
  ```kotlin
    import me.study.kt.Foo
    import me.study.java.Foo as JavaFoo

    fun doubleFoo() {
      val foo = Foo()
      val javaFoo = JavaFoo()
    }
  ```

### 문자열 템플릿
* 코틀린은 문자열 합치기도 사용가능하지만 많은 언어에서 지원하고 있는 문자열 템플릿 기능을 지원한다.
* 값이나 변수에 접두사로 $를 사용하여 간단하게 추가할 수 있다.
  ```kotlin
    val name = "Sam"
    val str = "hello $name"
  ```
* 임의의 표현식 같은 경우 접두사로 $를 추가하고 괄호 {}로 식을 감싸는 방법으로 추가할 수 있다.
  ```kotlin
    val name = "Sam"
    val str = "hello $name. Your name has ${name.length} characters"
  ```

### 범위
* 범위는 시작하는 값과 끝나는 값의 사이의 간격으로 정의, 비교가능한 타입은 범위를 생성할 수 있으며 .. 연산자를 사용해 생성
* 생성한 범위는 in 연산자를 사용하여 검사, 범위의 포함되기 위해선 시작값이 값이 크거나 같고, 끝나는 값보다 작거나 같아야 한다.
  ```kotlin
    val aToz = "a" .. "z"
    val isTrue = "c" in aToz
    val oneToNine = 1 .. 9
    val isFalse = 11 in oneToNine
  ```
* 정수형 범위(int, long, char)역시 for루프에서 사용할 수 있다.
* 연산자로 처리할 수 없는 범위를 생성하기 위한 라이브러리 함수도 존재, 예를들면 downTo(0)는 숫자를 하나씩 내리는 순으로 범위 생성하고
  rangeTo()는 하나씩 숫자를 올리는 순으로 범위 생성한다. 두 함수 모두 숫자 타입에 확장 함수형태로 정의되어 있다.
  ```kotlin
    val countingDown = 100.downTo(0)
    val rangeTo = 10.rangeTo(20)
  ```
* 범위를 생성하고 나면 새로운 범위를 반환할 수 있다. step()함수를 사용하여 범위에 있는 연속적인 항의 델타 값을 변경할 수 있다.
  ```kotlin
    val oneToFifty = 1..50
    val oddNumbers = oneToFifty.step(2)
  ```
* step() 함수에 음수 값을 사용해 숫자가 감소하는 범위를 만드는건 불가. 마지막으로 reversed()함수를 사용하면 범위를 반전시킬 수 있다.
  reversed() 함수는 시작값과 끝나는 값이 바뀐 새로운 범위를 리턴하며 단계값은 갑소하는 값이 된다.
  ```kotlin
    val countingDownEvenNumbers = (2..100).step(2).reversed()
  ```

### 루프
* 코틀린은 여러 언어에서 볼 수 있는 일반적인 루프 구조인 while 루프와 for 루프를 지원한다. while 루프의 경우 다른 언어에서의 사용법과 동일하다.
* 코틀린의 for 루프는 이터레이터라는 이름의 함수나 확장 함수를 정의한 객체를 반복하는데 사용된다. 모든 컬렉션은 이 함수를 제공한다.
  ```kotlin
    val list = listOf(1, 2, 3, 4)
    for (k in list) {
      println(l)
    }

    val set = setOf(1, 2, 3, 4)
    for (k in set) {
      println(k)
    }
  ```
* in 연산자는 항상 for 루프와 함께 사용된다. 연속적인 범위는 컬렉션을 지원하는 것뿐만 아니라 인라인 또는 바깥에서 정의하는것도 직접 지원한다.
  ```kotlin
    val oneToTen = 1..10
    for (k in oneToTen) {
      for (j in 1..5) {
        println(k * j)
      }
    }
  ```
  > 컴파일러는 범위를 특별한 방법으로 처리하며, JVM에서 직접 지원하는 인덱스 기반 for루프로 컴파일 한다. 이로 인해 이터레이터 객체를 생성함으로써 발생하는 성능상의 불이익을 피할 수 있다.
* 객체를 for 루프에서 사용하기 위하여 iterator라 불리는 함수를 구현해야 한다. 이 함수는 다음 두 함수를 제공하는 객체의 인스턴스를 반환해야 한다.
  * operator fun hasNext(): boolean
  * operator fun next(): T
* 코틀린은 표준 String 클래스도 iterator 확장 함수를 제공하기 때문에 for 루프에서 사용할 수 있다.
* 배열은 indices라는 확장 함수를 갖고 있으며, 이 함수는 배열의 인덱스를 반복하는데 사용할 수 있다.
  ```kotlin 
    for (index in array.indices) {
      println("Element $index is ${array[index]})
    }
  ```
  > 컴파일러는 배열에 대하여도 범위 루프와 마찬가지로 성능상의 불이익을 피하고자 배열을 일반적인 인덱스 기반의 for 루프로 컴파일한다.

### 예외 처리
* 코틀린에서 모든 예외는 Unchekced Exception이다. 
  * [CheckedException_UncheckedException](https://cheese10yun.github.io/checked-exception/)
* 예외를 처리할 때는 자바와 마찬가지로 try, catch, finally 블록을 사용할 수 있다.

### 클래스 인스턴스화
* 코틀린에선 객체를 생성할 때 new 키워드를 사용하지 않는다. 
  ```kotlin
    val file = File("/etc/nginx/nginx.conf")
    val date = Bigdecimal(100)
  ```

### 참조 동등성과 구조 동등성
* 두 객체가 메모리상에서 동일한 객체를 가리키고 있는 경우 -> 참조 동등성
  * 두 참조가 같은 인스턴스를 확인하기 위해선 === 연산자를 사용한다.
* 메모리상에서 두 객체는 다른 객체이지만 서로 같은 값을 가지고 있는 경우 -> 구조 동등성
  * 두 객체가 같은 값을 사용하기 위하연 == 연산자를 사용한다.
  * 이러한 함수 호출은 모든 클래스가 반드시 정의해놓고 있는 equals 함수를 사용하는 것으로 전환된다. 이는 자바에서 == 연산자가 사용되는 방법과는 다름을 명심해야한다.
    자바에서 == 연산자는 참조 동등성을 의미하며 이는 일반적으로 피해야하는 사항이다.

### 가시성 제어자 (접근 제어자)
* 가시성 제어자는 public, internal, protected, private 네가지가 있다. 제어자를 입력하지 않는다면 기본값은 public 제어자이다. 
  자바의 경우 기본값은 package-private 이다. 
* private
  * private로 정의한 최상위 함수, 클래스 또는 인터페이스는 오직 같은 파일에서만 접근할 수 있다.
  * 클래스, 인터페이스 또는 객체 내부에 있는 private 함수나 프로퍼티는 오직 같은 클래스나 인터페이스 또는 객체의 다른 멤버로만 접근할 수 있다.
* protected 
  * 최상위 함수, 클래스, 인터페이스 그리고 객체는 protected로 선언할 수 없다. 클래스나 인터페이스 내부에서 protected로 선언한 함수나 프로퍼티는 
    해당 클래스 또는 인터페이스뿐만 아니라 서브클래스 멤버까지만 접근 가능.
* internal 
  * internal은 모듈 개념을 다룬다. maven이나 gradle 또는 intellij 모듈로 정의된다. 
  * internal로 정의한 코드는 같은 모듈에 있는 다른 클래스나 함수에서 접근 가능. internal은 전체에서 public처럼 동작하기 보단 모듈에서의 public 개념이다.
  ```kotlin
    internal class Person {
      fun age(): Int = 21
    }
  ```

### 표현식으로서의 흐름 제어
* 표현식은 값을 평가하는 구문이다. 다음 표현식은 true를 평가한다.
  ```kotlin
    "hello".startWith("h")
  ```
* 구문은 결과 값을 반환하지는 않는다.
  ```kotlin
    //값을 대입은 하지만 반환하지는 않는 코드
    val a = 1
  ```
* 자바에서의 if - else나 try - catch 와 같은 흐름 제어 블록은 구문이다. 이러한 구문은 값을 평가하지 않고 외부에서 변수를 초기화 해주어야 한다.
  ```java
    public boolean isZero(int x) {
      boolean zero;
      if (x == 0) 
        isZero = true;
      else 
        isZero = false;
      return isZero;
    }
  ```
* 반면 코틀린에서의 if - else나 try - catch 흐름 제어 블록은 표현식이다. 결과값을 직접 대입할 수 있고, 함수로부터 반환 할수도 다른 함수에 인자로 전달할 수 있다.
  ```kotlin
    val date = Date()
    val today = if (date.year == 2016) true else false

    fun isZero(x: Int): Boolean {
      return if (x == 0) true else false
    }

    val success = try {
      readFile()
      true
    } catch (e: IOException) {
      false
    }
  ```
  > if를 표현식으로 사용할 경우에는 반드시 else를 포함하여야 한다. 그렇지 않으면 컴파일 에러가 발생할 것이다.

### 널 문법
* 코틀린에서는 널을 지정할 수 있는 변수를 ?와 함께 선언할 것을 요구한다.
  ```kotlin
    var str: String? = null
  ```
* 타입 확인과 형변환
  * is 연산자를 사용하여 타입을 확인한다. is 연산자는 자바의 instanceOf 연산자와 같은 기능을 함
    ```kotlin
      fun isString(any: Any): Boolean {
        return if (any is String) true else false
      }
    ```
### 똑똑한 형변환
* 자바에서의 형변환은 명시적으로 수행해야 한다.
  ```java
    public void printStringLength(Object obj) {
      if (obj instanceOf String) {
        String str = (String) obj;
        System.out.print(str.length());
      }
    }
  ```
* 코틀린 컴파일러는 Smart cast를 통하여 암시적으로 타입을 형변환 하게 된다.
  ```kotlin
    fun printStringLength(any: Any) {
      if (any is String) {
        println(any.length)
      }
    }
  ```

### 명시적 형변환
* 명시적으로 타입을 변환 하기 위해선 as 연산자를 사용한다. 형변환을 하지 못하는 경우 ClassCastException을 발생
  ```kotlin
    fun length(any: Any): Int {
      val string = any as String
      return string.length
    }
  ```
* Null이 가능한 값으로 형 변환을 하기 위해선 Null을 허용하는 변수로 선언해주어야 한다.
  ```kotlin
    val string: String? = any as String
  ```
* 형변환이 실패한 경우 널 값을 대신 전달 받고 싶으면 안전한 형변환 연산자인 as? 를 사용할 수 있다.
  ```kotlin
    val any = "/home/users"
    val string: String? = any as? String // string
    val file: File? = any as? File // null
  ```
### when 표현식
* when 에서는 마지막 분기가 else가 되도록 강제한다.
```kotlin
  fun whatNumber(x: Int) {
    when (x) {
      0 -> println("x is zero")
      1 -> println("x is 1")
      else -> println("x is neither 0 nor 1")
    }
  }
```
* if ... else 및 try ... catch와 마찬가지로 when도 표현식으로 사용될 수 있다.
  ```kotlin
    fun isMinOrMax(x: Int): Boolean {
      val isZero = when (x) {
        Int.MIN_VALUE -> true
        Int.MAX_VALUE -> true
        1,2 -> true // 분기가 같은 경우 콤마를 사용하여 묶을 수 있다. 
        else -> false
      }
      return isZero
    }
  ```
* when은 각 조건에 상수만 매치하도록 제한하지 않는다. 반환하는 타입이 매치하는 타입과 같으면 어떤 함수든 사용 가능하다.
  ```kotlin
    fun isAbs(x: Int): Boolean {
      return when (x) {
        Math.abs(x) -> true
        else -> false
      }
    }
  ```
* when은 범위 연산자 역시 지원한다. in 연산자를 사용해 값이 범위 안에 포함되어 있는지를 확인할 수 있으며, 만약 포함되어 있다면 해당 조건은 true로 평가된다.
  ```kotlin
    fun isSingleDigit(x: Int): Boolean {
      return when (x) {
        in -9..9 -> true
        else -> false
      }
    }

    fun isDieNumber(x: Int): Boolean {
       return when (x) {
         in listOf(1, 2, 3, 4, 5, 6) -> true
         else -> false
       }
    }

    fun startsWithFoo(any: Any): Boolean {
      return when (any) {
        is String -> any.startWith("Foo")
        else -> false
      }
    }
  ```
* 인자가 없는 when은 if ... else 절을 대체하게 된다.
  ```kotlin
    fun whenWithourArgs(x: Int, y: Int) {
      when {
        x < y -> println("x is less than y")
        x > y -> println("x is greater than y")
        else -> println("x must equal y")
      }
    }
  ```

### 타입 체게
* 코틀린에선 최상위 타입을 Any라고 부른다. 이는 자바의 오브젝트 타입과 유사하다.
* Any 타입은 toString, hashCode, equals 메소드를 정의하고 있다. 또한 apply, let, to와 같은 확장 함수도 정의하고 있다.
* Unit 타입은 자바의 void 와 유사하다. 유닛은 싱글톤 인스턴스와 함께 적절한 타입 Unit이나 ()로 나타낸다. 함수가 유닛을 반환하도록 정의되어 있다면, 이 함수는 싱글톤 유닛 인스턴스를 반환할것이다.
* Nothing은 값을 지니지 않는 타입이다. Any가 모든 타입의 슈퍼클래스이고 Nothing은 모든 타입의 서브클래스이다. 

