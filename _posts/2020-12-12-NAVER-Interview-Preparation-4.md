---
layout: post
title:  "[기술 면접 준비 - 4일차] 자료구조 & Spring MVC"
date:   2020-12-12
author: Green Frog Developer
categories: Interview
tags: Data-Structure Spring-MVC
---

# 자료구조 (Data Structure)

### Array 란 무엇인가요?

**Array는 가장 기본적인 자료 구조이며, 논리적 저장 순서와 물리적 저장 순서가 일치 합니다.** 원소의 인덱스 값을 알고 있으면 검색에 O(1)이 걸리지만 삽입이나 삭제등이 필요한 경우 원소들을 shift해줘야 하기 때문에 O(n)의 시간이 걸립니다.

### Linked List

**모든 노드들을 링크로 연결한 리스트를 말합니다.** 이 경우 삭제와 삽입은 O(1)만에 해결할 수 있지만, 원하는 위치에 원소를 삽입하거나 삭제하는 경우 이를 Search하는 시간이 필요하므로 O(n)의 시간이 걸립니다.

### 선형 자료구조와 비선형 자료구조의 차이는 무엇인가요?

**선형 자료구조는 데이터 요소들이 저장되어 있는 모습을 표현했을 때 직선이고, 비선형 자료구조는 데이터 요소들이 저장되어 있는 모습을 표현했을 때 직선이 아닌 것을 의미합니다.**

선형 자료구조의 대표적인 예는 Array, Queue 등이 있고, 비선형 자료구조의 대표적인 예는 Tree, Graph등이 있습니다.

### Stack

선형 자료구조의 일종으로 **Last In First Out(LIFO)**의 특징을 가지고 있습니다.

### Queue

선형 자료구조의 일종으로 **First In First Out(FIFO)**의 특징을 가지고 있습니다.

### Tree

**트리는 비선형 자료구조로서 node들과 이를 연결하는 edge들로 구성되어 있습니다.**

- 트리는 하나의 루트 노드를 갖는다.
- 트리에는 싸이클이 존재할 수 없습니다.

### Binary Tree

**이진 트리란 각 노드가 최대 두개의 자식을 갖는 트리를 말합니다.** 

- 완전 이진 트리: 마지막 레벨을 제외하고 모든 레벨이 완전히 채워져 있고 마지막 레벨의 노드가 왼쪽에서 오른쪽으로 채워지는 이진 트리
- 진 이진 트리 : 모든 노드가 0개 또는 2개의 자식 노드를 갖는 이진 트리
- 포화 이진 트리 : 모든 레벨에 노드과 꽉 찬 이진 트리를 가르킨다.

### Binary Search Tree

중복된 데이터를 갖는 노드가 없다는 가정하에 말씀드리겠습니다. **부모 노드가 왼쪽 자식보다 크고 오른쪽 자식보다는 작은 규칙을 만족하는 이진 트리를 Binary Search Tree** 라고 합니다. Binary Search Tree에서 탐색, 삽입, 삭제 연산의 시간 복잡도는 O(h) 즉, 높이에 비례합니다. 하지만 완전 이진 트리인 경우에 O(log n)으로 표현할 수도 있습니다.

### Heap

**힙(heap)은 최대 또는 최소를 빠르게 찾아내기 위해 만들어진 완전이진트리를 기본으로 한 자료구조로서 부모노드와 자식 노드간에 대소관계가 성립한다.**

힙의 시간복잡도는 최대 및 최소를 검색 시간복잡도는 O(1)입니다. 하지만 heapify의 시간 복잡도는 O(log N)이기 때문에 고민해 봄직 하다.

### Heap과 BST의 차이는 무엇인가요?

**힙은 최대 및 최소를 찾는데 (O(1)) 좋지만, BST는 모든 원소들을 찾는데 (O(log n)) 유용합니다.**

### Red Black Tree

**Red Black Tree란 자가 균형 이진 탐색 트리의 한 종류로써 이진 탐색 트리의 단점인 편향성을 색깔을 통해서 보완하기 위한 자료구조이다.** 검색, 삽입, 삭제의 시간 복잡도는 O(log n)이다.

### Hash Table

**Hash Table이란, 임의의 길이를 가진 키를 고정된 길이의 Hash Code로 변환시켜서 저장하는 자료구조를 의미**하구요 연산시 시간복잡도는 O(1)입니다. 

### Hash Function

**해시란 임의의 길이를 가진 데이터를 고정된 길이를 가진 데이터로 매핑 하는 함수를 해시 함수**라고 하며 이로부터 얻어진 값을 해시 값, 해시 코드, 짧게 말해서 해시라고도 합니다. 항상 Collision 즉, 서로 다른 두 개의 키가 같은 인덱스로 hashing되는 경우를 조심하는 알고리즘을 구현해야 한다.

