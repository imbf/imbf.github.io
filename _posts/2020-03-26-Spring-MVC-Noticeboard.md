---
layout: post
title:  "Spring MVC를 활용한 게시판 구축하기"
date:   2020-03-26
author: Green Frog Developer
categories: Spring
tags: Spring Java Project MVC
---

이 글은 Spring MVC를 활용한 게시판 구축하는 과정을 포스팅한 글이다.

필자는 개발 툴으로써 Intellij Ultimate를 사용하였음을 알리고 포스팅을 시작하겠다.

---

## Spring MVC의 Request처리 방법

---

스프링 MVC에서는 **`@Controller` 어노테이션이 붙은 클래스 안에 `@RequestMapping` 어노테이션이 붙은 메서드에서 클라이언트 요청을 처리**하게 된다.

이를 그림으로 나타내면 다음과 같다.

<img src="/assets/spring/Spring-MVC-Noticeboard-1.jpg" style="width:70%">

이를 기반으로 게시판을 구현해보자.

스프링 MVC의 특징을 중심으로 간단하게 만드는 게시판이기 때문에 페이징 기능과 검색 기능은 생략했다.

---

## 1. 스프링 MVC 프로젝트 생성

---










---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)