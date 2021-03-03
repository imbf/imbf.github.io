---
layout: post
title: "[실무 면접 준비 - 4] 객체지향 & JVM (OOP & JVM)"
date: 2021-03-02
author: Green Frog Developer
categories: Interview
tags: OOP SOLID Design-Pattern JVM Java Runtime-Storage-Area
---

---

## Object Oriented Programming(OOP)

---

### 좋은 코드란 무엇인가요?

코드란 특정 문제를 해결하기 위해 작성하는 것이라고 생각합니다. 이러한 문제들은 요구사항으로 표현할 수 있기 때문에 **좋은 코드란 현재의 요구사항을 만족하면서 불확실한 미래의 요구사항까지 유연하게 대처할 수 있는 코드**를 좋은 코드라고 생각합니다.



### 좋은 코드를 작성하기 위해서는 무엇이 가장 중요하나요?

좋은 코드를 작성하기 위해서는 **도메인에 대한 깊은 이해**가 가장 중요하다고 생각합니다. 도메인에 대한 깊은 이해 없이 작성된 코드는 일단 현재의 요구사항은 만족할지 몰라도 **도메인의 특성들이 코드에 잘 녹여져 있지 않기 때문에 조그마한 요구사항에도 코드의 큰 변화가 필요하게 됩니다.** 이러한 큰 변화들은 자연스레 버그를 유발하게 되구요 결국 좋지 못한 코드가 됩니다.

> 비즈니스 요구사항은 도메인으로부터 발생합니다. 만약 코드가 도메인의 특성이 잘 반영하지 않는다면 도메인으로부터 발생한 조그마한 요구사항에도 코드는 큰 변경을 요구할 것이며 이는 자연스레 많은 버그를 유발하게 됩니다. 이러한 버그들은 개발자들에게 부담으로 다가와서 변경에 소극적이게 되고 자연스레 기획자의 다양한 요구사항들을 받아들이지 못하게 됩니다. 
>
> 서비스를 운영하면서 수 많은 요구사항이 발생하는데 기획자가 이를 파악해서 개발자에게 요구사항을 전달함에도 불구하고 개발자가 이러한 요구사항을 시스템에 반영하지 못하는 코드 구조라... **너무 슬프다고 생각합니다. 저는 이러한 시스템을 구성하는 개발자는 되고 싶지 않습니다.**



### 객체 지향 프로그래밍이란 무엇인가요?

**객체 지향 프로그래밍이란 데이터와 프로시저를 각각 따로 관리하는 것이 아닌 객체에 위치시켜서 객체 간의 협력을 통해서 시스템을 구성하는 프로그래밍 방식입니다.**



### 절차 지향 프로그래밍이란 무엇인가요?

**절차지향 프로그래밍이란 순차적인 처리를 중요시하여 시스템을 구성하는 프로그래밍 방식입니다.**  



### 객체 지향과 절차 지향 프로그래밍의 장단점은 무엇이고 언제 사용해야 할까요?

**객체 지향 프로그래밍의 장점으로는 코드의 재사용성이 높아지고 유지보수가 쉬워진다는 장점이 있으며 단점으로는 객체 생성 및 참조 시에 발생하는 오버헤드 때문에 절차지향에 비해 상대적으로 느리다는 단점이 존재합니다.** 

**절차 지향 프로그래밍의 장점으로는 불필요한 오버헤드를 줄여 속도가 빠르다는 장점이 있으며 단점으로는 코드의 재사용성과 유지보수성이 객체지향 비해 떨어진다는 단점이 존재합니다.**

객체 지향의 경우 요구사항이 자주 발생하거나 변경되는 Web이나 Mobile App 같은 서비스 개발하면서 객체 지향을 사용하면 좋을 것 같구요, 절차 지향의 경우 요구사항 변경보다는 성능에 중점을 두고 싶은 일괄 처리와 같은 시스템에서 절차 지향을 사용하면 좋을것 같습니다.

