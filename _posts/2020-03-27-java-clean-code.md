---
title: Java Clean Code 작성법
description: Clean Code를 작성하는 방법 공유
categories:
 - dev
tags:
 - DDD
 - Java
 - Clean Code
---
> 해당 포스트는 DDD 세레나데의 강의를 정리하며 작성하였습니다.

# Java Code 작성 팁

* 불변객체(Value Object)를 사용하자.
	* 불변객체란 ?  
	위키 백과 - 컴퓨터 과학에서 가치 객체는 동일성을 기반으로하지 않는 단순 개체를 나타내는 작은 객체입니다.   
	즉, 두 객체가 동일한 값을 가질 때 동일하며 반드시 동일한 객체 일 필요는 없습니다. 가치 개체의 예는 금액 또는 날짜 범위를 나타내는 개체입니다.
	* 동일한 값을 가질때 동일한 객체임을 보장해주며 객체의 상태를 변경할 될 수 없는 객체이다. 
	* 의미를 명확하게 표현하거나 두 개 이상의 데이터가 개념적으로 하나인 경우 밸류 타입을 이용
	* 시스템이 성숙함에 따라 데이터 값을 객체로 대체
	* 밸류 객체의 값을 변경하는 방법은 새로운 밸류 객체를 할당하는 것뿐이다.
	* 정말 String으로 우편 번호를 표현할 수 있는가?
	* 항상 equals() 메서드를 오버라이드할 것을 권고한다.
> equals를 재정의하려거든 hashCode도 재정의하라 - Effective Java