### Graph란 무엇인가요?

**Node들과 이를 연결하는 Edge들을 모아 놓은 자료구조이다.** 방향 및 비방향 그래프 모두 존재하며, 사이클 및 self-loop 가 존재해도 상관 없다.

그래프의 구현 방법에는 인접 행렬과, 인접 리스트 등이 존재한다.

### 그래프 탐색에는 어떠한 방법이 존재하나요?

그래프는 따로 규칙이 존재하지 않기 때문에 **모든 정점을 탐색**해야만 하구요, **특정 정점을 기준으로 넓게 탐색하기 전에 깊게 탐색하는 방법인 DFS**와 **깊게 탐색하기 전에 넓게 탐색하는 방법인 BFS**가 존재합니다.

DFS와 BFS를 인접리스트로 구현할 경우에 시간복잡도는 O(V+E) 이구요, 인접 행렬로써 구현할 경우에 시간복잡도는 O(V^2)입니다.

### Minimum Spanning Tree란 무엇인가요?

**Spanning Tree란 그래프 G의 모든 V가 Cycle 없이 연결된 형태를 말하구요, 여러 Spanning Tree 중 Edge의 가중치의 합이 가장 작은 것을 Minimum Spanning Tree**라고 합니다.

### Minimum Spanning Tree를 구하는 방법에는 무엇이 있나요??

**MST를 구하는 방법 중 탐욕적인 방법으로 Edge 값이 가장 작은 것부터 탐색하는 Kruskal Algorithm과 Prim Algorithm이 존재합니다.**

Kruskal의 시간 복잡도는 O(ElogE)이고, Prim Algorithm의 시간 복잡도는 O(ElogV)이다.

---

# Spring MVC

### MVC 패턴이란 무엇인가요?

**MVC 패턴이란 웹 어플리케이션에서 자주 사용하는 디자인 패턴으로 시스템을 크게 Model, View, Controller로 나누어 모듈간에 결합도를 낮추고 응집도를 높이기 위해 사용하는 패턴**입니다.

- **Controller란 요청을 받아서 모델, 뷰와의 상호작용을 통해 적절한 응답을 제공해주는 중계자**의 역할을 합니다.
- **모델은 시스템에서 사용하는 도메인 모델** 등을 의미합니다.
- **뷰는 사용자에게 제공하는 화면** 등을 의미합니다.

### Spring MVC의 구조에 대해서 설명해 주실 수 있나요?

**Spring MVC의 시작은 DispatcherServlet이라고 할 수 있습니다. DispatcherServlet은 사용자의 HTTP 요청을 처리하기 위하여 등록된 핸들러로 디스패치하여 매핑 및 예외 처리 기능을 제공**합니다.

**DispatcherServlet으로 요청이 들어왔을 때 웹 서버의 대략적인 과정은 다음과 같습니다.**

1. DispatcherServlet으로 HTTP Request가 들어온다.
2. DispatcherServlet은 HandlerMapping을 통해서 요청 URL 등의 정보와 맵핑되는 적절한 Controller를 검색한다.
3. DispatcherServlet은 HandlerAdapter를 통해 HandlerMapping에서 결정된 핸들러 정보를 통해서 컨트롤러를 호출한다.
4. Controller는 요청을 받아 적절한 비즈니스 로직을 태운다.
5. 비즈니스 로직이 끝난 후 사용자에게 응답 할 View의 이름을 DispatcherServlet을 통해서 View Resolver로 전달한다.
6. View Resolver는 View name에 해당하는 view를 검색한다. (prefix, suffix, resource path)
7. 응답 받을 View가 존재한다면 DispatcherServlet으로 View를 가져오고 사용자에게 응답한다.

### Servlet이란 무엇인가요?

**HTTP 서블릿이란 자바단에서 클라이언트의 요청을 처리하고 응답을 제공하는 객체라고 할 수 있습니다.** 서블릿 하나당 스레드 하나가 할당 된다.

### Spring MVC를 위한 필수 설정

**Web Deployment Descriptor(web.xml)이란 클라이언트가 어떤 URL을 요청했을 때 어떤 Servlet 파일을 실행시킬 것인지를 매핑해놓은 파일이다.**

- Servlet 설정
- ContextConfigLocation 설정 : Context Loader가 load할 수 있는 설정 파일의 위치 명시
- Filter 설정
- SpringSecurityFilterChain

**Spring MVC Configuration Files**

- **dispatcher-servlet.xml**
    - Controller 관련
    - mvc:annotation-driven 설정
    - Component-scan 관련
    - 정적인 data 위치 mapping
    - ViewResolver 관련 (prefix, suffix, ...)