> **오버헤드(overhead)**
>
> 오버헤드란 어떤 처리를 하기 위해 들어가는 <u>간접적인 처리 시간, 메모리</u> 등을 말합니다.



### 객체 지향의 3대 특징에 대해서 설명해주세요.

**캡슐화란 객체의 속성과 행위를 하나로 묶어서 세부 구현은 내부로 감추고 최소한의 인터페이스만 객체 외부로 노출시키는 것을 의미합니다.** 이를 통해 자율적인 객체를 만들 수 있으며 다른 객체와 불필요하게 결합될 필요가 없어서 자연스레 시스템의 결합도는 떨어지고 응집도는 올라가게 됩니다. 이의 결과로서 시스템은 유지보수하기 쉬워지고 추후 변경에 유연하게 대처할 수 있게되는 효과를 가져옵니다.

**상속이란 자식이 부모의 특징을 물려받는 것을 의미하구요, 자바에서는 인터페이스 상속과 클래스 상속으로 나뉩니다.** 이를 통해 모듈 즉, 클래스의 재사용성이 높아지고 다형성을 구현할 수 있게 되어집니다. 하지만 무분별한 상속 자체는 모듈간의 결합도를 올리기 때문에 무분별하게 남용할 시 추후 변경에 유연하게 대응하지 못하는 시스템 구조를 가질 수 있습니다.

**다형성이란 같은 요청으로부터 응답이 객체의 타입에 따라 다르게 나타나는 것을 의미합니다.** 이를 통해 객체지향은 더 유연한 설계를 가질 수 있습니다.



### 객체(Object)와 클래스(Class)의 차이는 무엇인가요?

**클래스는 시스템의 정적인(static) 구성요소를 나타내는 것이구요, 객체는 클래스의 인스턴스로써 시스템의 동적인(dynamic) 구성 요소를 나타냅니다.**



### 추상화(Abstraction)란 무엇인가요?

**추상화란 특정 개념이나 개체를 보았을 때 특정 관점에서 관심있거나 중요한 부분을 강조하거나 추려내는 작업을 의미합니다.**  즉, 중요한 정보만 객체의 인터페이스로써 표시하고 세부 구현 정보는 객체의 내부로 숨겨서 객체 지향 프로그래밍을 하기 위한 매우 중요한 개념입니다.




### 객체 지향 설계 5원칙은 각각 무엇이며 우리는 이를 통해 무엇을 얻고 싶은 걸까요?

**객체 지향 설계 5 원칙은 원칙들의 앞글자만 따서 SOLID 원칙이라고도 불리는데요, 시스템의 결합도를 낮추고 응집도를 높이는 고전적인 시스템 설계 원칙을 객체 지향 관점에서 5가지의 원칙으로 재정립한 것입니다.**

- **단일 책임 원칙(Single Reponsibility Principle, SRP) :** 한 클래스는 하나의 책임만 가지며 클래스는 그 책임을 완전히 캡슐화해야 한다는 원칙입니다. 
- **개방 폐쇄 원칙(Open Closed Principle, OCP) :** 소프트웨어 요소(메서드, 클래스, 모듈)는 확장에 대해서는 열려 있어야 하고 수정(변경)에 대해서는 닫혀 있어야 한다는 원칙입니다.
- **리스코프 치환 원칙(Liskov Substitution Principle, LSP) :** 서브 클래스의 인스턴스는 슈퍼 클래스의 인스턴스로 치환할 수 있어야 한다는 원칙입니다.
- **인터페이스 분리 원칙(Interface Separation Principle, ISP) :** 클라이언트는 자신이 이용하지 않는 메서드에 의존하지 않아야 한다는 원칙입니다.
  - 큰 덩어리의 인터페이스들을 구체적이고 작은 단위들로 분리 시킴으로써 시스템의 결합도를 낮추고 응집도를 높인다.
- **의존 역전 원칙(Dependency Inversion Principle, DIP) :** 특정 소프트웨어 요소를 개발하면서 자신보다 변하기 쉬운것에 의존하지 말라는 원칙입니다. 즉, 보다 추상적인 것에 의존하라는 원칙입니다.



