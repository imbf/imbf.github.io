---
layout: post
title:  "[Operating System - Chapter 6] 동기화 도구들"
date:   2020-11-04
author: Green Frog Developer
categories: Computer-Science(CS)
tags: Operating-System CS Mutex Lock Semaphore Monitor
---

이 포스팅은 공룡책으로 알려진 Operating System Concepts의 6장인 **Synchronization Tools**를 공부하면서 정리한 포스팅이다.

---

## 6. 동기화 도구들 (Synchronization Tools)

---

**협력적 프로세스는 시스템 내에서 실행 중인 다른 프로세스의 실행에 영향을 주거나 영향을 받는 프로세스이다.**

이 장에서는 논리 주소 공간을 공유하는 협력적 프로세스의 질서 있는 실행을 보장하여, 이를 통해 **데이터의 일관성**을 유지하는 다양한 메커니즘에 대해 논의하도록 하겠다.

---

### 6.1 배경 (Background)

---

우리는 이미 프로세스가 병행하게 또는 병렬로 실행될 수 있다는 것을 알고 있다. 본 장에서는 프로세스가 병행 또는 병렬로 실행될 때 여러 프로세스가 공유하는 **데이터의 무결성**에 어떤 문제를 일으키는지에 관해 설명한다.

**동시에 여러 개의 프로세스가 동일한 자료를 접근하여 조작하고, 그 실행 결과가 접근이 발생한 특정 순서에 의존하는 상황을 경쟁 상황(race condition)이라고 한다.** 경쟁 상황으로부터 보호하기 위해, 우리는 한순간에 하나의 프로세스만이 데이터를 조작하도록 보장해야 한다. 이러한 보장을 위해, 우리는 어떤 형태로든 프로세스들이 동기화되도록 할 필요가 있다.

또한 다중 코어 시스템의 대두와 더불어 다중 스레드 어플리케이션의 개발에 대한 관심이 증가하고 있다. 다중 스레드 어플리케이션에서는 자원을 공유할 가능성이 매우 높은 여러 스레드가 서로 다른 처리 코어에서 병렬로 실행된다. 이러한 상황에서 race condition은 더 빈번하게 발생할 수 있어, 스레드들이 동기화되도록 할 필요가 있다.

---

### 6.2 임계구역 문제 (The Critical Section Problem)

---

프로세스 동기화에 관한 논의는 소위 임계구역 문제라고 불리는 문제로부터 시작한다. n개의 프로세스 {P0, P1, ..., P(n-1)}이 있는 시스템을 고려해 보자.

각 프로세스는 **임계구역(critical section)**이라고 부르는 코드를 포함하고 있고, 그 안에서는 적어도 하나 이상의 다른 프로세스와 공유하는 데이터에 접근하고 갱신할 수 있다. 이 시스템의 중요한 특징은 **한 프로세스가 자신의 임계구역에서 수행하는 동안에는 다른 프로세스들은 그들의 임계구역에 들어갈 수 없다는 사실이다.**

**임계구역 문제는 프로세스들이 데이터를 협력적으로 공유하기 위하여 자신들의 활동을 동기화할 때 사용할 수 있는 프로토콜(약속)을 설계하는 것이다.**

각 프로세스는 자신의 임계구역으로 진입하려면 진입 구역(entry section)에서 진입 허가를 요청해야 하고, 임계구역 뒤에는 퇴출 구역(exit section)이 따라올 수 있다. 코드의 나머지 부분들을 총칭하여 나머지 구역(reaminder section)이라고 한다.

이를 사용하는 프로세스의 일반적인 구조를 나타낸 그림은 아래와 같다.

<img src="/assets/computer-science/synchronization-tools-1.png" style="width:70%">

아래의 그림에 나와 있는 것처럼 entry section과 exit section은 프로세스의 동기화를 위해서 매우 중요하다.

임계구역 문제에 대한 해결안은 다음의 **세 가지 요구 조건**을 충족해야 한다.

