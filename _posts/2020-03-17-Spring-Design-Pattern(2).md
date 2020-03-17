---
layout: post
title:  "Singleton, Template Method and Factory Method Pattern"
date:   2020-03-17
author: Green Frog Developer
categories: Spring
tags: Java OOP Spring Design-Pattern 
---

# Spring이 사랑하는 Design Pattern

이 글에서는 Spring이 사랑하고 Spring이 적용한 디자인 패턴중 **Singleton Pattern, Template Method Pattern, Factory Method Pattern**에 대해서 알아볼 것이다.

---

## Signleton Pattern

---

**Singleton Pattern이란 인스턴스를 하나만 만들어 사용하기 위한 패턴이다.**  (ex. 커넥션 풀, 스레드 풀, 디바이스 설정 객체)

**Singleton Pattern을 구현하려면 다음의 조건을 만족해야한다.**
- new를 실행할 수 없도록 <u>생성자에 private 접근 제어자를 지정</u>한다.
- 유일한 단일 객체(Singleton Object)를 반환할 수 있는 정적 메서드가 필요하다.
- 유일한 단일 객체(Singleton Object)를 참조할 정적 참조 변수가 필요하다.

예제를 통해 Singleton Pattern에 대해서 이해해 보자.

```java
public class Singleton {
    static Singleton singletonObject; // 정적 참조 변수 (Singleton 객체를 저장하기 위한 참조 변수)

    private Singleton() { } // private 생성자 (new를 실행할 수 없도록 하기 위함)

    // 객체 반환 정적 메서드 (Singleton Object를 반환할 수 있는 정적 메서드)
    public static Singleton getInstance() {
        if(singletonObject == null) {
            singletonObject = new Singleton();
        }

        return singletonObject;     // Singleton Object reference 반환
    }
}
```

`getInstance()` <u>정적 메서드를 보면 정적 참조 변수에 객체가 할당돼 있지 않은 경우에 new를 통해 객체를 만들고 정적 참조 변수에 할당한다.
그렇지 않으면 정적 참조 변수에 할당돼 있는 유일한 객체의 참조를 반환한다.</u>

```java
public class Client {
    public static void main(String[] args) {
        // private 생성자이므로 new를 통해 인스턴스를 생성할 수 없다.
        // Singleton s = new Singleton();

        Singleton s1 = Singleton.getInstance();
        Singleton s2 = Singleton.getInstance();
        Singleton s3 = Singleton.getInstance();

        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s3);     // 14번째 줄

        s1 = null;
        s2 = null;
        s3 = null;
    }
}
```

16번째 줄을 실행하기 직전의 <u>T 메모리 스냅샷</u>은 다음과 같다.

<img src="/assets/spring/Singleton-T-Memory.jpg" style="width:50%">

위 그림에서 4개의 참조 변수(`singletonObject`, `s1`, `s2`, `s3`)가 **하나의 단일 객체(singleton Object)를 참조**하는 것을 볼 수 잇다.

<u>단일 객체인 경우 결국 공유 객체로 사용되기 때문에 속성을 갖지 않게 하는 것이 정석이다.</u>
그 이유는 단일 객체(Singleton Object)가 속성을 갖게 되면 하나의 참조 변수가 변경한 단일 객체의 속성이 다른 참조 변수에 영향을 미치기 때문이다. (static을 쓰지 말라는 것과 같은 논리)

<u>다만 읽기 전용 속성을 갖는 것은 문제가 되지 않는다.</u> 이와 더불어 단일 객체가 다른 단일 객체에 대한 참조를 속성으로 가진 것 또한 문제가 되지 않는다.(Singleton Bean 제약조건)

**Singleton Pattern Feature**
- private 생성자를 갖는다.
- 단일 객체 참조 변수를 정적 속성으로 갖는다.
- 단일 객체 참조 변수가 참조하는 단일 객체(Singleton Object)를 반환하는 getInstance() 정적 메서드를 갖는다.
- 단일 객체는 쓰기 가능한 속성을 갖지 않는 것이 정석이다. (읽기 전용 속성을 갖는 것은 괜찮다.)

#### Singleton Pattern이란 클래스의 인스턴스, 즉 객체를 하나만 만들어 사용하는 패턴이다.

---

## Template Method Pattern

---

예제를 통해 Template Method Pattern이란 무엇인가에 대해서 알아보자!!