### 디자인 패턴이란 무엇인가요?

**소프트웨어를 설계할 때 특정 문맥에서 자주 발생하는 문제들을 해결할 수 있는 재사용 가능한 Best Practice 설계 패턴을 의미합니다.**



### Singleton 패턴은 무엇이고 언제 사용되며 어떻게 구현하나요?

**Singleton 패턴이란 시스템에서 인스턴스가 오직 1개만 생성되어야 하는 경우에 사용되는 패턴을 의미합니다.** 시스템에서 동일한 커넥션 객체를 만든다던지, 하나만 사용되어야하는 객체를 만들 때 매우 유용합니다.

- **Eager Initialization**

  ```java
  import org.junit.jupiter.api.Test;
  
  import static org.junit.jupiter.api.Assertions.assertEquals;
  
  public class Singleton {
      private static Singleton singleton = new Singleton();
      
      private Singleton() {
      }
      
      public static Singleton getInstance() {
          return singleton;
      }
      
      @Test
      public void singletonTest() {
          Singleton singletonA = Singleton.getInstance();
          Singleton singletonB = Singleton.getInstance();
          assertEquals(singletonA, singletonB);
      }
  }
  ```

- **Lazy Initialization**

  ```java
  import org.junit.jupiter.api.Test;
  
  import static org.junit.jupiter.api.Assertions.assertEquals;
  
  public class Singleton {
      private static Singleton singleton;
      
      private Singleton() {
      }
      
      public static synchronized Singleton getInstance() {
          if (singleton == null) {
              singleton = new Singleton();
          }
          return singleton;
      }
      
      @Test
      public void singletonTest() {
          Singleton singletonA = Singleton.getInstance();
          Singleton singletonB = Singleton.getInstance();
          assertEquals(singletonA, singletonB);
      }
  }
  ```



### Factory Method 패턴은 무엇이고 언제 사용되며 어떻게 구현하나요?

**부모 클래스로부터 객체를 생성하는 부분을 서브 클래스에 위임하여 다형성을 통해 클래스간의 결합도를 낮추기 위한 디자인 패턴을 의미합니다.** 팩토리 메서드 패턴을 사용함으로써 객체 생성시 발생하는 요구사항들에 유연하게 대처할 수 있습니다. 

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class FactoryMethod {
    
    @Test
    public void factoryMethodTest() {
        Factory factoryA = new StringFactory();
        Factory factoryB = new EnumFactory();
        
        assertEquals(factoryA.create("APPLE").getClass().getName(),
                factoryB.create("APPLE").getClass().getName());
    
    
        assertEquals(factoryA.create("GRAPE").getClass().getName(),
                factoryB.create("GRAPE").getClass().getName());
    }
}

abstract class Factory {
    abstract Fruit create(String name);
}

class StringFactory extends Factory {
    @Override
    Fruit create(String name) {
        Fruit result = null;
        switch (name) {
            case "APPLE":
                result = new Apple();
                break;
            case "GRAPE":
                result = new Grape();
                break;
            case "MELON":
                result = new Melon();
                break;
        }
        return result;
    }
}

class EnumFactory extends Factory {
    @Override
    Fruit create(String name) {
        Fruit result = null;
        FruitCategory fruitCategory = FruitCategory.valueOf(name);
        if (fruitCategory.equals(FruitCategory.APPLE))
            result = new Apple();
        
        if (fruitCategory.equals(FruitCategory.GRAPE))
            result = new Grape();
        
        if (fruitCategory.equals(FruitCategory.MELON))
            result = new Melon();
        
        return result;
    }
}

enum FruitCategory {
    APPLE, GRAPE, MELON
}


class Fruit {

}

class Apple extends Fruit {

}

class Grape extends Fruit {

}