1. **상호 배제(mutual exclusion):** 프로세스 P가 자기의 임계구역에서 실행된다면, 다른 프로세스들은 그들 자신의 임계구역에서 실행될 수 없다.
2. **진행(progress):** 자기의 임계구역에서 실행되는 프로세스가 없고 그들 자신의 임계구역으로 진입하려고 하는 프로세스들이 있다면, 나머지 구역에서 실행 중이지 않는 프로세스들만 다음에 누가 그 임계구역으로 진입할 수 있는지를 결정하는 데 참여할 수 있으며, 이 선택은 무한정 연기될 수 없다.
3. **한정된 대기(bounded waiting):** 프로세스가 자기의 임계구역에 진입하려는 요청을 한 후부터 그 요청이 허용될 때까지 다른 프로세스들이 그들 자신의 임계구역에 진입하도록 허용되는 횟수에 한계가 있어야 한다.

임의의 한순간에 많은 커널 모드 프로세스들이 운영체제 안에서 활성화될 수 있기 때문에 운영체제를 구현하는 코드(커널 코드)는 경쟁 조건이 발생하기 쉽다. <u>경쟁 조건이 발생하기 쉬운 커널 자료구조로는 메모리 할당을 관리하는 자료구조, 프로세스 리스트를 유지하는 자료구조, 인터럽트 처리를 위한 자료구조 등이 있다.</u>

운영체제 내에서 임계구역을 다루기 위해서 **선점형 커널**과 **비선점형 커널**의 두 가지 일반적인 접근법이 사용된다.
- **선점형 커널은 프로세스가 커널 모드에서 수행되는 동안 선점되는 것을 허용한다.**
    - 선점형 커널에 대해서는 동일한 주장을 할 수 없어 경쟁 조건이 발생할 수 있기 때문에 공유되는 커널 자료구조에서 경쟁 조건이 발생하지 않는다는 것을 보장하도록 신중하게 설계되어야 한다.
- **비선점형 커널은 커널 모드에서 수행되는 프로세스의 선점을 허용하지 않고 커널 모드 프로세스는 커널을 빠져나갈 때까지 또는 봉쇄될 때까지 또는 자발적으로 CPU의 제어를 양보할 때까지 계속 수행된다.**
    - 한 순간에 커널 안에서 실행 중인 프로세스는 하나밖에 없으므로 경쟁 조건을 염려할 필요는 없다.

커널 모드 프로세스가 대기 중인 프로세스에 프로세서를 양도하기 전에 오랫동안 실행할 위험이 적기 때문에 선점형 커널은 더 응답이 민첩할 수 있다. 하지만 선점형 커널을 설계하기 앞서 경쟁 조건을 잘 고려해야만 한다.

---

### 6.3 Peterson의 해결안 (Peterson's Solution)

---

Peterson 해결안은 Critical Section과 Remainder Section을 번갈아 가며 실행하는 두 개의 프로세스로 한정된다. 보통 프로세스는 P0와 P1로 번호를 매긴다. Peterson의 해결안은 두 프로세스가 두 개의 데이터 항목을 공유하도록 하여 해결한다. 이는 다음과 같다.

```java
int turn; // Critical Section으로 진입할 순번을 나타내는 변수
boolean flag[2]; // 각 프로세스가 Critical Section으로 진입할 준비가 되었다는 것을 표현한 배열
```
> flag[i]가 참이라면 Pi가 Critical Section으로 진입할 준비가 된다.

**Pi의 Peterson 알고리즘은 다음과 같다.**

<img src="/assets/computer-science/synchronization-tools-2.png" style="width:70%">


Critical Section으로 진입하기 위해서는 Pi는 먼저 flag[i]를 참으로 만들고, turn을 j로 지정한다. 이렇게 함으로써 Pi는 Pj가 Critical Section으로 진입하기를 원한다면 진입 가능하다는 것을 보장한다. 만일 두 프로세스가 동시에 진입하기를 원한다면 진입 turn은 거의 동시에 i와 j로 지정될 것이다. 이 때의 경우 turn의 궁극적인 값이 둘 중 누가 먼저 Critical Section으로 진입할 것인가를 결정한다.

**flag[j]가 false(Pj가 remainder section 수행)가 되거나 turn이 i(Pj는 준비완료됬고, Pi가 Entry Section에서 대기하고 있음)일 경우 Pi는 Critical Section에 들어갈 수 있다.**

