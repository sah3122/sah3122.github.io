---
title: This 와 Super에 대한 오해
description: This 와 Super에 대한 오해
categories:
 - dev
tags:
 - Java
---
> 오브젝트(조영호님)책을 보며 정리한 내용, 작성중입니다.

Java에서 This 란 ?
 **this는 인스턴스의 자기 자신**을 의미한다. this.은 주로 **필드(전역변수)와 메소드 또는 생성자의 매개변수가 동일할 때 인스턴스 필드임을 명확히 하기 위해 사용**한다.

자기 자신을 의미한다 -> Self 참조값을 가진다. 

[image:691E34BC-B312-4128-986B-E438E22D6657-313-00013C90DB2EC097/C225693B-BF77-4EA2-8E73-BE2039800B75.png]

> Java에서 Super란 ?

super는 자식 클래스가 부모 클래스로부터 상속받은 멤버를 참조할 때 사용하는 참조 변수입니다. 클래스 내의 멤버변수와 지역변수의 이름이 같을 경우 구분을 위해 this를 사용하듯이 부모 클래스와 자식 클래스의 멤버의 이름이 같을 경우 super를 사용합니다. this와 super는 인스턴스의 주소값을 저장하는데 static 메서드(클래스 메서드)와는 무관하게 사용됩니다.

출처: https://freestrokes.tistory.com/72 [FREESTROKES DEVLOG]

상속받은 멤버를 참조할 때 사용하는 참조 변수 -> 참조값을 변경한다. 

[image:670EA140-164A-491D-BD46-2C3720574555-313-00013C953E7391ED/45933790-F795-4A95-BA5D-CD0536292A53.png]

간단한 예시 코드를 통하여 This와 Super에 대한 오해를 풀어봅시다. 

```java
public class Main {
    public static void main(String[] args) {
        Weapon weapon = new Knife();

        weapon.use();
    }
}

public class Weapon {
    private int damage = 0;

    public void use() {
        System.out.println("Use Weapon");
        attack();
    }

    public void attack() {
        System.out.println("Weapon : " + damage);
    }
}

public class Sword extends Weapon{
    @Override
    public void attack() {
        attack();
    }
}

public class Knife extends Sword {
    private int dagame = 4;

    @Override
    public void attack() {
        System.out.println("knife : " + dagame);
    }
}
```

