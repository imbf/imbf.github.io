---
layout: post
title: "[실무 면접 준비 - 6] Pinpoint Webhook Contribution"
date: 2021-03-05
author: Green Frog Developer
categories: Interview
tags: Pinpoint Webhook Alarm Batch Web Monitoring
---

---

## Pinpoint

---

### Pinpoint는 무엇인가요?

**Pinpoint란 대규모 분산 시스템을 위한 APM(Application Performance Management)도구 입니다.**

-  **ServerMap**을 통하여 컴포넌트간에 연결되는 방식들을 시각화하고 각 컴포넌트를 분석합니다. 
- **Realtime Active Thread Chart**를 통하여 실시간으로 어플리케이션 내에서 활동하고 있는 스레드를 모니터링 합니다.
- **Request/Response Scatter Chart**를 통하여 시간 경과에 따른 요청 수 및 응답 패턴을 시각화하여 잠재적인 문제를 식별합니다.
- **CallStack**을 통하여 분산 환경에서 모든 트랜잭션에 대해 코드레벨 수준의 가시성을 제공하여 병목 현상과 실패 지점을 식별합니다.
- **Inspector**을 통하여 어플리케이션에 추가적인 정보를 볼 수 있습니다. (CPU 사용량, Memory/Grabage Collection, TPS, JVM Arguemnts, ...)



> ### Pinpoint Architecture
>
> <img src="/assets/interview/naver-practical-interview-preparation6-1.png" style="width:100%">




### Pinpoint에 어떠한 기여를 하셨나요?

**저는 Pinpoint Batch 모듈에 Webhook 기능을 추가하는 작업과 관련 도메인을 재설계 하는 작업을 진행하였습니다. 간단히 말해서 Batch가 application 상태를 주기적으로 체크하여 수치가 Threshold를 초과할 경우 클라이언트에게 webhook을 전송하는 기능을 개발하였습니다.**



### 웹훅(Webhook)이란 무엇인가요?

**웹훅(Webhook)이란 서버측에서 특정 이벤트가 발생할 경우 클라이언트 측으로 알림을 준다던가 관련 정보를 전송해주는 것을 의미합니다.** 저희는 알람 및 정보 전송을 위해서 HTTP Protocol을 사용했습니다.

>### 웹과 인터넷
>
>**인터넷(Internet)**은 컴퓨터로 연결하여 TCP/IP 프로토콜을 이용해 정보를 주고받는 컴퓨터 네트워크입니다.
>
>**웹(WEB)**은 인터넷에 연결된 컴퓨터를 통해 사람들이 정보를 공유할 수 있는 전 세계적인 공간을 의미합니다.
>
>-출처: wiki-



### 웹훅 기능을 기여하는데 얼마나 걸리셨죠??

**작년 8월에 시작해서 올해 1월 말까지 진행했으니 대략 6개월 정도 걸렸던 것 같습니다.** 개발 기간이 이정도까지 오래 걸리리라고는 상상도 못했지만 Pinpoint가 체크하는 Metric이 엄청 많다 보니 확장성 있고 유지보수성이 좋은 Webhook Json Spec을 정하는데 오래 걸렸고, 기존의 Alarm 도메인 설계 자체도 적절하지 못한 것 같아서 이를 리팩토링 하는데 시간이 오래걸렸습니다. 또한 머지가 되야하는 시점에 Batch가 원래 Web 모듈 내에 존재 했었는데 독립적인 모듈로 분리가 되면서 저도 이를 반영해야 했기 때문에 더 오래걸렸던 것 같습니다. **하지만 오래 걸린 만큼 스펙과 구현에 더 집중할 수 있어서 더 확장성 있고 유지보수성이 좋은 개발을 할 수 있었습니다.**



### 웹훅 기능을 개발하면서 어려운 점이 있으셨나요??

Webhook을 개발하면서 가장 어려웠던 점은 Pinpoint에서 체크하는 Metric들의 타입이 너무 다르다는 것이었습니다. 추상 클래스인 AlarmChecker의 구현체에 따라서 Metric 값의 타입은 Integer, Integer 배열, 특정 객체 배열 등 다양한 타입이 존재했습니다. 기존 SMS와 Email을 통해서 알람을 보낼 때는 수집된 값의 형식이 달라도 전송하기 전에 Message Template을 통해 특정 텍스트 형식을 만들어 보내면 됬었지만, Webhook 기능은 Pinpoint와 다른 어플리케이션간에 확장성 제공이 목적이었기 때문에 Metric 별로 특정 텍스트 형식을 만들어 보내기에는 사용자의 입장에서 자유도가 떨어지고 특정 값이 필요할 경우 사용자가 다시 문자열 처리를 해야만하는 책임을 주는 상황이었습니다.

**이러한 이슈를 해결하기 위해 Metric 별로 수집하는 값의 타입을 모두 추상화할 수 있는 DTO 구조를 설계 및 구현했고, 수집된 Metric 값의 형식에 기반하여 도메인 구조를 재설계 및 구현함으로써 문제를 해결할 수 있었습니다.**



### 핀포인트를 개발하시면서 얻은 점은 무엇인가요??

핀포인트 컨트리뷰션을 통해 정말 많은 부분을 배웠지만 그래도 가장 인상깊게 얻었던 점은 세 가지입니다.