* 일급 컬렉션을 사용하자. 
	* 참고 [일급 컬렉션 (First Class Collection)의 소개와 써야할 이유](https://jojoldu.tistory.com/412)
	* 일급 컬렉션 이란 ? 
> 콜렉션을 포함한 클래스는 반드시 다른 멤버 변수가 없어야 한다.
각 콜렉션은 그 자체로 포장돼 있으므로 이제 콜렉션과 관련된 동작은 근거지가 마련된셈이다.
필터가 이 새 클래스의 일부가 됨을 알 수 있다.
필터는 또한 스스로 함수 객체가 될 수 있다.
또한 새 클래스는 두 그룹을 같이 묶는다든가 그룹의 각 원소에 규칙을 적용하는 등의 동작을 처리할 수 있다.

```java
public class LottoNumbers {
	  private static final Integer LOTTO_NUMBER_SIZE = 6;
    private final List<Integer> numbers;

    public LottoNumbers(List<Integer> numbers) {
        validNumber(numbers);
        this.numbers = numbers;
    }

    private void validNumber(List<Integer> numbers) {
        if (Objects.isNull(numbers) || number.size() != LOTTO_NUMBER_SIZE) {
            throw new IllegalArgumentException();
        }
    }

    public Integer totalValue() {
        return numbers.stream()
                .mapToInt(Integer::intValue)
                .sum();
    }
}
```
* 항상 방어적 복사를 생각하자.
	* Java 에서는 값을 참조하는 방법이 크게 두가지가 존재
		* Call by Reference
		* Call by Value
	* 일반적으로 객체간의 값 복사는 Call by Reference 방식으로 통하여 값이 복사 된다.
	이 방식으로 값을 복사 하면 위에서 말한 일급 컬렉션이 불변하지 않게 되는 문제점이 있다. 그래서 항상 방어적 복사(DeepCopy)를 생각하는 방식	을 권장한다.
  
```java
List<Integer> lottoNumbers = new ArrayList<>();
lottoNumbers.add(4);
lottoNumbers.add(5);
lottoNumbers.add(6);
lottoNumbers.add(8);
lottoNumbers.add(13);
lottoNumbers.add(45);

Numbers numbers = new Numbers(lottoNumbers);
System.out.println(numbers.totalValue());
// 81

lottoNumbers.add(2);
System.out.println(numbers.totalValue());
// 83
```

```java
public class Numbers {
    private static final Integer LOTTO_NUMBER_SIZE = 6;
    private final List<Integer> numbers;

    public Numbers(List<Integer> numbers) {
        validNumber(numbers);
        this.numbers = new ArrayList<>(numbers);
        this.numbers = Collections.unmodifiableList(numbers);
    }

    private void validNumber(List<Integer> numbers) {
        if (Objects.isNull(numbers) || numbers.size() == LOTTO_NUMBER_SIZE) {
            throw new IllegalArgumentException();
        }
    }

    public Integer totalValue() {
        return numbers.stream()
                .mapToInt(Integer::intValue)
                .sum();
    }
}
```
* setter 보다 의미있는 이름의 메소드를 사용해보자.
	* 객체에 getter 메서드와 setter 메서드를 무조건 추가하는 것은 좋지 않은 버릇
	* 특히 setter 메서드는 객체의 핵심 개념이나 의도를 코드에서 사라지게 한다.
	* setter 메서드의 또 다른 문제는 객체를 생성할 때 완전한 상태가 아닐 수도 있다는 것이다.
	* 도메인 객체가 불완전한 상태로 사용되는 것을 막으려면 생성 시점에 필요한 것을 전달해 주어야 한다.

```java
changeShippingInfo() vs setShippingInfo()
completePayment() vs setOrderState()
```
* 주/부 생성자를 활용해보자.
	* 주 생성자 
		* 모든 상태를 가진 완전한 객체를 생성하는 생성자
	* 부 생성자
		* 상태가 완전하지 않은 객체를 생성하는 생성자 
	* 객체를 생성할 때 상태가 완전하지 않은 객체를 생성하는 경우가 있다.
	이런 객체들을 생성할 때 마다 객체의 상태를 검사하는 로직을 남발하는 경우가 존재한다. 
	해당 로직들을 주 생성자에 위임하여 항상 동일한 로직을 타도록 설정 할 수 있다.

```java
public class Menu {
    private String name;
    private BigDecimal price;
    private Long menuGroupId;
    private List<MenuProduct> menuProducts;

    public Menu(String name, BigDecimal price, List<MenuProduct> menuProducts) {
        this(name, price, null, menuProducts);
    }

    public Menu(String name, BigDecimal price, Long menuGroupId, List<MenuProduct> menuProducts) {
        validName(name);
        validPrice(price);
        validMenuProducts(menuProducts);

        this.name = name;
        this.price = price;
        this.menuGroupId = menuGroupId;
        this.menuProducts = menuProducts;
    }

    private void validMenuProducts(List<MenuProduct> menuProducts) {
        if (Objects.isNull(menuProducts) || menuProducts.size() == 0) {
            throw new IllegalArgumentException();
        }
    }

    private void validPrice(BigDecimal price) {
        if (Objects.isNull(price) || price.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException();
        }
    }

    private void validName(String name) {
        if(Objects.isNull(name) || StringUtils.isEmptyOrWhitespace(name)) {
            throw new IllegalArgumentException();
        }
    }
}
```
* 생성자 보다는 정적 팩토리 메소드를 사용하자.
	* [정적 팩토리 메서드(static factory method) - 기계인간 John Grib](https://johngrib.github.io/wiki/static-factory-method-pattern/)
* Package By Feature, Not Layer
	* 패키지를 생성하는 전략 크게 두가지로 나뉘어 생성이 가능하다.
		1. Package by Layer
			* Spring에서 일반적으로 사용하는 Controller / Service / Repository 등의 패키지를 최상단 패키지로 설정하여 프로젝트를 구성하는 방식
		2. Package by Feature
			* 도메인 중심으로 패키지를 생성하는 방식.
				* 각 도메인이 최 상단 패키지가 되어 하위에 api / application / infrastructure / domain 등을 가지게 된다.
				* 정답은 없다.
	* 프로젝트의 규모가 커질수록 Package By Layer 의 구조는 복잡성이 증가하게 된다. 
	* 반면 Feature중심의 패키지 구조로 되어 있으면 연관성이 있는 패키지들이 새롭게 추가 되기 때문에 기존 패키지에 영향을 미치지 않게 되며
	필요한 기능들이 군집해 있기 때문에 다른 패키지들간의 영향을 줄일수 있게 된다. 

	* 참고 자료를 보시는것이 더욱 좋습니다 : ) 
	* http://www.javapractices.com/topic/TopicAction.do?Id=205
	* https://medium.com/@ssowonny/package-by-feature-in-clean-architecture-projects-e14d25e3905e

#JAVA/Code