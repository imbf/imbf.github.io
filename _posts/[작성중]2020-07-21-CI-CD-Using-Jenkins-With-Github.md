---
layout: post
title:  "nCloud 서비스를 활용한 CI/CD With Jenkins"
date:   2020-07-27
author: Green Frog Developer
categories: Spring
tags: CI/CD Jenkins nCloud GitHub ShellScript 
---

1. nCloud 서비스 가입
2. nCloud 서버 생성 (with jenkins)
3. nCloud 공인 IP 할당
4. ACG규칙 적용
5. Jenkins 설정
 - jenkins 플러그인 설정
 - 새로운 아이템 생성
 - 아이템별 구성 설정 (깃헙, build, Script, ...)
 - ssh 설정 (exec script after build, build address, ...)
 - GitHub Webhook 설정
 - 확인

### 리버스 프록시 설정 : https://www.joinc.co.kr/w/man/12/proxy

### 졸두형 Jenkins 설정 : https://jojoldu.tistory.com/290

- sudo vi /etc/nginx/sites-enabled/default 파일에서 프록시 설정 했음 -> 혹시나 에러 있을경우 참조

- **Ubuntu :  sudo unable resolve host Error Message Solution**
  [https://blog.illustudio.co.kr/2017/03/02/%EC%9A%B0%EB%B6%84%ED%88%AC-sudo-unable-resolve-host-%EC%97%90%EB%9F%AC-%EB%A9%94%EC%8B%9C%EC%A7%80-%ED%95%B4%EA%B2%B0/](https://blog.illustudio.co.kr/2017/03/02/우분투-sudo-unable-resolve-host-에러-메시지-해결/)

  

  