class Melon extends Fruit {

}
```



### Adapater 패턴은 무엇이고 언제 사용되며 어떻게 구현하나요?

**어댑터 패턴(Adapter pattern)은 클래스의 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로 서로 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들이 작동하도록 해줍니다.** 대표적으로 JDBC가 어댑터 패턴을 활용하여 시스템을 단일한 인터페이스로 조작할 수 있게끔 해줍니다.

```java
class ServiceA {
    void runServiceA() {
        System.out.println("ServiceA");
    }
}

class ServiceB {
    void runServiceB() {
        System.out.println("ServiceB");
    }
}

interface AdapterService {
    void runService();
}

class AdapterServiceA implements AdapterService{
    ServiceA sa1 = new ServiceA();
    
    public void runService() {
        sa1.runServiceA();
    }
}

class AdapterServiceB implements AdapterService{
    ServiceB sb1 = new ServiceB();
    
    public void runService() {
        sb1.runServiceB();
    }
}

public class ClientWithAdapter {
    public static void main(String[] args) {
        AdapterService asa1 = new AdapterServiceA();
        AdapterService asb1 = new AdapterServiceB();
        
        asa1.runService();
        asb1.runService();
    }
}
```



### Template Method 패턴은 무엇이고 언제 사용되며 어떻게 구현하나요?

**템플릿 메서드 패턴(Template Method Pattern)이란 알고리즘의 구조를 바꾸지 않고 서브클래스에서 알고리즘의 특정 단계를 재정의하는 것을 의미합니다.** 즉, 상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 가장 대표적인 방법이다.

```java
import org.junit.jupiter.api.Test;

abstract class Business {
    public void print() {
        System.out.println("======================");
        printBusinessName();
        System.out.println("======================");
    }
    
    abstract void printBusinessName();
}

class Cafe extends Business {
    @Override
    void printBusinessName() {
        System.out.println("카페");
    }
}

class HairShop extends Business {
    @Override
    void printBusinessName() {
        System.out.println("헤어샵");
    }
}

public class TemplateMethod {
    @Test
    public void test() {
        Cafe cafe = new Cafe();
        HairShop hairShop = new HairShop();
        cafe.print();
        hairShop.print();
    }
}

```



### Strategy 패턴은 무엇이고 언제 사용되며 어떻게 구현하나요?

**전략 패턴(Strategy Pattern)이란 전략 객체를 통해서 알고리즘의 구조를 바꿀 수 있는 디자인 패턴을 의미합니다.** 전략 객체를 위한 인터페이스를 정의하고 알고리즘의 구조를 바꾸고 싶은 경우 직접 로직을 수정하지 않고 전략 객체를 바꿔줌으로써 시스템의 결합도를 낮출 수 있습니다.

> Java Util의 PriorityQueue가 전략 패턴을 사용하는 대표적인 예이며 Comparator 인터페이스가 전략 객체로써 사용됩니다.

```java
import org.junit.jupiter.api.Test;

public class StrategyTest {
    @Test
    public void strategyTest() {
        Strategy strategy = null;
        Solider rambo = new Solider();
    
        strategy = new StrategyGun();
        rambo.runContext(strategy);
        
        System.out.println();
    
        strategy = new StrategySword();
        rambo.runContext(strategy);
    }
}

interface Strategy {
    void runStrategy();
}

class StrategyGun implements Strategy {
    @Override
    public void runStrategy() {
        System.out.println("탕, 타당, 타다당");
    }
}

class StrategySword implements Strategy {
    @Override
    public void runStrategy() {
        System.out.println("챙.. 채쟁챙 챙챙");
    }
}

class Solider {
    void runContext(Strategy strategy) {
        System.out.println("전투 시작");
        strategy.runStrategy();
        System.out.println("전투 종료");
    }
}
```



### Proxy 패턴과 Decorator패턴은 각각 무엇이고 언제 사용되며 어떻게 구현하나요?

**Proxy와 Decorator 패턴 모두 Target Object와 동일한 인터페이스를 가진 객체를 이용해서 다양한 행위를 추가하기 위한 의도가 존재합니다.**



**Decorator 패턴은 기존 타겟 객체의 기능에 책임을 추가시키는 것을 의미합니다.** 예로 들어 어떤 메소드의 시작이나 끝에 <u>로깅이나 트랜잭션을 추가</u>해주는 것을 Decorator 패턴이 적용된 예시라고 할 수 있습니다.

```java
package com.webhook.receiver.slack.test;