- **applicationContext.xml**
    - DataSource 주입
    - Properties 등록
    - tx:annotation-driven 설정 (어노테이션 기반 트랜잭션 동작 설정)
    - Session Factory 등록 및 TransactionManager 설정

### Spring Boot MVC와 Spring MVC의 차이

**Spring Boot기반 MVC는 기존에 Spring MVC에서 필수적으로 설정해야 하는 톰캣설정 및 web.xml에 관련된 설정을 스프링 부트의 내부모듈에 의해서 구동시 자동설정** 해준다.

**@SpringBootApplication 어노테이션은 @ComponentScan, @EnableAutoConfiguration, @Configuration 에노테이션** 등으로 이루어져 있는데 이게 핵심이다.

HttpMessageConverter, ViewResolver, Resource, jar 등, 톰캣 설정 및 web.xml에 관련된 설정을 스프링 부트의 내부모듈에 의해서 구동시 자동설정 해준다.

### Spring Boot 설정 관련 이슈는 있었는가?

**Spring Boot MVC의 자동 설정에 가장 중요한 클래스는 WebMvcAutoConfiguration**인데 위에  @ConditionalOnMissingBean(WebMvcConfigurationSupport.class)와 같은 애노테이션이 있었습니다.

즉, WebMvcConfigurationSupport Bean이 존재하면 WebMvcAutoConfiguration bean이 생성되지 않아 web mvc 자동 설정이 되지 않는다는 이야기 입니다.

**우리는 Spring Boot MVC에 설정을 추가하기 위해서는 WebMvcConfigurer를 상속받아서 빈으로 만들어야 합니다.**

이 설정이 추가되는 코드는 **DelegatingWebMvcConfiguration** 클래스의 로직에서 WebMvcConfigurer Bean들을 주입받고 이를 Spring MVC 설정에 추가합니다.

### Spring MVC에서 Spring Boot Starter가 해주는 것들은 무엇인가요??

프로젝트의 pom.xml을 보면 일단 가져오는 **프로젝트 의존성 부터 차이**가 존재한다. **spring mvc는 spring 프로젝트의 spring-webmvc라는 의존성**을 가져오고, **spring boot mvc는 spring boot 프로젝트의 spring-boot-starter-web**을 가져온다.

spring boot starter web은 기본적인 spring mvc를 사용하기 위한 많은 설정을 자동화 해주고 애노테이션 기반으로 프로젝트의 설정을 할 수 있도록 한다.

**spring boot web mvc가 해주는 설정들은 다음과 같다.**

- 내장된 웹 애플리케이션 서버(Tomcat, Jetty, Undertow)
- 의존성을 손쉽게 관리할 수 있는 Project Object Model
- 설정의 표준화와 자동화
    - Spring Framework의 수 많은 XML 기반의 설정을 Spring Boot의 자동 설정으로 제거하거나 properties 형식과 JavaConfig 형식으로 대체
    - 각종 Filter, ViewResolver, DispatcherServlet설정, ComponentScan, Annoation기반 설정 등을 자동으로 설정해준다.

### Spring MVC와 WAS의 관계

**클라이언트는 tomcat에게 요청을 보내고 tomcat은 URL 및 기타 정보에 따라(web.xml) 처리를 위해 요청을 보낼 서블릿**을 결정합니다. spring web mvc에서 이 서블릿은 대부분 DispatcherServlet을 의미하고 이를 통해 Controller단으로 들어와 비즈니스 로직을 태운 뒤 적절한 모델을 만들어 뷰에 랜더링한 후 톰캣에게 다시 전송되고 톰캣은 클라이언트에게 다시 전송합니다.

### Executable Jar File

**Spring Boot maven plugin을 사용하면 Spring Boot loader가 Tomcat이 내재된 실행가능한 Jar파일을 생성**해준다. 우리는 이를 통해 jar 파일만 실행시키면 되는 것이니 스프링을 구성할 때 매우 편리하다.

### Spring MVC에서 Exception 이 발생하면 어떻게 되는가?

**request mapping 또는 request handler로 부터 예외가 발생하면 DispatcherServlet은 HandlerExceptionResolver구현체가 예외를 처리해 적절한 응답을 클라이언트에게 보낸다.** 나는 Jackson 사용간에 Deserialize에서 에러 발생을 경험해 보았기 때문에 **DefaultHandlerExceptionResolver**가 400번 상태 코드를 맵핑해서 사용자에게 응답해주었다.

### Spring Message Converter는 어떻게 동작하는가?

**Spring Boot는 HttpMessageConvertersAutoConfiguration을 이용해서 Converter 서비스도 자동 구성해줌으로 이를 이용하면 된다. Json Parsing을 위해 MappingJackson2HttpMessageConverter가 사용된다.**

