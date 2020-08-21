---
layout: post
title:  "[Operating System - Chapter 1] 운영체제란 무엇인가?"
date:   2020-08-21
author: Green Frog Developer
categories: Computer-Science(CS)
tags: Operating-System OS CS Computer
---

이 포스팅은 공룡책 알려진 Operating System Concepts의 1장을 공부하면서 정리한 포스팅이다.

---

## 운영체제란 무엇인가?

---

### 운영체제가 할 일 (What Operating Systems Do)

컴퓨터 시스템은 다음 그림처럼 대게 네 가지 구성요소인 **하드웨어, 운영체제, 응용 프로그램, 사용자**로 구분할 수 있다.

<img src="/assets/computer-science/what-is-the-operating-system-1.png" style="width:30%">

**운영체제는 다양한 사용자를 위해 다양한 응용 프로그램 간의 하드웨어 사용을 제어하고 조종한다.**

운영체제는 항상 실행 중인 **커널(Kernel)**과 응용 프로그램 개발을 쉽게 하고 기능을 제공하는 미들웨어 프레임워크 및 시스템 실행 중에 시스템 관리하는데 도움이 되는 **시스템 프로그램(System Program)**이 포함된다.

> 일반적으로 항상 실행되는 프로그램을 **커널(Kernel)**이라고 한다,