```java
abstract class Animal {
    // 템플릿 메서드
    public void playWithOwner() {
        System.out.println("귀염둥이 이리온...");
        play();
        runSomething();
        System.out.println("잘했어");
    }

    // 추상 메서드
    abstract void play();

    // Hook(갈고리) 메서드
    void runSomething() {
        System.out.println("꼬리 살랑 살랑~");
    }
}

class Dog extends Animal {
    @Override // 추상 메서드 오버라이딩
    void play() {
        System.out.println("멍! 멍!");
    }

    @Override // Hook(갈고리) 메서드 오버라이딩
    void runSomething() {
        System.out.println("멍! 멍!~ 꼬리 살랑 살랑~");
    }
}

class Cat extends Animal {
    @Override // 추상 메서드 오버라이딩
    void play() {
        System.out.println("야옹~ 야옹~");
    }

    @Override // Hook(갈고리) 메서드 오버라이딩
    void runSomething() {
        System.out.println("야옹~ 야옹~ 꼬리 살랑 살랑~");
    }
}

public class Driver {
    public static void main(String[] args) {
        Animal bolt = new Dog();
        Animal kitty = new Cat();

        bolt.playWithOwner();
        System.out.println();
        kitty.playWithOwner();
    }
}

/* 실행결과
귀염둥이 이리온..
멍! 멍!
멍! 멍!~ 꼬리 살랑 살랑~
잘했어

귀염둥이 이리온..
야옹~ 야옹~
야옹~ 야옹~ 꼬리 살랑 살랑~
잘했어
*/
```

위 예제에서 상위 클래스인 Animal에는 템플릿(견본)을 제공하는 `playWithOwner()` 메서드와 하위 클래스에서 구현을 강제하는 `play()` 추상 메서드, 
하위 클래스가 선택적으로 오버라이딩할 수 있는 `runSomething()` 메서드가 있다.

하위 클래스인 Dog과 Cat은 상위 클래스인 Animal에서 구현을 강제하고 있는 `play()` 메서드를 반드시 구현해야 한다. `runSomething()` 메서드는 선택적으로 오버라이딩 할 수 있다.

또한 위의 예제를 통해서 템플릿 메서드 패턴(Template Method Pattern)이 **의존 역전 원칙(DIP)**를 활용하고 있음을 알 수 있다. (<u>가변성이 더 낮은 추상클래스(Animal)에 의존</u>)

#### 템플릿 메서드 패턴(Template Method Pattern)이란 상위 클래스의 견본(Template) 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴이다.

---

## Factory Method Pattern

---

팩터리는 공장을 의미한다. 공장은 물건을 생산하는데 객체 지향에서 팩터리는 객체를 생산한다.

**즉, 팩터리 메서드(Factory Method)는 객체를 생성하고 반환하는 메서드를 말한다.** 
여기서 패턴(Pattern)이 붙으면 하위 클래스에서 팩터리 메서드를 오버라이딩해서 객체를 반환하게 하는 것을 의미한다.

예제를 통해 Factory Method Pattern에 대해 알아보자

```java
abstract class Animal {
    // 추상 팩터리 메서드
    abstract AnimalToy getToy(); 
}

// 팩터리 메서드가 생성할 객체의 상위 클래스 (팩터리 메서드가 반환하는 상위 타입)
abstract class AnimalToy {
    abstract void identify();
}

// 팩터리 메서드가 생성할 객체
class DogToy extends AnimalToy {
    @Override
    public void identify() {
        System.out.println("나는 테니스공! 강아지의 친구!");
    }
}

class Dog extends Animal {
    // 추상 팩터리 메서드 오버라이딩
    @Override
    AnimalToy getToy() {
        return new DogToy();    // DogToy 클래스의 인스턴스 반환
    }
}

//팩터리 메서드가 생성할 객체
class CatToy extends AnimalToy {
    @Override
    public void identify() {
        System.out.println("나는 캣타워! 고양이의 친구!");
    }
}

class Cat extends Animal {
    // 추상 팩터리 메서드 오버라이딩
    @Override
    AnimalToy getToy() {
        return new CatToy();    // CatToy 클래스의 인스턴스 반환
    }
}

public class Driver {
    public static void main(String[] args) {
        // 팩터리 메서드를 보유한 객체들 생성
        Animal bolt = new Dog();
        Animal kitty = new Cat();

        // 팩터리 메서드가 반환하는 객체들
        AnimalToy boltBall = bolt.getToy();
        AnimalToy kittyTower = kitty.getToy();

        //팩터리 메서드가 반환한 객체들을 사용
        boltBall.identity();
        kittyTower.identify();
    }
}
```

위 예제의 클래스 다이어그램은 다음과 같다.

<img src="/assets/spring/Factory-Method-Example-Class-Diagram.jpg" style="width:80%">

클래스 다이어그램을 위와 아래 반반씩 살펴보면 팩터리 메서드 패턴이 **의존 역전 원칙(DIP)**를 활용하고 있음을 알 수 있다. (<u>Animal, AnimalToy 추상클래스를 의존</u>)

#### 즉, 팩토리 메서드 패턴(Factory Method Pattern)이란 오버라이드된 메서드가 객체를 생성하고 반환하는 패턴을 말한다.

---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)