import org.junit.jupiter.api.Test;

interface Develop {
    void run();
}

class Plan implements Develop {
    @Override
    public void run() {
        System.out.println("기획");
    }
}


class Design implements Develop {
    private Develop develop;
    
    public Design(Develop develop) {
        this.develop = develop;
    }
    
    @Override
    public void run() {
        develop.run();
        System.out.println("설계");
    }
}

class Implement implements Develop {
    private Develop develop;
    
    public Implement(Develop develop) {
        this.develop = develop;
    }
    
    @Override
    public void run() {
        develop.run();
        System.out.println("개발");
    }
}

public class Decorator {

    @Test
    public void decoratorTest() {
        Develop plan = new Plan();
        Develop design = new Design(plan);
        Develop implement = new Implement(design);
        
        implement.run();
    }
}
```



**Proxy 패턴은 기존 타겟 객체의 기능에 특정 책임을 추가시키는 것이 아니라, 이러한 기능들을 다양한 방식으로 제어 해주는 즉, 접근에 대한 제어가 목적인 패턴을 의미합니다.** 예로 들어 <u>메소드의 실행을 remote host로 보낸다든지, lazy Initialization 한다든지, 캐쉬된 데이터를 사용하는 등의 작업</u>들을 Proxy 패턴이 적용된 예시라고 할 수 있습니다.

```java
import org.junit.jupiter.api.Test;

interface Coffee {
    String getName();
}

class Americano implements Coffee {
    @Override
    public String getName() {
        return "아메리카노 맛있어~";
    }
}

class CoffeeProxy implements Coffee {
    private Coffee coffee;
    
    public CoffeeProxy(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getName() {
        // 호출에 대한 흐름 제어가 주목적!!!
        return coffee.getName();
    }
}

public class Proxy {
    @Test
    public void proxyTest() {
        Coffee proxy = new CoffeeProxy(new Americano());
        System.out.println(proxy.getName());
    }
}
```



### 연관(Association) vs 합성(Composition) vs 집합(Aggregation)의 차이는 무엇인가요?

**두 객체 간의 관계가 존재할 때 이를 연관(Association)이라고 부릅니다.**  연관(Association)은 합성(Composition)과 집합(Aggregation)으로 나눌 수 있습니다.

**Engine 객체가 Car 객체의 일부분 일 때 이 둘의 관계는 Composition 이라고 합니다.** 즉, Car 객체가 죽으면 Engine 객체도 죽습니다. (Engine 객체가 Car 객체 내부에서 생성) **(Part of 관계)**

```java
public class Car {
    private final Engine engine;  
       