Peterson의 해결안이 Ciritical Section 문제를 해결하기 위해선 위에서 명시한 것 처럼 3가지 요구조건을 만족해야 한다. 같이 살펴보자.

1. 상호 배재(Mutual Exclusion)
    - flag[2]와 turn 변수에 의해서 하나의 프로세스만 Critical Section에서 연산을 수행할 수 있음으로 Mutual Exclusion는 지켜진다.
2. 진행(Progress)
    - 각 프로세스가 자신이 Critical Section을 수행할 동안 while문에서 다른 프로세스를 유한하게 대기하도록 만드는 방법을 통하여 Progress를 지킬 수 있다.
3. 한정된 대기(bounded waiting)
    - 각 프로세스들은 Critical Section에 진입하려는 요청을 한 후부터 다른 프로세스가 Critical Section을 수행하는 동안 유한하게 대기함으로 bounded waiting 또한 지켜진다.

Peterson의 해결안은 최신 컴퓨터 아키텍처에서 작동한다고 보장되지 않는다. <u>주된 이유는 시스템 성능을 향상하기 위해 프로세스 또는 컴파일러가 종속성이 없는 읽기 및 쓰기 작업을 재정렬</u> 할 수 있기 때문이다. 예로 들어, Peterson 해결안의 entry section의 첫 두 문장의 순서를 바꾸게 되면 임계구역 문제를 해결할 수 없다.

---

### 6.4 동기화를 위한 하드웨어 지원 (Hardware Support for Synchronization)

---

Critical Section 문제의 소프트 웨어 기반 해결책은 최신 컴퓨터 아키텍처에서 작동하지 않을 수 있다.

이 절에서는 Critical Section 문제를 해결하기 위한 지원을 제공하는 **세 가지 하드웨어 명령**을 제시한다. 이러한 프리미티브(초기의, 원초적인, ...) 연산은 동기화 도구로 직접 사용될 수 있거나 더 추상적인 동기화 기법의 기초 형태로 사용될 수 있다.

#### 6.4.1 메모리 장벽 (Memory Barriers)

**컴퓨터 아키텍처가 응용 프로그램에게 제공하는 메모리 접근 시 보장되는 사항을 결정한 방식을 메모리 모델이라고 한다.**

일반적으로 메모리 모델은 두 가지 범주 중 하나에 속한다.
1. **강한 순서:** 한 프로세서의 메모리 변경 결과가 다른 모든 프로세서에 즉시 보임.
2. **약한 순서:** 한 프로세서의 메모리 변경 결과가 다른 프로세서에 즉시 보이지 않음.

**컴퓨터 아키텍처는 메모리의 모든 변경 사항을 다른 모든 프로세서로 전파하는 명령어를 제공하여 다른 프로세서에서 실행 중인 스레드에 메모리 변경 사항이 보이는 것을 보장**한다. 이러한 명령어를 **메모리 장벽(Memory Barriers) 또는 메모리 펜스(Memory Fences)**라고 한다.

메모리 장벽 명령어가 실행될 때, 시스템은 후속 적재 또는 저장 연산이 수행되기 전에 모든 적재 및 저장이 완료되도록 한다. 따라서 명령이 재정렬 되더라도 메모리 장벽은 향후 적재 또는 저장 작업이 수행되기 전에 저장 작업이 메모리에서 완료되어 다른 프로세서에 보이도록 한다.

메모리 장벽은 매우 낮은 수준의 연산으로 간주하며 일반적으로 Mutual Exclusion을 보장하는 특수 코드를 작성할 때 커널 개발자만 사용한다.

#### 6.4.2 하드웨어 명령어 (Hardware Instructions)

**많은 현대 기계들은 한 워드(word)의 내용을 검사하고 변경하거나, 두 워드의 내용을 원자적으로 교환(swap)할 수 있는, 즉 인터럽트 되지 않는 하나의 단위로서, 특별한 하드웨어 명령어들을 제공한다.** 우리는 이들을 사용하여 간단한 방식으로 Critical Section을 해결할 수 있다. 

이 명령어들을 추상적으로 표현하자면 test_and_set()과 compare_and_swap()이 있다.

**test_and_set()** 명령어는 원자적(atomically)으로 실행된다. 아래의 그림은 test_and_set() 명령어의 정의문이다.

