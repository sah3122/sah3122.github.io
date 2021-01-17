---
title: 악취나는 코드를 개선개선기 - Part 1
description: 일급 함수를 이용한 코드 개선
categories:
 - dev
tags:
 - java
 - cleancode
comments: true
---

# 일급 함수를 이용한 코드 개선기

```
일급 함수를 이용하여 코드를 어떻게 개선할 수 있는지에 대해 소개해드리겠습니다. 
우선 일급 객체(first-class object) 또는 일급 시민(first-class citizen)란 어떤 객체를 뜻하는것인지 알아야합니다. 
일반적으로 일급 객체가 되기 위한 조건으로 아래 3가지를 만족해야 합니다. 
```

1. 메소드의 인자로 전달 가능해야 한다. 
2. 변수에 할당 할 수 있어야 한다.
3. 리턴 값으로 반환 할 수 있어야 한다.

> 간략하게 정리하자면 우리가 일반적으로 만들고 사용해온 객체는 위 3가지 조건을 만족하니 일급 객체라고 할 수 있습니다. 

## 일급 함수란 무엇인가 ? 
```
함수형 프로그래밍의 관점에서는 함수를 마치 일반값처럼 사용해서 인수로 전달하거나, 결과로 반환받거나,
자료구조에 저장할 수 있음을 의미하고 일반값처럼 취급할 수 있는 함수를 일급함수라고 한다.
```

* 일반값이나 함수형 프로그래밍의 관점이라는 단어를 제외한다면, 위에서 정리한 일급 객체가 되기 위한 조건 3가지와 일치하는것을 알수 있습니다. 
* Java 8 이전엔 Java는 일급 함수를 지원하지 않았지만 지금은 Java도 일급 함수를 지원한다! 라고 이야기 할 수 있습니다.   
* Java가 어떻게 일급 함수를 지원하는지 간략한 예제 코드를 통하여 알아보겠습니다. 

### 함수를 매개변수로 전달.

```java
// 정수형 리스트를 계산하는 클래스 정의.
public class IntCalculator {
    private final List<Integer> numbers;

    public IntCalculator(List<Integer> numbers) { 
        this.numbers = numbers;
    }

    // 일반적인 메소드, 상태값을 모두 더하여 리턴한다. 
    public int calculate() {
        return numbers.stream()
                      .reduce(0, (integer, integer2) -> integer + integer2);
    }
    
    // 함수를 매개변수로 전달 받는방법. 
    public int calculateByFunction(Function<List<Integer>, Integer> calculator) {
        return calculator.apply(numbers);
    }
}

public class Main {

    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 10)
                                            .boxed()
                                            .collect(Collectors.toList());

            IntCalculator intCalculator =
                    new IntCalculator(numbers);

            // 객체의 계산 메소드 호출
        int result = intCalculator.calculate();

        // 함수를 매개변수로 전달.
        int resultByFunction = intCalculator.calculateByFunction((List<Integer> numbers) ->
            numbers.stream()
                    .reduce(0, (integer, integer2) -> integer + integer2)
        );
    }
}
```

* 예제 코드를 확인해본결과 일급 객체가 되기 위한 첫번째 조건인 함수를 매개변수로 전달 가능함을 증명해보았습니다. 
* 위 코드를 실행하는 경우 두 계산식 모두 55라는 결과를 반환하고 있습니다. 
* 일급 함수가 되기위한 나머지 조건들도 예제코드를 통해 구현 가능한지 알아보겠습니다. 


### 함수를 변수에 할당

```java
// 함수를 변수에 할당
private static Function<List<Integer>, Integer> SUM_ALL = (List<Integer> numbers) -> numbers.stream()
                                                                              .reduce(0, (integer, integer2) -> integer + integer2);

private static Function<List<Integer>, Integer> SUM_EVEN = (List<Integer> numbers) -> numbers.stream()
                                                                                              .filter(integer -> integer % 2 == 0)
                                                                                              .reduce(0, (integer, integer2) -> integer + integer2);

public static void main(String[] args) {
    List<Integer> numbers = IntStream.rangeClosed(1, 10)
                                        .boxed()
                                        .collect(Collectors.toList());

    int resultBySumAll = SUM_ALL.apply(numbers);

    int resultBySumEven = SUM_EVEN.apply(numbers);
}
```

