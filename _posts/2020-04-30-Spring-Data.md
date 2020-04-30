---
layout: post
title:  "Spring Data란 무엇인가?"
date:   2020-04-30
author: Green Frog Developer
categories: Spring
tags: Spring Data
---

이 글에서는 **Spring Data**란 무엇인지에 대해서 알아볼 것이다.

---

## 1. Spring Data

---

### Spring Data란?

- Spring Data의 목적은 기본 데이터 저장소의 특수한 특성을 유지하면서 **데이터 접근을 위한 친숙하고 일관된 Spring 기반의 프로그래밍 모델을 제공**하는 프로젝트이다.

- Spring Data는 **데이터 접근 기술, relational and non-relational database, map-reduce 프레임워크, 클라우드 기반의 서비스**를 쉽게 사용할 수 있도록 해준다.

- Spring Data는 데이터베이스와 관련된 많은 하위 프로젝트(Spring Data JPA, Spring Data REST, ...)를 포함하는 포괄적인 프로젝트이다.

### Spring Data의 특징

- 강력한 저장소 및 사용자 정의 객체 맵핑 추상화

- 저장소 메소드 이름으로부터 동적 쿼리 파생

- 기본 속성을 제공하는 Implementation domain 기본 클래스

- transparent auditing을 위한 지원 (created, last changed)

- 사용자 정의 저장소 코드를 통합할 수 있는 가능성

- JavaConfig 및 custom XML namespaces를 통한 손쉬운 Spring과의 통합

- Spring MVC 컨트롤러와의 향상된 통합

- cross-store persistence에 대한 실험적 지원

### 주요 모듈

- **Spring Data Commons :** 모든 Spring Module을 뒷받침하는 핵심 Spring 개념 (CrudRepository, PagingAndSortingRepository 인터페이스)

- **Spring Data JDBC :** spring-jdbc에 대한 Spring Data 추상화를 제공하는 모듈 (CrudRepository를 구현하는 SimpleJdbcRepository 클래스)

- **Spring Data JPA :** JPA를 위한 스프링 데이터 저장소 지원 (JpaRepository 인터페이스, SimpleJpaRepository 클래스)

- **Spring Data MongoDB :** MongoDB를 위한 스프링 기반 객체 문서 지원 및 저장소 

- **Spring Data REST :** 스프링 데이터 저장소들을 hypermedia 기반의 Restful 리소스로 export 해주는 모듈

- **Spring Data Redis :** Spring Application에서 Redis를 손쉽게 구성 및 접근할 수 있도록 하는 모듈

- ...

### Community Module

- Spring Data Aerospike

- Spring Data ArangoDB

- Spring Data Couchbase

- Spring Data Azure Cosmos DB

- Spring Data Cloud Datastore

- Spring Data Cloud Spanner

- Spring Data DynamoDB

- ...

---
> 참조 
> - Spring Reference
