---
layout: post
title:  "Strategy Pattern and Template Callback Pattern"
date:   2020-03-18
author: Green Frog Developer
categories: Spring
tags: Java OOP Spring Design-Pattern 
---

# Spring이 사랑하는 Design Pattern

이 글에서는 Spring이 사랑하고 Spring이 적용한 디자인 패턴중 **Strategy Pattern, Template Callback Pattern**에 대해서 알아볼 것이다.

---

## Strategy Pattern

---

**전략 패턴(Strategy Pattern)을 구성하는 세 요소는 다음과 같다.**
- 전략 메서드를 가진 전략 객체
- 전략 객체를 사용하는 컨텍스트(전략 객체의 사용자/소비자)
- 전략 객체를 생성해 컨텍스트에 주입하는 클라이언트(제3자, 전략 객체의 공급자)

 이를 그림으로 나타내면 다음과 같다.

<img src="/assets/spring/Strategy-Pattern-Component.png" style="width:70%">


전략 패턴을 구성하는 세 요소에 대해서 알아봤으니 이제 전략 패턴이 어떻게 쓰이는지 예제를 통해서 알아보자.

먼저 코드를 보기전에 요구사항부터 가정해 보자. 군인이 있다고 상상해 보자. 그리고 그 군인이 사용할 무기가 있다고 하자. 보급 장교가 무기를 군인에게 지급해 주면
군인은 무기에 따라 전투를 수행하게 된다. <u>이 이야기를 전략 패턴에 따라 구분해 보면 무기는 전략이 되고, 군인은 컨텍스트, 보급 장교는 제3자, 즉 클라이언트가 된다.</u>
이를 전략 패턴(Strategy Pattern)을 사용해서 구현해 보자.

```java
// 다양한 무기(전략)를 공통된 방식으로 사용하기 위해 인터페이스 정의!!! (아주 중요)
interface Strategy {
   public abstract void runStrategy(); // 전략 메서드
}

// 다양한 전략, 즉 무기를 구현하자. 먼저 총이다.
class StrategyGun implements Strategy {
   @Override
   public void runStrategy() {
      System.out.println("탕, 타당, 타다당");
   }
}

// 이번에는 검(전략)이다.
class StrategySword implements Strategy {
   @Override
   public void runStrategy() {
      System.out.println("챙.. 채쟁챙 챙챙");
   }
}

// 마지막으로 활(전략)이다.
class StrategyBow implements Strategy {
   @Override
   public void runStrategy() {
      System.out.println("슝.. 쐐액.. 쉑, 최종 병기");
   }
}

// 이번에는 무기(전략)를 사용할 군인(컨텍스트)을 구현하자.
class Solider {
   void runContext(Strategy strategy) {
      System.out.println("전투 시작");
      strategy.runStrategy();
      System.out.println("전투 종료");
   }
}

// 마지막으로 무기(전략)를 조달(생성)해서 군인(컨텍스트)에게 지급(주입)해 줄 보급장교(클라이언트, 제3자)를 구현하자.
public class Client {
   public static void main(String[] args) {
      Strategy strategy = null;
      Solider rambo = new Solider;  // 컨텍스트 생성

      // 총(전략)을 람보(컨텍스트)에게 전달해서 전투를 수행하게 한다.
      strategy = new StrategyGun(); // 전략 생성
      rambo.runContext(strategy);   // 컨텍스트에게 전략 전달 및 사용
      System.out.println();

      // 검(전략)을 람보(컨텍스트)에게 전달해서 전투를 수행하게 한다.
      strategy = new StrategySword();  // 전략 생성
      rambo.runContext(strategy);      // 컨텍스트에게 전략 전달 및 사용
      System.out.println();

      // 활(전략)을 람보(컨텍스트)에게 전달해서 전투를 수행하게 한다.
      strategy = new StrategyBow();    // 전략 생성
      rambo.runContext(strategy);      // 컨텍스트에게 전략 전달 및 사용
   }
}
/* 실행결과
전투시작
탕, 타당, 타다당
전투종료

전투시작
챙.. 채쟁챙 챙챙
전투종료

전투시작
슝.. 쐐액.. 쇅, 최종 병기
전투종료
*/
```

위의 코드처럼 전략을 다양하게 변경하면서 컨텍스트를 실행할 수 있다. **전략 패턴은 다양한 곳에서 다양한 문제 상황의 해결책으로 사용되기 떄문에 <u>디자인 패턴의 꽃</u>이라고 불리운다.**

만약 전략 패턴이 템플릿 메서드 패턴과 유사하다고 생각한다면 아주 제대로 생각한 것이다. 
**그렇지만 단일 상속만이 가능한 자바 언어에서 상속이라는 제한이 있는 <u>템플릿 메서드 패턴</u>보다는 객체를 주입하는 <u>전략 패턴</u>이 더 많이 활용된다.**

다음의 그림은 위 예제를 클래스 다이어그램으로 나타낸 것이다.

<img src="/assets/spring/Strategy-Pattern-Example-Class-Diagram.png" style="width:80%">

위의 클래스 다이어그램을 보면 이 예제는 전략 패턴(Strategy Pattern)을 적용해서 **개방 폐쇄 원칙(OCP)과 의존 역전 원칙(DIP)를 적용**한 것이라고 짐작할 수 있다.

#### 전략 패턴(Strategy Pattern)이란 클라이언트가 전략을 생성해 전략을 실행할 컨텍스트에 주입하는 패턴을 의미한다.

---

## Template Callback Pattern

---

**템플릿 콜백 패턴은 전략 패턴의 변형으로, 스프링의 3대 프로그래밍 모델 중 하나인 DI(의존성 주입)에서 사용하는 특별한 형태의 전략 패턴이다.**

템플릿 콜백 패턴은 전략 패턴과 모든 것이 동일한데 **전략을 익명 내부 클래스로 정의**해서 사용한다는 특징이 있다.

앞에서 살펴본 전략 패턴(Strategy Pattern) 코드를 템플릿 콜백 패턴(Template Callback Pattern)으로 바꿔보자!!

```java
interface Strategy {    // 전략 인터페이스
   public abstract void runStrategy();
}

class Solider {         // 컨텍스트
   void runContext(String weaponSound) {
      System.out.println("전투 시작");
      executeWeapon(weaponSound).runStrategy();
      System.out.println("전투 종료");
   }

   private Strategy executeWeapon(final String weaponSound) {
      return new Strategy() {       // 전략을 생성하는 코드가 컨텍스트 내부로 들어왔다. (익명 내부 클래스)
         @Override
         public void runStrategy() {
            System.out.println(weaponSound);
         }
      };
   }
}

public class Client {      // 클라이언트
   public static void main(String[] args) {
      Solider rambo = new Solider();

      rambo.runContext("총! 총조종총 총! 총!");
      System.out.println();
      rambo.runContext("칼! 카가갈 칼! 칼!");
      System.out.println();
      rambo.runContext("도끼! 독독.. 도도독 독끼!");
   }
}
```

스프링은 이런 형식으로 템플릿 콜백 패턴을 DI(Dependency Injection)에 적극 활용하고 있다.

위의 예제에서는 템플릿 콜백 패턴을 사용해서 **개방 폐쇄 원칙(OCP)와 의존 역전 원칙(DIP)**를 적용하고 있다.

#### 템플릿 콜백 패턴(Template Callback Pattern)이란 전략을 익명 내부 클래스로 구현한 전략 패턴이다.

---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)