    public Car(){
       engine  = new Engine();
    }
}

class Engine {
    private String type;
}
```

**Organization 객체가 Person 객체들을 가지고 있을 때 이 둘의 관계는 Aggregation 이라고 합니다.** 즉, Organization 객체가 죽어도 Person 객체는 죽지 않습니다. (Person 객체가 Organization 객체 외부에서 독립적으로 생성) **(has A 관계)**

```java
public class Organization {
    private List<Person> employees;
}

public class Person {
    private String name;   
}
```



---

### Java Virtual Machine (JVM)

---

### JVM 아키텍처

<img src="/assets/interview/naver-practical-interview-preparation4-1.png" style="width:100%">



### java와 class파일의 차이는 무엇이고 어떻게 동작하나요?

**응용 프로그래머는 자바 파일(.java)을 작성하고 컴파일러가 이를 바이트 코드 파일(.class)로 변경시켜 Class Loader에 의해 JVM 메모리에 로드됩니다.** 즉, java 파일은 프로그래머가 작성하는 파일이고, class 파일은 컴파일러에 의해서 java 파일이 바이트 코드로 변경된 파일입니다. JVM은 클래스 로더를 사용해서 클래스 파일을 JVM 메모리 즉, Runtime Data Area에 올려놓습니다.



### 자바는 어떠한 Linking 알고리즘을 사용하고 이러한 알고리즘이 가능한 이유는 무엇인가요?

**Linking이란 여러 개의 코드와 데이터를 모아서 연결하는 작업을 의미합니다.** Linking을 통해서 생성된 실행 가능한 파일은 Loader를 통해 메모리에 배치됩니다.

**자바는 동적 링킹(Dynamic Linking)을 사용하여 실행 가능한 파일을 만들 때 프로그램에서 사용하는 모든 라이브러리 모듈을 복사하지 않고, 해당 모듈의 주소만 가지고 있다가 런타임에 실행 파일과 라이브러리가 메모리에 위치될 때 해당 모듈의 주소로 가서 필요한 것을 들고 오는 방식을 사용합니다.** (가상 메모리 방식과 비슷한 감도 없지 않아 있다.)

**자바가 동적 링킹(Dynamic Linking)을 사용할 수 있는 이유는 .class 파일이 실행 가능한 형태가 아닌 JVM이 읽을 수 있는 형태 Java Byte Code 이기 때문입니다.** <u>class 파일은 JVM위에서 Linking 작업을 수행할 수 있도록, 라이브러리에 대한 Symbolic Reference만을 가지고 있게 됩니다.</u>



### Runtime Dynamic Loading이란 무엇인가요?

**Runtime Dynamic Loading이란 소스코드에서 객체를 참조하는 순간에 동적으로 Loading 하는 방식입니다.** 이는 Reflection 이라는 기술의 기본입니다.



### Class Loader란 무엇인가요?

**Class Loader는 컴파일 타임에 생성된 바이트코드(.class)를 런타임에 Load하고 배치하는 일을 담당합니다.** 클래스가 참조되는 순간 동적으로 Load 및 Link가 이루어지는 Dynamic Loading을 담당하는 주체입니다. 



### Runtime Data Area 란 무엇인가요?

**Runtime Data Area란 JVM이 프로그램을 수행하기 위해 OS로부터 할당 받는 메모리 영역을 의미합니다.** Runtime Data Area는 각각의 목적에 따라 5개의 영역으로 나뉘며 이는 PC Register, JVM Stacks, Heap, Method Area, Native Method Stacks 입니다.



### PC Register는 무엇인가요?

**PC Registers는 Thread가 생성될 때 마다 생기는 공간으로 Thread가 어떠한 명령을 실행하게 될지에 대한 부분을 기록하는 공간입니다.**



### JVM Stacks는 무엇인가요?

JVM Stack은 스레드마다 할당되며 스레드와 동시에 생성되고 소멸됩니다. **JVM 스택은 Frame을 넣고 빼는 작업을 수행하는 곳으로 지역 변수와 부분적인 결과를 보유하고 메소드 호출과 반환에서 역할을 수행합니다.**



### JVM Stack에 존재하는 Frame은 무엇인가요?

**Frame은 데이터와 부분적인 결과를 저장하고, 동적인 linking을 수행하고, 메소드를 위해 값을 리턴하고, 예외를 dispatch 하기 위해 사용되어집니다.** 프레임은 메서드를 호출하는 스레드의 JVM Stack으로 부터 할당되어지고 메서드가 호출되어질 때 생성되고 메서드가 완료되어질 때 소멸합니다.

**Frame은 Local Variable Section, Operand Stack, Frame Data 총 3부분으로 구성되어 있습니다.**

- **Local Variable Section**

  <u>Method의 Parameters와 Local Variables을 저장</u>합니다. Primitive Type의 경우 Local Variable Section에 저장되지만 Reference Type은 Local Variable Section에 저장되지 않고 heap에 저장한 후 저장된 위치의 Reference를 Frame의 Local Variable Section에 저장합니다.

- **Operand Stack**

  Frame의 Operand Stack이란 스레드의 작업공간이라고 할 수 있다. 프로그램 연산을 수행하면서 필요한 데이터 및 결과를 저장하는 곳이다.

- **Frame Data**

  Constant Pool Resolution 정보와 Method가 정상 종료되었을 때의 정보, 비정상 종료했을 때 Exception 관련 정보가 포함되어 있습니다.

  > Constant Pool Resolution이란 Constant Pool의 Pointer 정보를 의미합니다.
  >
  > Constant Pool은 Method Area에 있는 곳으로 **Class의 모든 Symbolic Reference가 저장**되어 있습니다.

### Native Method Stack은 무엇인가요?

**Native Method Stack이란 Native Method 즉, 자바 프로그래밍 언어 이외의 언어로 작성된 메서드를 지원하기 위한 스택이며 이를 호출할 시 JVM Stack은 동작하지 않고, Native Method Stack을 활용하여 Frame을 push, pop 한다.**



### Method Area란 무엇인가요?

Method Area란 모든 JVM 스레드 사이에서 공유되어지는 영역으로 JVM이 시작할 때 생성되어지며 종료될 때 소멸합니다. **Method Area는 run-time constant pool, 필드, 메소드, 생성자 등의 클래스별 구조를 저장합니다.** 이러한 정보들은 ClassLoader에서 넘겨 받은 Class File의 관련 정보를 추출하여 저장합니다.



### Run-Time Constant Pool이란 무엇인가요?

**run-time constant pool은 클래스 파일에 있는 constant_pool table에 대한 클래스별 또는 인터페이스별 runtime 표현입니다.** 클래스와 인터페이스의 모든 Constant 정보를 가지고 있는 곳이며 여기서의 <u>Constant는 상수만을 의미하는 것이 아니라 Literal Constant, Type Field(Local Variable, Class Variable), Method로의 모든 Symbolic Reference</u> 까지 확장된 개념을 의미한다.

> ### Symbolic Reference
>
> 자바에서는 특정 객체를 참조할 때 Memory Address를 직접 참조하는게 아니라, **객체의 이름으로 참조**합니다. 이렇게 **객체의 이름으로 참조하는 것은 Symbolic Reference**라고 합니다. 
>
> Symbolic Reference는 Constant Pool에 저장되며 객체에 접근할 필요가 있으면 Constant Pool에서 Symbolic Reference를 통해 해당 객체의 Memory Address를 찾아 동적으로 연결합니다.



### Heap이란 무엇인가요?

Heap은 JVM의 모든 스레드 사이에서 공유되어지는 영역입니다. Heap은 JVM이 시작할 때 생성되고 종료될 때 소멸됩니다. **Heap은 모든 클래스의 인스턴스와 배열에 대한 메모리가 할당되는 영역입니다.** 자바에서 객체들은 절대 명시적으로 할당이 해제되지않으며 Garbage Collector에 의해서만 메모리가 회수됩니다.



### JVM Heap 구조

<img src="/assets/interview/naver-practical-interview-preparation4-2.png" style="width:100%">

- **Young Generation :** Object가 최초로 할당 되는 장소이며 Eden이 꽉 차게 되면 Eden 의 모든 객체들의 참조 여부를 따져서 참조가 되어있는 Object이면 Survivor영역으로 옮기고, 참조가 없는 Object라면, 그냥 남겨 놓는다. 이후에 Eden 영역을 모두 청소해버린 다. 
- **Old Generation :** Young Generation영역에서 Live Object로 오랫동안 살아남아 성숙한 Object는 Old Generation영역으로 이동하게 된다. 이를 **Promotion**이라고 한다. 비교 적 오랫동안 참조가되어 앞으로도 계속 이용될 확률이 높은 Object를 저장하는 공간이다.



### Execute Engine

Execute Engine은 JVM의 다양한 메모리 영역과 통신합니다.  **Execute Engine은 클래스 로더를 통해서 JVM의 run-time data area에 할당되어진 바이트 코드를 실행하는 역할을 담당합니다.** Execute Engine은 Java byte code를 실행하기 위해 3가지 주요 요소인 Interpreter, JIT Compiler, Garbage Collector을 포함한다.

- **Interpreter :** Interpreter는 Java Byte Code를 읽고 OS의 native code로 변화하여 이들을 순차적인 방식으로 실행하는 구성요소를 의미합니다.
- **JIT Compiler :** JIT Compiler는 컴파일러를 통해 interpreter의 느린 실행의 단점을 상쇄하고 성능을 향상시킵니다.
- **Garbage Collector :** 메모리를 자동으로 관리하는 도구를 의미하며 백그라운드에서 항상 실행하고있는 daemon 스레드 입니다.

**Java Native Interface(JNI) :** JNI는 자바 메소드 호출과 관련된 native libraries 와의 다리 역할을 해줍니다.



### Garbage Collection

**Garbage Collection이란 Heap을 재활용하기 위해 Root Set에서 참조되지 않는 Object(Unreachable Object)를 없애 가용한 공간을 만드는 작업이라고 할 수 있습니다.**

Garbage Collection은 보통 메모리의 압박이 있을 때 수행되고 이 과정에서 Heap의 단편화(Fragmentation)가 발생할 수 있습니다. 이러한 문제를 해결하기 위해 Garbage Collection은 Compaction Algorithm을 사용합니다.

JVM Garbage Collection은 여러 알고리즘을 사용하는데요 대표적으로는 **Reference Counting Algorithm**이 존재합니다.

> ### Root Set
>
> **Reacheable Reference 여부를 파악하려면 항상 <u>유효한 최초의 참조</u>가 있어야 하는데 이를 객체 참조의 root set 이라고 합니다.**
>
> **Root Set의 구분**
> 
> 1. **Stack의 참조 정보**
>
>    Local Variable Section과 Operand Stack의 참조
>
> 2. **JVM Method Area의 Constant Pool에서 참조** (정적 변수에 의한 참조)
>
> 3. **Native Method Stack으로 넘겨진 Object의 Reference 참조**
>
> 이 3가지 Reference 정보에 의해 직간접적으로 참조되고 있다면 Reachable Object이고, 그렇지 않다면 모두 Garbage Object(Unreachable Object)이다.



### Reference Counting Algorithm이란 무엇인가요?

**Reference Counting Algorithm의 기본 아이디어는 각 Object마다 Reference Count를 관리하여 Object가 참조되면 Count를 1증가시키고 참조가 사라지면 1을 감소하는 식으로 동작합니다.** 이 알고리즘에서 객체 참조는 직접적인 참조와 간접적인 참조 모두를 포함합니다.

Reference Counting Algorithm은 Garbage Object를 확인하는 방법이 매우 간단하다는 장점이 있지만, 참조의 변경이 있을 때마다 Object의 Reference Count를 변경해줘야 하기 때문에 동기화 측면에서도 그리고 비용적인 측면에서도 좋지 않습니다.

LinkedList와 같은 Self Circular Reference 즉, 선형 참조의 경우에는 분명히 Garbage Object임에도 불구하고 Count가 0이 아니여서 Reachable Object로 간주되어 GC의 대상이 안된다면 메모리 누수(Memory Leak)가 발생할 수 있습니다.

> ### 메모리 누수 (Memory Leak)
>
> **메모리 누수란 프로그램에서 필요하지 않은 메모리들이 계속 점유되고 있는 현상을 의미합니다.** 즉, 더이상 사용하지 않는 객체가 GC에 의해서 회수되지 않고 계속 누적되는 현상을 의미합니다. 메모리 누수가 계속 중첩되다보면 heap 영역의 공간이 부족해져서 OutOfMemeoryError가 발생할 수 있습니다.


