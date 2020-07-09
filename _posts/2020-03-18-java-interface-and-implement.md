---
title: 인터페이스와 구현에 대한 정리
description: 객체지향의 사실과 오해 내용 정리
categories:
 - dev
tags:
 - 객체지향의 사실과 오해
 - Java
---
> 객체 지향의 사실과 오해, 오브젝트 책을 보고 제가 느낀점들을 사내공유 하고자 만든 자료입니다.  

# 인터페이스와 구현
객체 지향의 사실과 오해, 오브젝트 책을 보고 제가 느낀점들을 사내공유 하고자 만든 자료입니다.  

객체란 자기 주도적으로 책임, 협력, 역할을 가지고 있는 상태를 의미한다.   

여기서 책임이란 어떠한 일을 맡아서 하는 것이다.  

커피 전문점 도메인을 가지고 인터페이스와 구현에 대해 알아 보자.  
커피 전문점 도메인은 고객, 바리스타, 커피, 메뉴판등의 객체를 가지고 있다고 가정한다.

고객이란 어떤 책임을 가지고 있을까 ?  
일반적으로 생각 하면 ‘주문’이라는 책임을 가지고 있을 것이다.  
모든 주문은 고객이 주도적으로 생성하게 될것이고 생성된 주문을 통하여 바리스타는 주문에 해당하는 커피를
고객에게 전달하게 된다. 따라서 고객은 주문이라는 책임을 가지고 있는 셈이다.

협력이란 ?   
고객이 ‘주문’이라는 책임을 가지고 다른 객체에게 `주문하라` 라는 메시지를 전달 한다.  
여기서 메시지는 객체간의 의사소통을 하기 위한 방법이다.   
여기서 고객은 주문하라 라는 메시지를 통해 바리스타에게 커피를 만들어라고 다시 한번 메시지를 전달하게 될것 이다.  
결론적으로 커피전문점에서 고객이 커피를 주문하기 위해선 고객과 바리스타간의 협력을 통하여 최종적으로 커피가 주문되게 되니 둘 간의 협력 관계가 형성 되는 것이다.

역할이란 ?   
역할은 메시지를 책임지는 집합으로 볼 수 있다.   
여러 커피전문점을 생각해보자.   
어떤 커피전문점에서는 들어오는 커피에 따라서 바리스타의 역할이 세분화 될 수 있다.  
예를 들면 에스프레소를 책임지는 바리스타, 라떼를 책임지는 바리스타 등으로 바리스타의 역할중 세분화 되어 있는 영역에서 똑같이 커피를 만들어라 라는 책임을 다하고 있는것이다.  
일반적으로 역할은 추상화 과정에서 나타난다.  
상위 수준에서의 책임을 쉽고 간단하게 표현하고 설계를 유연하게 만들수 있는 장점이 있다.

자율적인 객체  
객체는 상태와 행동을 함께 가지는 복합적인 존재이며 스스로 판단하고 행동하는 자율적인 존재라는 것이다.   
대부분의 프로그래밍 언어는 외부에서 접근을 통제 할 수 있는 접근 제어 메커니즘을 제공한다.   
접근 제어를 위해  public, private 와 같은 접근 수정자가 존재한다.  
객체 내부에 대한 접근을 통제하는 이유는 객체를 자율적인 존재로 만들기 위해서다.   
객체지향의 핵심은 스스로 상태를 관리하고 판단하고 행동하는 자율적인 객체들의 공동제를 구성 하는 것이다.  
캡슐화와 접근제어는 객체를 두 부분으로 나눌수 있다.   
외부에서 접근 가능한 퍼블릭 인터페이스(public interface), 내부에서만 접근 가능한 부분인 구현(implementation)이라고 부른다. 

캡슐화  
상태와 행동을 하나의 객체안에 모으는 이유는 객체의 내부 구현을 외부로부터 감추기 위함이다.  
잘 설계된 캡슐화를 사용하면 변경 가능성이 높은 부분은 내부에 숨기고, 외부에는 상대적으로 안정적인 부분만 공개함으로 변경의 여파를 통제 할 수 있다.  
변경될 가능성이 높은 부분을 구현이라 부르고 상대적으로 안정적인 부분을 인터페이스라고 부른다.  
일반적으로 사용하고 있는 getter / setter를 사용한 캡슐화는 진정한 의미의 캡슐화가 아닌것이 중요하다.

간단한 예시를 통하여 인터페이스와 구현에 대해서 알아 보자.