- 첫 번째로는 핀포인트가 여러 인스턴스로 구성되어 있는 프로젝트인 만큼 전체 시스템 아키텍처를 볼 수 있는 능력이 향상되습니다.
- 두 번째로는 평소에 객체지향에 매우 관심이 많은데 모니터링 도메인에서 적용된 객체지향에 대해서도 배울 수 있었습니다.
- 세 번째로는 오픈 소스 컨트리뷰션에 대해서 자신감을 얻을 수 있었습니다. 오픈 소스에 컨트리뷰션 하는게 크게 어렵지않고 '내가 기여한 코드를 통해서 나 뿐만 아니라 오픈 소스를 사용하는 전 세계의 수 많은 개발자들이 편리해 질 수 있겠구나' 라는 점을 깨달았구요 '다음번에도 오픈소스에 기여할 수 있겠다'라는 자신감이 생겼습니다. 실제로 인턴 기간에 Spring 프로젝트의 Data Redis에 오류가 존재해서 이를 컨트리뷰션해 머지가 된 경험을 하였습니다.



### Pinpoint의 Alarm Batch 구조에 대해서 설명해주실 수 있나요??

**Pinpoint의 Alarm Batch구조는 JobLauncher에서 정해진 시각마다 AlarmJob이 실행되는데 AlarmJob은 하나의 AlarmStep으로 이루어져 있고, AlarmStep은 Chunk기반의 Processing인 AlarmReader, AlarmProcessor, AlarmWriter로 구성되어 있습니다.**

AlarmReader는 HBase에서 어플리케이션의 Metric 데이터를 가져오고 AlarmProcessor가 이를 가공해 지정한 Threshold가 넘는지 확인합니다. 그 후 AlarmWriter는 Threshold가 넘는 Metric값과 관련 어플리케이션 정보를 User에게 email, sms, webhook 등을 통해 전송하는 구조를 가지고 있습니다.



### HBase란 무엇인가요?

**HBase란 HDFS(Hadoop Distributed File System)위에서 만들어진 분산 컬럼 기반의 NoSQL 데이터베이스입니다.** HBase에 저장된 대용량 데이터에 대해 강한 일관성과 실시간에 가까운 연산을 제공합니다.



### 분산형 데이터베이스는 무엇인가요?

**분산형 데이터베이스란 데이터를 여러 인스턴스에 분산해서 저장하고 관리함으로써 높은 신뢰성과 성능을 제공하는데 효과적인 데이터베이스를 분산형 데이터베이스라고 합니다.** 대표적으로 Apache HBase, Apache Cassandra 등이 존재합니다.



### DB 하위 호환 기능은 어떻게 구현 하셨나요?

webhook 기능을 개발하면서 DB 테이블에 컬럼이 하나 추가될 필요가 있었습니다. 컬럼이 추가되지 않으면 webhook 기능을 사용하지 못하기 때문에 이를 사용하기 위해서는 기존 사용자가 직접 DB에 들어가서 컬럼을 추가하는 쿼리문을 실행 해야만 했었습니다.

그러나 웹훅 기능을 사용하지는 않지만 다른 기능 때문에 최근 Release를 도입하는 기존 사용자에게 "DB에 들어가서 Column을 추가해! 그렇지 않으면 알람 기능 자체를 사용하지 못할거야!" 라고 말하는 것은 적절하지 않다고 생각했기 때문에 웹훅을 사용하지 않는 기존 사용자를 위해서 하위 호환 기능을 구현할 필요가 있었습니다.

**하위 호환을 위해 webhook.enable이라는 property를 만들었구요 이를 통해 웹훅 기능을 사용하는 유저와 사용하지 않는 유저를 구분했습니다. 즉, webhook.enable이 true라고 설정되어 있다면 컬럼이 추가된 쿼리문이 DB에 전송되고 webhook이 동작하며, false라고 설정되어 있다면 컬럼이 추가되지 않은 쿼리문이 날라가고 webhook을 비활성화 하면서 하위 호환을 유지할 수 있었습니다.** 

**이를 통해 평소에 전혀 경험해보지 못했던 데이터베이스 하위 호환을 경험할 수 있어서 즐겁고 색다른 경험이었습니다.**



### 어떻게 Alarm 도메인을 재설계 했는지 자세히 알 수 있을까요?

기존 Alarm 도메인은 단순한 ErrorCount나 API TotalCount인 Integer 값을 checking하는 AlarmChecker와 Agent가 붙여진 어플리케이션의 CPU나 Heap사용량 등을 checking하는 AgentChecker로 나뉩니다. 

이러한 Checker들이 검색하는 값의 타입이 각각 달라서 Webhook JSON Data Spec을 만드는데 어려움이 생겼고 이를 추상하기 위해서 Checker들이 검색하는 Metric 값의 타입을 기준으로 관련 도메인을 추상화 했고 리턴하는 Data 값을 추상화 할 수 있는 DTO를 설계하였습니다.

**Metric을 담을 DTO를 구현하기 위해서 어떠한 메서드 정의나 구현이 없는 오직 추상화만을 위한 마커 인터페이스를 만들었구요 이를 상속하는 AlarmCheckerDTO와 AgentCheckerDTO를 만들었습니다. 제네릭을 사용해 Metric별로 객체의 타입이 매우 다른 AgentCheckerDTO를 만들 수 있었고 이를 통해 더 확장성 있고 유지보수성이 좋은 Webhook JSON Data Spec을 만들 수 있었습니다.**

