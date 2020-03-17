---
layout: post
title:  "Adapter, Proxy and Decorator Pattern"
date:   2020-03-17
author: Green Frog Developer
categories: Spring
tags: Java OOP Spring Design-Pattern 
---

# Spring이 사랑하는 Design Pattern

이 글에서는 Spring이 사랑하고 Spring이 적용한 디자인 패턴중 **Adapter, Proxy and Decorator Design Pattern**에 대해서 알아볼 것이다.

---

## Adapter Pattern

---

어댑터를 번역하면 **변환기(Converter)** 라고 할 수 있다. 변환기의 역할은 서로 다른 <u>두 인터페이스 사이에 통신</u>이 가능하게 하는 것이다.
주변에서 가장 흔히 볼 수 있는 변환기로는 핸드폰과 전원 콘센트사이에서 둘을 연결해 주는 **충전기**가 있다.

또 다른 예로는 데이터베이스 관련 프로그램을 작성해 본 사람이라면 **ODBC/JDBC 인터페이스**를 알고 있을것이다.
바로 ODBC/JDBC가 **어댑터 패턴을 이용**해 다양한 데이터베이스 시스템을 단일한 인터페이스로 조작할 수 있게끔 해주는 대표적인 예이다.

이제는 직접 예제 코드를 통해 어댑터 패턴을 이해해 보자.

**\<어댑터 패턴이 적용되지 않은 코드\>**

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

public class ClientWithNoAdapter {
   public static void main(Stirng[] args) {
      ServiceA sa1 = new ServiceA();
      ServiceB sb1 = new ServiceB();

      sa1.runServiceA();
      sb1.runServiceB();
   }
}
```

클라이언트(ClientWithNoAdapter)를 살펴보면 `sa1` 참조 변수와 `sb1` 참조 변수를 통해 호출하는 각 메서드가 비슷한 일을 하지만 <u>메서드명이 다른 것</u>을 볼 수 있다. 
<u>(추후 유지보수 및 확장에 중요한 문제를 일으킬 수 있다.)</u>

**\<어댑터 패턴이 적용된 코드\>**  (ServiceA에 대한 변환기와 ServiceB에 대한 Adapter 추가)

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

class AdapterServiceA {
   ServiceA sa1 = new ServiceA();

   void runService() {
      sa.runServiceA();
   }
}

class AdapterServiceB {
   ServiceB sb1 = new ServiceB();

   void runService() {
      sb1.runServiceB();
   }
}

public class ClientWithAdapter {
   public static void main(Stirng[] args) {
      AdapterServiceA asa1 = new AdapterServiceA();
      AdapterServiceB asb1 = new AdapterServiceB();

      asa1.runService();
      asb1.runService();
   }
}
```

클라이언트(ClientWithAdapter)가 변환기(Adapter)를 통해 `runService()`라는 동일한 메서드명으로 두 객체의 메서드를 호출하는 것을 볼 수 있다.

위의 두개 예제를 비교해 보니까 어댑터 패턴(Adapter Pattern)에 대해서 느낌이 오는가? 
그렇다. 어댑터 패턴은 합성. **즉 객체를 속성으로 만들어서 참조하는 디자인 패턴이다.** 

#### 어댑터 패턴이란 한마디로 호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기(Adapter)를 통해 호출하는 패턴이다.

---

## Proxy Pattern

---

프록시는 **대리자, 대변인**이라는 뜻을 가진 단어이다. 대리자/대변인이라고 하면 다른 누군가를 대신해서 그 역할을 수행하는 존재를 말한다.

그렇다면 프록시(대리자, 대변인)가 어떻게 쓰이는지에 대해서 코드를 통해 알아보자.

**\<프록시 패턴이 적용되지 않은 코드\>**
```java
class Service {
   public String runSomething() {
      return "서비스 짱!!!";
   }
}

public class ClientWithNoProxy {
   public static void main(String[] args) {
      // 프록시를 이용하지 않는 호출
      Service service = new Service();
      System.out.println(service.runSomething());
   }
}
```

위의 코드에서는 `ClientWithNoProxy`가 `Service`객체의 `runSomething()`메서드를 직접 호출하는 것을 볼 수 있다.