<img src="/assets/computer-science/synchronization-tools-3.png" style="width:70%">

이를 활용한 Critical Section의 Mutucal Exclsion 해결 알고리즘은 아래와 같다.

<img src="/assets/computer-science/synchronization-tools-6.png" style="width:70%">

위의 알고리즘은 lock이라는 공유 변수가 한 프로세스에서 false일 경우 lock을 true로 만들고 자신은 Critical Section으로 들어간다. Critical Section에서 빠져나온 프로세스가 lock을 false로 만들면 다른 프로세스가 Critical Section에 들어가고 이러한 과정이 반복되는 알고리즘이다.


**compare_and_swap()** 명령어는 test_and_set() 명령어와 마찬가지로 두 개의 워드에 원자적인 연산을 하지만 두 워드의 내용 교환에 기반을 둔 다른 기법을 사용한다. 아래의 그림은 compare_and_swap() 명령어의 정의문이다.

<img src="/assets/computer-science/synchronization-tools-4.png" style="width:70%">

Critical Section 요구 조건을 모두 만족시키는 compare_and_swap() 명령어를 이용한 알고리즘에 대해서 설명하도록 하겠다. 공통 데이터 및 코드는 아래와 같다.

```java
boolean waiting[n]; // 프로세스i가 대기하고 있다는 것을 나타내는 배열
int lock;
```

<img src="/assets/computer-science/synchronization-tools-5.png" style="width:70%">

Pi가 임계구역에 진입하는 경우는 오직 waiting[i] == false 이든지 key == 0 이 되어야 한다. lock이 0일 때에만 key가 0 이되고 lock이 1로 바뀌면서 무한 루프에서 빠져나올 수 있다.

waiting[i]를 false로 바꾸고 임계 구역을 실행한 뒤 i를 순차적으로 증가시켜 자신과 같을 때 까지 또는 i가 기다리고 있을 때 까지 무한 루프를 돌고, 선택된 프로세스를 Ciritical Section에 진입할 수 있도록 해주는 알고리즘이다.

위의 알고리즘은 Ciritical Section 해결안이 만족해야 하는 요구조건인 Mutual Exclusion, Progress, Bounded Waiting를 모두 만족한다.

#### 6.4.3 원자적 변수 (Atomic Variables)

**원자적 변수(atomic variable)은 정수 및 부울과 같은 기본 데이터 유형에 대한 원자적 연산을 제공한다. 원자적 변수는 카운터가 증가할 때와 같이 갱신되는 동안 단일 변수에 대한 데이터 경쟁이 있을 수 있는 상황에서 상호 배제를 보장하는데 사용할 수 있다.**

원자적 변수를 지원하는 대부분의 시스템은 원자적 변수에 접근하고 조작하기 위한 기능뿐만 아니라 특별한 원자적 데이터 유형을 제공한다.

원자적 변수는 운영체제 및 병행 응용 프로그램에서 일반적으로 사용되지만 카운터 및 시퀀스 생성기와 같은 공유 데이터 한 개의 갱신에만 제한되는 경우가 많다.

---

### 6.5 Mutex Locks

---

**mutex lock은 mutual exclusion lock의 축약 형태로서 Critical Section을 해결하기위한 하드웨어 기반의 해결책보다 상위 수준의 해결책이다.** 우리는 Critical Section을 보호하고, 따라서 Racing Condition을 방지하기 위해 mutex lock을 사용한다. 

**즉, 프로세스는 Critical Section에 들어가기 전에 반드시 lock을 획득해야 하고 Critical Section을 빠져나올 때 lock을 반환해야 한다.**

mutex lock 에 대해서 알기 위해서 우리는 3가지를 알아야 한다.
1. acquire(): 락을 획득하는 함수
2. release(): 락을 반환하는 함수
3. available: 락의 가용 여부를 표시하는 변수

acquire() 함수의 정의는 다음과 같다.

```java
acquire() {
    while(!available)
        ; /* busy wait */
    available = false;
}
```

release() 함수의 정의는 다음과 같다.
```java
release() {
    available = true;
}
```

아래의 그림은 Mutex lock을 사용한 Critical Section 문제 해결 알고리즘이다.

