---
title: 악취나는 코드를 개선해보자 - 1
description: 일급 함수를 이용한 코드 개선기
categories:
 - dev
tags:
 - java
 - cleancode
comments: true
---

```
이번 포스팅에서는 일급 함수를 이용하여 코드를 어떻게 개선할 수 있는지에 대해 알아보겠습니다. 
우선 일급 객체(first-class object) 또는 일급 시민(first-class citizen)의 정의를 가져와봤습니다. 
일급 객체가 되기 위한 조건으로 아래 3가지를 만족해야 합니다. 
```

1. 메소드의 인자로 전달 가능해야 한다. 
2. 변수에 할당 할 수 있어야 한다.
3. 리턴 값으로 반환 할 수 있어야 한다.


* 간략하게 정리하자면 우리가 일반적으로 만들고 사용해온 객체는 위 3가지 조건을 만족하니 일급 객체라고 할 수 있습니다. 

## 일급 함수란 무엇인가 ? 
```
함수형 프로그래밍의 관점에서는 함수를 마치 일반값처럼 사용해서 인수로 전달하거나, 결과로 반환받거나,
자료구조에 저장할 수 있음을 의미하고 일반값처럼 취급할 수 있는 함수를 일급함수라고 한다.
```

* 일반값이나 함수형 프로그래밍의 관점이라는 단어를 제외한다면, 위에서 정리한 일급 객체가 되기 위한 조건 3가지와 일치하는것을 알수 있습니다. 
* Java 8 이전엔 Java는 일급 함수를 지원하지 않았지만 지금은 Java도 일급 함수를 지원한다! 라고 이야기 할 수 있습니다.   
* Java가 일급 함수를 지원하는지 간략한 예제 코드를 통하여 알아보겠습니다. 

### 함수를 매개변수로 전달.

```java
// 정수형 리스트를 계산하는 클래스 정의.
public class Calculator {
    private final List<Integer> values;

    public Calculator(List<Integer> values) {
        this.values = values;
    }

    // 일반적인 메소드, 상태값을 모두 더하여 리턴한다. 
    public int calculate() {
        return values.stream()
                .reduce(0, (integer, integer2) -> integer + integer2);
    }
    
    // 함수를 매개변수로 전달 받는방법. 
    public int calculateByFunctional(Function<List<Integer>, Integer> calculator) {
        return calculator.apply(values);
    }
}

public class Main {

    public static void main(String[] args) {
	    Calculator calculator =
                new Calculator(List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

	    // 객체의 계산 메소드 호출
      int result = calculator.calculate();

      // 함수를 매개변수로 전달.
      int resultByFunction = calculator.calculateByFunctional((List<Integer> values) ->
          values.stream()
                .reduce(0, (integer, integer2) -> integer + integer2)
      );
    }
}
```

* 예제 코드를 확인해본결과 일급 객체가 되기 위한 첫번째 조건인 함수를 매개변수로 전달 가능함을 증명해보았습니다. 
* 나머지 조건들도 어떤 방식으로 구현 가능한지 알아보겠습니다. 


### 함수를 변수에 할당

```java
public static void main(String[] args) {
    Calculator calculator =
            new Calculator(List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

    // 함수를 변수에 저장
    Function<List<Integer>, Integer> sumAll = (List<Integer> values) -> values.stream()
                                                                              .reduce(0, (integer, integer2) -> integer + integer2);

    // 함수를 매개변수로 전달.
    int resultByFunction = calculator.calculateByFunctional(sumAll);
}
```

* 모든 값을 더하는 함수를 sumAll 이라는 **Function 타입의 변수에 저장**하여 매개 변수로 전달 하고 있습니다. 

### 함수를 리턴값으로 반환

```java
public class Calculator {

    public Function<List<Integer>, Integer> getCalculator() {
        return (values) -> values.stream()
                                 .reduce(0, (integer, integer2) -> integer + integer2);
    }
}

public class Main {

    public static void main(String[] args) {
        List<Integer> values = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        Calculator calculator = new Calculator();

        Function<List<Integer>, Integer> calculatorFunction = calculator.getCalculator();
        Integer result = calculatorFunction.apply(values);
    }
}
```



# 참고
* [일급객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)
* Modern Java In Action - 함수형 프로그래밍