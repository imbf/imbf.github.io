---
layout: post
title:  "[기술 면접 준비 - 5일차] 스마트 도어락 (Mobius)"
date:   2020-12-13
author: Green Frog Developer
categories: Interview
tags: Smart-Doorlock Mobius Node.js oneM2M
---

# 스마트 도어락 (Smart Doorlock)

### 스마트 도어락 프로젝트를 진행하셨는데 스마트 도어락이 무엇인가요??

**스마트 도어락이란 IoT 오픈소스 서버 플랫폼인 Mobius를 활용한 사용자의 완벽한 외출을 도와주는 IoT 플랫폼입니다.** 자택 현관문 내부에 존재하는 기존의 도어락 본체는 제거하고 그 자리에 터치식 디스플레이를 설치하여 **사용자에게 외출시에 필요한 날씨 및 미세먼지 등의 다양한 정보를 제공하고 가스 밸브 및 전등과 같은 가전들을 등록해 상태확인 및 제어할 수 있는 기능을 가진 IoT 플랫폼**입니다.

### 모비우스 플랫폼을 사용하셨다고 했는데 모비우스 플랫폼은 무엇인가요??

**모비우스란 oneM2M 표준을 기반으로한 오픈소스 IoT 서버 플랫폼입니다.** Mobius는 다양한 도메인 IoT 애플리케이션에 대한 미들웨어로써 oneM2M표준에 해당하는 공통 서비스 기능인 CSE와 AE 등록, 데이터 관리, 구독/알림 서비스, 보안 등을 제공합니다.

### oneM2M이란 무엇인가요? 표준을 말씀하시는 건가요??

**oneM2M이란 말 그대로 one + M2M의 합성어로 Machine간의 통신을 하나의 인프라 환경에 통합하기 위한 플랫폼 개발 표준화 단체 및 표준입니다.**

<!-- **산업별로 파편화된 서비스 플랫폼 개발 구조에서 벗어나 응용서비스 인프라 환경을 통합하는 사물인터넷 공동서비스 플랫폼 개발을 위한 표준화 단체 및 표준입니다.** -->

### 방금 말씀하신 IN은 무엇인가요?

**IN 이란 Infrastructure Node를 의미하는 것으로 인프라 도메인에 위치한 IN-CSE를 포함하는 서버 기기**를 의미합니다.

### 방금 말씀하신 CSE는 무엇인가요?

**CSE란 Common Service Entity의 줄임말로 oneM2M 서비스 플랫폼에서 공통적으로 제공되어야하는 서비스 기능을 제공하는 미들웨어**를 말합니다.

### 미들웨어란 무엇인가요??

**미들웨어란 OS와 응용 소프트웨어 중간에서 조정과 중개의 역할을 수행하는 소프트웨어**를 말하구요 미들웨어를 사용하면 응용 소프트웨어가 운영체제로부터 제공받는 서비스 이외에 추가적인 서비스를 받을 수 있습니다.

ex) 메시지 처리 소프트웨어, 데이터베이스 시스템

### 방금 말씀하신 ADN이란 무엇인가요??

**ADN이란 Application Dedicated Node의 줄임말로 필드 도메인에 위치한 ADN-AE를 포함하고 CSE를 포함하지 않는 논리적 기기**를 말합니다.

### 방금 말씀하신 AE란 무엇인가요??

**AE란 Application Entitiy의 줄임말로 M2M(Machine to Machine)서비스를 제공하기 위한 논리적인 엔티티**를 의미합니다.

### &Cube와 TAS는 무엇인가요?

**&Cube는 IoT 디바이스에 탑재되는 S/W 플랫폼으로 IN-CSE와 연동할 수 있도록 지원하는 소프트웨어**입니다. **TAS란 실제 사물을 디바이스에 연결하기 위한 S/W로써 사물과 &Cube간의 연결통로를 만드는 역할**을 한다.

즉, 사물과 &Cube를 Tas로써 연결하고 사물과 IN-CSE 즉, Mobius를 &Cube로써 연결할 수 있게 해줍니다.

### Node.js를 IN-AE로 사용한 이유가 있나요??

꼭 Node.js를 사용해야만하는 특별한 이유는 없었구요, **마감까지 프로젝트를 빠르게 빌드업** 시키기 위해서 사용했던 것 같습니다.

### Node.js의 특징에 대해서 알고 있나요??

**Node.js란 자바스크립트를 사용하고 Non-blocking I/O와 단일-스레드 이벤트 루프를 통한 높은 처리 성능**을 가지고 있습니다. 또한 **내장 HTTP 서버 라이브러리를 포함**하고 있어 웹 서버에서 아파치 등의 별도의 소프트웨어 없이 동작하는 것이 가능하기 때문에 빠르게 프로토타입을 제작 가능합니다.

### Non-blocking I/O란 무엇인가요?

**여러 I/O 가 실행될 때 하나의 I/O가 다른 I/O에 의해서 block되지 않고 병렬로써 실행**되는 것을 의미합니다.