* SUM_ALL, SUM_EVEN 이라는 함수를 변수에 할당하여 호출가능함을 알아보았습니다. 
* 자주 사용하고, 간단한 계산식 같은 기능들을 메소드로 정의하기 보다는 함수 객체로 정의하여 사용해보는것을 도전해보세요 :)

### 함수를 리턴값으로 반환

```java
public class IntCalculator {

    public Function<List<Integer>, Integer> getCalculator() {
        return (numbers) -> numbers.stream()
                                 .reduce(0, (integer, integer2) -> integer + integer2);
    }
}

public class Main {

    public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 10)
                                        .boxed()
                                        .collect(Collectors.toList());

        IntCalculator intCalculator = new IntCalculator();

        Function<List<Integer>, Integer> calculator = intCalculator.getCalculator();
        Integer result = calculator.apply(numbers);
    }
}
```

* 마지막으로 함수를 리턴값으로 반환하는 예제를 살펴보겠습니다. 
* IntCalculator 클래스의 getCalculator 메소드는 **Function<List<Integer>, Integer>** 타입의 함수를 리턴하는것을 확인할 수 있습니다. 
* 이처럼 함수 자체를 반환하는것이 어떤 의미가 있을지, 어떻게 활용할 수 있을지는 악취나는 코드 개선기 시리즈를 진행하며 천천히 알아가보겠습니다. 

## 일급 함수를 이용한 중복 코드 개선기

* 업무 진행 중 A, B를 수정할 수 있는 기존 기능에서 C, D, F 까지 수정 가능하도록 추가해달라는 요청이 들어왔다. 
* 기존 코드를 살펴보니 A, B, C, D, E는 수정 되는 주체만 다를뿐, 나머지 유효성 검사와 조건식이 같은 코드였습니다.   
  이미 A, B 기능에서는 중복 코드가 발생한 상태.. 복잡하게 생각하지 않고 C, D, E도 비슷한 구조로 개발한다면 크게 고민하지 않아도 되는 문제 였지만 `중복 코드`가 발생하는것을 지켜볼수 없어 수정되는 로직를 함수객체로 분리하여 전달하기로 결정하였습니다. 
* 설명이 이해가 되지 않는 분들을 위하여 간단한 예제 코드로 일급 함수를 매개변수로 전달하여 중복 코드 개선을 어떻게 하는지 살펴보겠습니다 :)

```java
public class PositiveNumberCalculator {

    public int sumAll(List<Integer> numbers) {
        boolean hasNegative = numbers.stream()
                           .anyMatch(integer -> integer < 0);

        if (hasNegative) {
            throw new IllegalArgumentException("음수가 포함될 수 없습니다.");
        }

        // 모든 값을 더한다.
        Integer total = numbers.stream()
                              .reduce(0, (integer, integer2) -> integer + integer2);

        if (total > 20) {
            return total;
        }
        
        return 0;
    }

    public int sumEven(List<Integer> numbers) {
        boolean hasNegative = numbers.stream()
                                     .anyMatch(integer -> integer < 0);

        if (hasNegative) {
            throw new IllegalArgumentException("음수가 포함될 수 없습니다.");
        }

        // 짝수인 값만 더한다.
        Integer total = numbers.stream()
                                .filter(integer -> integer % 2 == 0)
                                .reduce(0, (integer, integer2) -> integer + integer2);

        if (total > 20) {
            return total;
        }

        return 0;
    }

    public int sumOdd(List<Integer> numbers) {
        boolean hasNegative = numbers.stream()
                                     .anyMatch(integer -> integer < 0);

        if (hasNegative) {
            throw new IllegalArgumentException("음수가 포함될 수 없습니다.");
        }

        // 홀수인 값만 더한다.
        Integer total = numbers.stream()
                                .filter(integer -> integer % 2 == 1)
                                .reduce(0, (integer, integer2) -> integer + integer2);
        
        if (total > 20) {
            return total;
        }

        return 0;
    }
}
```