<img src="/assets/computer-science/synchronization-tools-7.png" style="width:70%">

지금까지 설명한 구현 방식의 단점은 **바쁜 대기(busy waiting)**를 해야 한다는 것이다. 프로세스가 임계구역에 있는 동안 임계구역에 들어가기 원하는 다른 프로세스들은 acquire() 함수를 호출하는 반복문을 계속 실행해야 한다. 이러한 busy waiting은 다른 프로세스가 생산적으로 사용할 수 있는 CPU 주기를 낭비한다.

우리가 설명한 mutex lock 유형을 **스핀락(spinlock)**이라고도 한다. 락을 사용할 수 있을 때까지 프로세스가 회전하기 때문이다. 그러나 스핀락은 프로세스가 락을 기다려야 하고 문맥 교환에 상당한 시간이 소요될 때 문맥 교환이 필요하지 않다는 장점이 있다.

최신 다중 코어 컴퓨팅 시스템에서 스핀락은 많은 운영체제에서 널리 사용된다. **일반적으로 락이 유지되는 기간이 문맥 교환을 두 번(1.스레드를 대기상태로 2.대기중인 스레드를 복원) 하는 시간보다 짧은 경우 스핀락을 사용한다.**

> **락 경합(Lock Competition)**
>
> 락에 대한 경합 상태일 수도 비경합 상태일 수도 있다.
>
> 락을 획득하려고 시도하는 동안 스레드가 봉쇄되면 락은 경합 상태로 간주한다.
>
> 스레드가 락을 얻으려고 시도할 때 락을 사용할 수 있으면 락은 비경합 상태로 간주한다.
>
> **높은 경합 상태의 락은 병행 실행 응용 프로그램의 성능을 전체적으로 저하한다.**



---

### 6.6 세마포 (Semaphores)

---

**세마포 S는 정수 변수로서, 초기화를 제외하고는 단지 두 개의 표준 원자적(atomical) 연산 wait()와 signal()로만 접근할 수 있다.**

wait() 연산은 원래 "검사하다"를 의미하는 네덜란드어 proberen에서 **P**, 그리고 signal() 연산은 "증가하다"를 의미하는 verhogen에서 **V**라고 지어졌다.

wait()의 정의는 다음과 같다.

```c
wait(S) {
    while (S <= 0)
        ; // busy wait
    S--;
}
```

signal()의 정의는 다음과 같다.

```c
signal(S) {
    S++;
}
```

wait()와 signal() 연산 시 세마포의 정수 값을 변경하는 연산은 반드시 원자적으로 수행되어야 한다.

#### 6.6.1 세마포 사용법 (Semaphore Usage)

운영체제는 종종 카운팅(counting)과 이진(binary) 세마포를 구분한다. **카운팅 세마포의 값은 제한 없는 영역을 갖지만 이진 세마포의 값은 0과 1사이의 값만 가능하다.**

**카운팅 세마포는 유한한 개수를 가진 자원에 대한 접근을 제어하는데 사용될 수 있다.** 세마포는 가용한 자원의 개수로 초기화된다. 각 자원을 사용하려는 프로세스는 세마포에 wait() 연산을 수행하며, 이때 세마포의 값은 감소한다. 프로세스가 자원을 방출할 때는 signal() 연산을 수행하고 세마포는 증가하게 된다. **세마포의 값이 0이 되면 모든 자원이 사용중임을 나타낸다.** 이후 자원을 사용하려는 프로세스는 세마포 값이 0보다 커질 때까지 봉쇄된다.

#### 6.6.2 세마포 구현 (Semaphore Implementation)

**Busy Waiting을 피하기 위해 세마포 S를 대기하면서 일시 중지된 프로세스는 다른 프로세스가 signal() 연산을 실행하면 재시작되어야 한다.** 프로세스는 sleep() 연산에 의해서 일시 중지되고 wakeup() 연산에 의하여 재시작된다. (대기상태 <-> 준비 완료 상태)

세마포를 활용한 Critical Section 문제 해결 알고리즘을 구현하기 위해 세마포의 정의는 다음과 같다.

```c
typedef struct {
    int value; 
    struct process *list;
}semaphore;
```

wait() 연산의 정의는 다음과 같다.
```c
wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        add this process to S -> list;
        sleep();
    }
}
```