```java
public class Customer {

    public void order(String menuname, Menu menu, Barista barista) {
        MenuItem menuItem = menu.choose(menuname);
        Coffee coffee = barista.makeCoffee(menuItem);
    }
}
```
고객은 `order`라는 메시지를 통하여 `주문`이라는 책임을 실행한다.  
여기서 중요한 점은 order라는 메소드가 public으로 공개 되어 있다는것이다.  
이로서 다른 누군가는 Customer는 order라는 메시지를 전송할 수 있다라는것을 알 수 있다.  
내부 구현으로 Menu를 통하여 어떤 커피를 만들어야 하는지 받아 오고 바리스타에게 해당하는 커피를 만들라는 메시지를 전달한다.

```java
public class Menu {
    private List<MenuItem> menuItems;

    public Menu(List<MenuItem> menuItems) {
        this.menuItems = menuItems;
    }

    public MenuItem choose(String menuname) {
        return menuItems.stream()
                .filter(menuItem -> menuItem.name().equals(menuname))
                .findAny()
                .orElseThrow(IllegalArgumentException::new);
    }
}
```
Menu 객체를 살펴보자.  
여기서 중요하게 봐야할 부분은 choose가 아닌 menuItems를 내부에 상태로 가지고 있다는 것이다. 
Menu는 MenuItem에 의존하고 있는 상태가 된것이다.   
코드를 살펴보면 Menu를 생성할 때 MenuItems를 생성자로 받아 생성하고 있습니다.  
이렇게 구현한 이유는 MenuItem이 추가 되더라도 다른 객체에서는 추가적인 수정이 필요 없고 오직 MenuItem을 의존하고 있는 Menu 클래스만 수정하여 choose라는 책임을 다하면 되는것 이다. 

```java
public class MenuItem {
    private String menuname;
    private BigDecimal price;
    private MenuType menuType;

    public MenuItem(String menuname, BigDecimal price, MenuType menuType) {
        this.menuname = menuname;
        this.price = price;
        this.menuType = menuType;
    }

    public String name() {
        return menuname;
    }

    public BigDecimal cost() {
        return price;
    }

    public MenuType menuType () {
        return menuType;
    }
}
```

```java
public enum MenuType {
    ESPRESSO, COLDBREW, LATTE
}
```

```java
public class Barista {

    public Coffee makeCoffee(MenuItem menuItem) {
        return make(menuItem);
    }

    private Coffee make(MenuItem menuItem) {
        if(menuItem.menuType().equals(MenuType.ESPRESSO)) {
            //Do SomeThing
            return new Coffee(menuItem);
        } else if(menuItem.menuType().equals(MenuType.COLDBREW)) {
            //Do SomeThing
            return new Coffee(menuItem);
        } else if(menuItem.menuType().equals(MenuType.LATTE)) {
            //Do SomeThing
            return new Coffee(menuItem);
        }

        throw new IllegalArgumentException();
    }
}
```
이번 내용에서 가장 중요한 바리스타 클래스 입니다.   
바리스타 객체는 `커피를 만들어라`라는 책임을 가지고 있는 클래스 이며 코드를 보시면 makeCoffee라는 메소드가 public 으로 오픈 되어 있습니다.   
실제로 커피를 만드는 로직은 private으로 되어 있는 make라는 메소드에서 진행하고 있는데 이렇게 구현한 이유가 캡슐화를 하기 위해서 입니다.   
외부에서는 바리스타는 커피를 만들수 있다 라는것을 알고 있지만 커피를 어떻게, 어떤 방식으로 만드는지는 모르도록 내부에 구현 로직을 private 으로 감싸서 숨긴 예시라고 할 수 있습니다.
여기서 public 으로 오픈되어 있는 메소드는 인터페이스이며 private 으로 숨겨둔 메소드가 구현이라고 생각 하시면 될 것 같습니다.  
이러한 방식의 장점은 마찬가지로 커피를 만드는 방법이 추가 되더라도 커피는 바리스타 객체가 자율적으로 만들 수 있기 때문에 다른 클래스의 코드는 수정을 하지 않아도 된다는 것입니다. 

```java
public abstract class BaristaI {
    public Coffee makeCoffee(MenuItem menuItem) {
        //doSomeThing
        Coffee make = make(menuItem);
        //dosomeThing
        return make;
    }

    protected abstract Coffee make(MenuItem menuItem);

}
```

바리스타 클래스를 Template Method 패턴으로 구현하여 깔끔한 코드로 확인 할 수 있다.

참조
[Notion – The all-in-one workspace for your notes, tasks, wikis, and databases.](https://www.notion.so/3-68823e689c7c4b88891303261af1d333)