* 이번 예제 코드로는 양수를 더하는 계산기 클래스를 구현해보았습니다. 
* 전달 받은 리스트의 전체, 홀수, 짝수의 합을 구하는 예제인데 총합이 20이 넘는 경우 결과값을 반환하지만 20 미만인 경우 0을 반환하도록 조건을 추가하였습니다.  
* 여기서 주의 깊게 보셔야 할 부분은 음수의 포함여부와 총합이 20이 넘는지를 매번 검사하고 있다는 점입니다. 
* 조건에 따라 숫자들의 합을 구하는 로직만 변경될 뿐 나머지 코드는 중복이 발생하고 있습니다. 3의 배수의 합을 구하는 계산식이 추가되거나 다른 조건의 계산식이 추가되는 경우 동일하게 중복 코드가 발생하는 구조라는것을  알수 있어야 합니다. 
* 이러한 구조의 비즈니스 로직에서 일급 함수를 사용하여 중복 코드를 개선할 수 있는지 알아보겠습니다. 

```java
public class PositiveNumberCalculator {

    public int calculate(List<Integer> numbers, Function<List<Integer>, Integer> calculator) {
        boolean hasNegative = numbers.stream()
                                     .anyMatch(integer -> integer < 0);

        if (hasNegative) {
            throw new IllegalArgumentException("음수가 포함될 수 없습니다.");
        }

        Integer total = calculator.apply(numbers);

        if (total > 20) {
            return total;
        }

        return 0;
    }
}


public static void main(String[] args) {
        List<Integer> numbers = IntStream.rangeClosed(1, 10)
                                         .boxed()
                                         .collect(Collectors.toList());

        PositiveNumberCalculator calculator = new PositiveNumberCalculator();

        int sumAll = calculator.calculate(numbers, (integers) -> integers.stream()
                                                                         .reduce(0, (integer, integer2) -> integer + integer2));

        int sumEven = calculator.calculate(numbers, (integers) -> integers.stream()
                                                                          .filter(integer -> integer % 2 == 0)
                                                                          .reduce(0, (integer, integer2) -> integer + integer2));

        int sumOdd = calculator.calculate(numbers, (integers) -> integers.stream()
                                                                         .filter(integer -> integer % 2 == 1)
                                                                         .reduce(0, (integer, integer2) -> integer + integer2));
    }
```

* 한눈에 봐도 PositiveNumberCalculator 클래스의 코드가 확 줄어든것을 알수 있습니다. 중복되는 코드를 가진 메소드를 하나로 통합하고, 핵심이 되는 비즈니스 로직을 일급 함수의 특징인 매개변수로 전달 가능함을 이용하여 전달받도록 개선해보았습니다. 
* 개선된 구조에서 새로운 요구사항이 추가되는 경우 조건식이나 유효성검사와 같은 코드는 중복이 발생하지 않을것이고 중요한 비즈니스 로직만 구현하여 `전달`해주는것으로 기능을 추가할 수 있게 되었습니다. 
* 추가적으로 전달하고 있는 함수를 객체로 저장한다면 재사용성까지 가질수 있을것 같습니다. :) 
* 이와 비슷한 디자인 패턴이 있다는것을 눈치챈 분들도 계실것 같습니다. 생각하신것처럼 이러한 구조는 `템플릿 메소드 패턴`을 함수형 프로그래밍 패턴에 맞게 각색한것 입니다. 


> 템플릿 메소드 패턴(template method pattern)은 소프트웨어 공학에서 동작 상의 알고리즘의 프로그램 뼈대를 정의하는 행위 디자인 패턴이다. 알고리즘의 구조를 변경하지 않고 알고리즘의 특정 단계들을 다시 정의할 수 있게 해준다.


* 여기서 의미하는 뼈대란 예제에서 유효성 검사, 조건식을 뜻하게 되고 특정 단계가 함수로 전달한 계산식 비즈니스 로직이 됩니다. 

## 정리
* 일급 함수의 특징인 매개 변수로 함수를 전달할 수 있음을 이용하여 중복 코드를 제외한 비즈니스 로직을 메소드 내부에서 구현하는것이 아닌 메소드 외부에서 전달하는 방법으로 코드 리팩토링을 할 수 있었습니다. 
* 카카오 헤어샵 개발팀은 앞으로 기회가 될때마다 악취나는 코드 개선기를 공유해 드리려고 합니다. 다음 시리즈도 기대해주세요 :) 


# 참고
* [일급객체](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4)
* Modern Java In Action - 함수형 프로그래밍
* [템플릿 메소드 패턴](https://ko.wikipedia.org/wiki/%ED%85%9C%ED%94%8C%EB%A6%BF_%EB%A9%94%EC%86%8C%EB%93%9C_%ED%8C%A8%ED%84%B4)