signal() 연산의 정의는 다음과 같다.
```c
signal(semaphore *S) {
    S->value ++;
    if (S->value <= 0) {
        remove a process P from S -> list;
        wakeup(P);    
    }
}
```

wakeup()과 sleep()은 프로세스를 일시 중지 or 재실행시키는 운영체제의 기본적인 시스템 콜이다.

세마포의 프로세스 리스트는 Bounded Waiting를 보장하도록 잘 구현해야만 한다.

단일 Processor 환경에서는 wait()와 signal() 연산들이 원자적으로 실행되는것을 보장하기 위해 실행되는 동안 인터럽트를 금지함으로써 해결할 수 있지만, 다중 코어 환경에서는 모든 처리 코어에서 인터럽트를 금지하여야만 한다. 이는 매우 어려울 수 있으며 성능을 심각하게 감소시킬 수 있음으로 많은 부분을 고려해야만 한다.

---

### 6.7 모니터 (Monitors)

---

mutex락 혹은 세마포를 사용할 때에도 타이밍 오류는 여전히 발생할 수 있다. 예로들어, wait()과 signal() 연산의 순서가 뒤바뀌는 경우, Critical Section이 끝나고 signal()대신 wait()이 호출되는 경우

이러한 오류를 처리하기 위한 한 가지 전략은 간단한 동기화 도구를 통합하여 고급 언어 구조물을 제공하는 것이다. 이번엔 고급 언어 구조물 중 하나인 **모니터(monitor)**를 살펴보자!!

#### 6.7.1 모니터 사용법 (Monitor Usage)

추상화된 데이터 형(abstract data type, ADT)은 데이터와 이 데이터를 조작하는 함수들의 집합을 하나의 단위로 묶어 보호한다. 이때 함수의 구현은 ADT의 특정한 구현과 독립적이다.

**모니터 형은 모니터 내부에서 프로그래머가 정의한 상호 배제가 보장되는 일련의 연산자 집합을 포함하는 ADT이다.** 모니터 형은 인스턴스의 상태를 정의하는 변수들과 이를 조작할 수 있는 프로시저 또는 함수들의 본체도 같이 포함하고 있다. 

따라서 모니터 내에 정의된 함수만이 오직 모니터 내에 지역적으로 선언된 변수들과 형식 매개변수들에만 접근할 수 있다. (객체의 캡슐화와 매우 비슷한 구조)

아래의 그림은 모니터의 개략도이다.

<img src="/assets/computer-science/synchronization-tools-8.png" style="width:50%">


**모니터 구조물은 모니터 안에 항상 하나의 프로세스만이 활성화되도록 보장해 준다.** 그러나 지금까지 정의한 모니터 구조물은 어떤 동기화 기법을 모델링하는 데에는 충분한 능력을 제공하지 않는다.

condition이라는 구조물로 동기화 기법들을 제공해 보자. 자신의 동기화 기법을 작성할 필요가 있는 프로그래머는 하나 이상의 condition 형의 변수를 정의할 수 있다.

```c
condition x,y;
```

이 condition 형 변수에 호출될 수 있는 연산은 오직 wait()와 signal()이다. x.wait()은 이 연산을 호출한 프로세스는 다른 프로세스가 x.signal()을 호출할 때까지 일시 중지 되어야 한다는 것을 의미한다.

Java와 C# 등을 포함한 많은 프로그래밍 언어들은 이 절에서 설명한 모니터의 개념을 편입 시켰다.

#### 6.7.2 세마포를 이용한 모니터의 구현 (Implementing a Monitor Using Semaphores)

각 모니터마다 mutex라는 이진 세마포가 정의되고 그 초기 값은 1이다. 프로세스는 모니터로 들어가기 전에 wait(mutex)를 실행하고 모니터를 나온 후에 signal(mutex)을 실행해야 한다.

모니터 구현 시 signal-and-wait 기법을 사용한다. Signaling 프로세스는 실행 재개되는 프로세스가 모니터를 떠나는지 아니면 wait() 할 때까지 그 자신이 다시 기다려야 하므로 next라는 이진 세마포가 추가로 필요하게 되고 0으로 초기화된다.

