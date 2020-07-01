---
layout: post
title:  "Spring Security + JWT를 통해 프로젝트에 인증 구현하기"
date:   2020-06-29
author: Green Frog Developer
categories: Spring
tags: Spring Security JWT 
---

Spring Security와 JWT를 활용해서 프로젝트에 인증을 어떻게 구현했는지에 대해서 포스팅 하려고 한다.

토큰을 사용한 인증은 처음이기도 하고, Google OAuth를 함께 사용하는 바람에... 개념이 해깔려 며칠간 계속 해맸던 것 같다.

이제서야 조금 감을 잡으며 프로젝트에 인증 구현을 마쳤는데.. 아직 구현되지 않은 기능들이 매우 많다 ㅎㅎ 언능 구현해야 겠다!!!

**서론이 길었다!!! 지금부터 Spring Security란 무엇이고, Spring Security와 JWT를 사용해서 어떻게 인증을 구현했는지에 대해서 포스팅 하겠다.**

---

## Spring Security란?

---

**Spring Security란 인증과, 권한부여, 일반적인 공격에 대한 보호의 기능을 제공하는 프레임워크이다.** 즉, Spring Security를 사용하면 보안 관련 처리를 자체적으로 구현 할
필요 없이 쉽고 안전하게 필요한 기능을 구현할 수 있다.

Spring Security의 Servlet 보안 지원은 **Servlet Filter(이하 Filter)**를 기반으로 한다. 클라이언트가 어플리케이션으로 request를 보내면, Container는 요청 URI의 경로에 따라 어떤 Filter와 어떤 Servlet을 적용할 것인지 결정한다.

Spring은 여러 Filter중 Servlet Container lifecycle과 ApplicationContext사이에 bridging할 수 있는 `DelegatingFilterProxy`라는 Filter를 제공한다.

Spring Security의 Servlet 보안 지원은 `DelegatingFilterProxy`가 감싸고 있는 **`FilterChainProxy`**에 의해 수행되며, `FilterChainproxy`는 **`SecurityFilterChain`**을 통해 많은 쟉업을 **Security Filter** 인스턴스에 위임한다.

위 과정을 그림으로 나타내면 다음과 같다.

<img src="/assets/spring/Spring-Security-With-JWT-1.png" style="width:70%">

SecurityFilterChain은 스프링에서 보안과 관련된 여러 Security Filter List를 갖고 있는 객체로 Security Filter List를 순회하면서 필터링을 실시한다.

SecurityFilterChain에 존재하는 Security Filter순서는 다음과 같다.

1. ChannelProcessingFilter
2. ConcurrentSessionFilter
3. WebAsyncManageIntegrationFilter
4. SecurityContextPersistenceFilter
5. HeaderWriterFilter
6. CorsFilter
7. CsrfFilter
8. LogoutFilter
9. OAuth2AuthorizationRequestRedirectFilter
10. Saml2WebSsoAuthenticationRequestFilter
11. X509AuthenticationFilter
12. AbstractPreAuthenticatedProcessingFilter
13. CasAuthenticationFilter
14. OAuth2LoginAuthenticationFilter
15. Saml2WebSsoAuthenticationFilter
16. **UsernamePasswordAuthenticationFilter** 
17. ConcurrentSessionFilter

**... (총 33개의 Spring Security Filter가 존재한다.)**

위의 Security Filter중 Security Authentication Filter는 `AuthenticationManager`를 통해 인증을 수행한다. **AuthenticationManager는 어떻게 Spring Security의 필터들이 인증을 수행하는지에 대한 명세를 정의해 놓은 인터페이스이다.**

이 `AuthenticationManager`는 일반적으로 **`ProviderManager`**로 구현되며, `ProviderManager`는 여러 `AuthenticationProvider`에 인증을 위임한다.

`ProviderManager`에 설정된 `AuthenticationProvider`중 어느 것도 성공적으로 인증을 수행할 수 없다면, 인증은 실패할 것이고 알맞는 예외가 ProviderManager에 건내질 것이다.

**여러 `AuthenticationProvider`중 하나라도 인증에 성공한다면 `ProviderManager`에게 인증된 `Authentication`객체를 반환하고 이는 event로 `AuthenticationFilter`에 전송된다.**

**`AuthenticationFilter`는 `SecuritycontextHolder`의 `SecurityContext`에 인증된 Authentication을 저장할 수 있도록 한다.**

아래의 그림은 **Username and Password 인증 방식의 아키텍처**이다.

<img src="/assets/spring/Spring-Security-With-JWT-2.png" style="width:100%">

위 그림의 `AuthenticationFilter`의 역할은 `UsernamePasswordAuthenticationFilter`가 수행한다.

Spring Security를 전체적으로 설명하기에는 한 포스팅으로 부족하기 때문에 여기까지만 설명하도록 하고, 이제 나의 프로젝트에 어떻게 Spring Security + JWT를 적용했는지에 대해서 알아보도록 하겠다.

---

## Spring Security + JWT를 프로젝트에 어떻게 적용 했는가?

---



















<!-- ---

**새로운 Spring Security Filter를 만들고 등록하는 방법을 포스팅 한 블로그(Baeldung)**

- https://www.baeldung.com/spring-security-custom-filter -->