**\<프록시 패턴이 적용된 코드\>**
```java
interface IService {
   String runSomething();
}

class Service implements IService {
   public String runSomething() {
      return "서비스 짱!!!";
   }
}

class Proxy implements IService {
   IService service1;

   public String runSomething() {
      System.out.println("호출에 대한 흐름 제어가 주목적, 반환 결과를 그대로 전달")

      service1 = new Service();
      return service1.runSomethig();
   }
}

public class ClientWithProxy {
   public static void main(String[] args) {
      // 프록시를 이용한 호출
      Iservice proxy = new Proxy();
      System.out.println(proxy.runSomething());
   }
}
```
위 예제의 프록시 패턴에 적용된 설계 패턴
1. 개방 페쇄 원칙(OCP) : `Iservice` 인터페이스의 참조변수를 사용해 변화에는 닫혀있고 확장에는 열려 있다.
2. 의존 역전 원칙(DIP) : `IService` 인터페이스에 의존

**프록시 패턴의 경우 실제 서비스 객체가 가진 메서드와 같은 이름의 메서드를 사용하는데, 이를 위해 <u>인터페이스를 사용</u>한다.** 

인터페이스를 사용하면 서비스 객체가 들어갈 자리에 Proxy 객체를 대신 투입해 <u>클라이언트 쪽에서는</u> 실제 서비스 객체를 통해 메서드를 호출하고 반환값을 받는지, Proxy 객체를 통해 메서드를 호출하고 반환값을 받는지 전혀 모르게 처리할 수 있다.

**프록시 패턴의 중요 포인트**
- Proxy는 실제 서비스와 같은 이름의 메서드를 구현한다. 이 때 **인터페이스**를 사용한다.
- Proxy는 실제 서비스에 대한 참조 변수를 갖는다.(합성)
- Proxy는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 돌려준다.
- Proxy는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다.

프록시 패턴은 실제 서비스 메서드의 반환값에 가감하는 것을 목적으로 하지 않고 <u>제어의 흐름을 변경하거나 다른 로직을 수행</u>하기 위해 사용한다.

#### 즉, 프록시 패턴이란 제어 흐름을 조정하기 위한 목적으로 중간에 대리자(Proxy)를 두는 패턴이다.

---

## Decorator Pattern

---

데코레이터는 **도장/도배업자**를 의미한다. 여기서는 **장식자**라는 뜻을 가지고 논리를 풀어보자!!

데코레이터 패턴은 프록시 패턴과 구현 방법이 같다. 다만 프록시 패턴은 클라이언트가 최종적으로 돌려 받는 반환값을 조작하지 않고 그대로 전달하는 반면
**데코레이터 페턴은 클라이언트가 받는 반환값에 장식을 덧입힌다.**

**\<데코레이터 패턴이 적용된 코드\>**
```java
interface IService {
   public abstract String runSomething();
}

class service implements IService {
   public String runSomething() {
      return "서비스 짱!!!";
   }
}

class Decorator implements IService {
   IService service

   public String runSomething() {
      System.out.println("호출에 대한 장식이 주목적, 클라이언트에게 반환 결과에 장식을 더하여 전달");

      service = new Service();
      return "정말" + service.runSomething();
   }
}

public class ClientWithDecorator {
   public static void main(String[] args) {
      Iservice decorator = new Decorator();
      System.out.println(decorator.runSomething());
   }
}
```
위 예제의 데코레이터 패턴에 적용된 설계 패턴
1. 개방 페쇄 원칙(OCP) : `Iservice` 인터페이스의 참조변수를 사용해 변화에는 닫혀있고 확장에는 열려 있다.
2. 의존 역전 원칙(DIP) : `IService` 인터페이스에 의존

**Decorator Pattern의 중요 포인트는 다음과 같다.**
- Decorator는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 **인터페이스**를 사용한다.
- Decorator는 실제 서비스에 대한 참조 변수를 갖는다.(합성)
- Decorator는 실제 서비스의 같은 이름을 가진 메서드를 호출하고, 그 반환값에 장식을 더해 클라이언트에게 돌려준다.
- Decorator는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다.

#### 즉, 데코레이터 패턴이란 메서드의 호출의 반환값에 변화를 주기 위해 중간에 장식자(Decorator)를 두는 패턴이다.

---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)