signaling 프로세스는 자신을 중단시키기 위해 next를 사용할 수 있다. 정수형 변수 next_count에서도 next에서 일시 중지 되는 프로세스의 개수를 세기 위해 제공된다. 따라서 각 외부 프로시저 F는 아래로 대체된다.

```c
wait(mutex);

    ...
    body of F
    ...

if (next_count > 0) // 일시 중지된 프로세스가 존재한다면
    signal(next);   // next 프로세스를 실행 재개한다.
else
    signal(mutex);
```

위와 같이 알고리즘을 작성한다면 Mutual Exclusion은 보장된다.

#### 6.7.3 모니터 내에서 프로세스 수행 재개 (Resuming Processes within a Monitor)

"조건 변수 x에 여러 프로세스가 일시 중지 되어 있을 때 어떠한 프로세스를 수행 재개시킬 것인가??"에 대해서 논의해 보면 가장 간단한 방법은 FCFS 순이다. 하지만 많은 경우 이러한 간단한 스케줄링 기법은 충분하지 않다.

이를 위해서 아래와 같은 형식의 **conditional-wait 구조**를 사용할 수 있다. 이 구조물은 다음과 같은 형태를 가진다.

`x.wait(c);`

여기서 c는 정수이고, 우선순위 번호(priority number)라고 불리며 일시 중지 되는 프로세스의 이름과 함께 저장된다. 즉, x.signal()이 수행되면 가장 작은 우선순위 번호를 가진 프로세스가 다음번에 수행 재개 된다.

이 새로운 기법을 설명하기 위해 아래와 같은 구조를 가진 ADT인 ResourceAllocator 모니터를 예로든다.

<img src="/assets/computer-science/synchronization-tools-9.png" style="width:80%">

이 모니터는 한 개의 자원을 여러 프로세스 사이에 할당해 준다. 각 프로세스는 자원을 할당받기를 원하면 그 자원을 사용할 최대 시간을 지정한다. 모니터는 이 중 가장 적은 시간을 희망한 프로세스에 자원을 할당해 준다. 이 자원을 액세스하려는 프로세스는 아래의 순서를 따라야 한다.

```c
R.acquire(t);   // R은 ResourceAllocator형 인스턴스이다.

    ...
    access the resource;
    ...

R.release();
```

time을 사용해서 한 개의 자원을 접근하는데 무리없이 동작하는 것 처럼 보이지만 사실 다음과 같은 문제가 발생할 수 있다.

- 프로세스가 자원에 대한 허락을 받지 않고 자원을 액세스 할 경우
- 프로세스가 자원에 대한 허락을 받은 다음 그 자원을 방출하지 않을 경우
- 프로세스가 자원에 대한 허락을 받지 않았는데도 그 자원을 방출할 경우
- 프로세스가 자원에 대한 허락을 받은 다음 방출하지 않은 상태에서 또 그 자원을 요청할 경우

사실 위와 동일한 문제들은 모니터를 사용할 때 뿐만 아니라 세마포를 사용할 때도 동일하게 발생한다. 

이 문제를 해결하기 위해서 **자원 액세스 연산 자체를 ResourceAllocator 모니터 내부에 두는 방법**이 있고, 프로세스들이 올바른 순서를 지키도록 보장하기 위해서 **ResourceAllocator 모니터와 모니터가 관리하는 자원을 사용하는 모든 프로그램을 검사**하는 방법이 있다.

---

### 6.8 라이브니스 (Liveness)

---

**라이브니스는 프로세스가 실행 수명주기 동안 진행되는 것을 보장하기 위해 시스템이 충족해야 하는 일련의 속성을 말한다.** 즉, 프로세스가 lock을 얻기 위해 무기한 대기하는 것은 "라이브니스 실패"의 한 예이다.

다양한 형태의 라이브니스 실패가 존재한다. 그러나 모두 성능과 응답성이 나쁜 것이 특징이다. 라이브니스 실패의 매우 간단한 예는 무한 루프이다. 즉, Mutex 락 및 Semaphore를 사용하여 상호 배제를 제공하려는 노력은 종종 병행 프로그래밍에서 이러한 실패로 이어질 수 있다.

#### 6.8.1 교착 상태 (Deadlock)

대기 큐를 가진 Semaphore 구현은 두 개 이상의 프로세스들이, 오로지 대기 중인 프로세스들 중 하나에 의해서만 야기될 수 있는 `signal()연산`를 무한정 기다리는 상황이 발생할 수 있다. 이런 상태에 도달했을 때, 이들 프로세스들을 **교착 상태(deadlock)**라고 한다.

**한 집합 내의 모든 프로세스가 그 집합 내의 다른 프로세스만이 유발할 수 있는 이벤트를 기다릴 때, 이 프로세스들의 집합이 교착 상태에 있다고 말한다.** 우리가 여기서 주로 관심을 두고 있는 "이벤트"들은 mutex 락과 Sempahore 같은 자원의 획득과 방출이다.

#### 6.8.2 우선순위 역전 (Priority Inversion)

**높은 우선순위 프로세스가 현재 낮은 우선순위 프로세스 또는 연속된 낮은 우선순위 프로세스들에 의해 접근되고 있는 커널 데이터를 읽거나 변경할 필요가 있을 때 스케줄링의 어려움이 생기게 된다.**

통상 커널데이터는 락에 의해 보호되기 때문에 낮은 우선순위 프로세스가 자원의 사용을 마칠 때까지 높은 우선순위 프로세스가 기다려야 한다. 낮은 우선순위 프로세스가 또 다른 높은 우선순위 프로세스에 의해 선점되는 경우에 상황은 더욱 복잡해진다. 이러한 경우 낮은 우선순위 프로세스는 계속 기다려야만 한다. 이 라이브니스 문제는 **우선순위 역전(priority inversion)**문제로 알려져 있다.

통상 우선순위 역전 문제는 **우선순위 상속 프로토콜(priority-inheritance protocol)**을 구현하여 해결한다. 우선순위 상속 프로토콜의 하나의 예시로서, 더 높은 우선순위 프로세스가 필요로 하는 자원에 접근하는 모든 프로세스는 문제가 된 자원의 사용이 끝날 때까지 더 높은 우선순위를 상속받는다. 자원 사용이 끝나면 원래 우선순위로 되돌아간다.

---

### 6.9 평가(Evaluation)

---

일반적으로 하드웨어 솔루션은 매우 낮은 수준으로 간주하며 mutex 락과 같은 다른 동기화 도구를 구성하기 위한 기초로 사용된다.

그러나 최근 lock overhead 없이 경쟁 조건으로부터 보호하는 **lock free algorithm**을 구현하기 위해 CAS(Compare And Swap) 명령을 사용하는데 중점을 두고 있다. 이러한 락 없는 솔루션은 오버헤드가 낮고 확장성이 있기 때문에 인기를 얻고 있지만 알고리즘 자체는 개발 및 테스트가 어려운 경우가 많다.

CAS 기반 접근 방식은 낙관적인 접근법으로 간주되고, locking 기반 접근 방식은 비관적 전략으로 간주된다.

경쟁 조건을 해결하기 위한 기법의 선택은 시스템 성능에도 큰 영향을 줄 수 있다.

모니터와 조건 변수와 같은 고급 도구의 매력은 단순성과 사용 편의성으로부터 나온다.

다행스럽게도 병행 프로그래밍의 요구 사항을 해결하는 확장 가능하고 효율적인 도구를 개발하기 위한 많은 연구가 진행되고 있다. 이는 다음과 같다.
- 더 효율적인 코드를 생성하는 컴파일러 설계
- 병행 프로그래밍을 지원하는 언어 개발
- 기존 라이브러리 및 API의 성능 향상

다음 7장에서는 개발자가 사용할 수 있는 다양한 운영체제 및 API가 이 장에서 제시된 동기화 도구를 어떻게 구현하는지 알아보도록 하자.

---

**6장에서는 프로세스 중심의 동기화에 대해서 주로 다루었다. 나는 프로세스 동기화 뿐만 아니라 멀티 스레드를 처리하는 다중 코어 시스템에서 어떻게 스레드간에 동기화를 유지하는지도 궁금하기 때문에 추후 더 공부할 계획이다.**

**6장은 여기서 마치도록 하고 7장인 Synchonization Examples에서 뵙도록 하겠다. 꾸벅~**