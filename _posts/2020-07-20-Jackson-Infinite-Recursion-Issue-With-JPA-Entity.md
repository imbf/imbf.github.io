---
layout: post
title:  "Jackson Infinite Recursion Issue With JPA Entity"
date:   2020-07-20
author: Green Frog Developer
categories: Spring
tags: Spring Jpa Serialize Infinite Recursion
---

이번 포스팅에서는 프로젝트 진행 중 **JPA Entity 객체를 JSON으로 Serialize시킬 때 발생하는 Jackson Infinite Recursion Issue**에 대해서 알아보고 이를 어떻게 해결했는지에 대해서 포스팅 하도록 하겠다.

---

## JPA Entity JSON Serialize

---

JSON으로 Serialize하고 싶은 JPA Entity Class는 다음과 같다. **\[Self Reference\]**

```java
@Entity @Builder @Getter
@NoArgsConstructor @AllArgsConstructor
public class Directory {

    @Id
    @GeneratedValue
    private Integer id;

    @NotNull
    private String title;

    @NotNull
    private DirectoryCategory category;

    @ManyToOne(fetch = FetchType.LAZY)
    private Directory parentDirectory;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Directory> childDirectories = new ArrayList<>();

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Component> components = new ArrayList<>();
}
```

**Spring MVC Handler Method에서 ResponseEntity를 사용해 위의 Entity 객체를 JSON으로 Serialize하는 중에 다음과 같은 예외<u>(Infinite Recursion : StackOverflowError)</u>가 발생했다.**

로그(log)는 다음과 같다.

>**org.springframework.web.util.NestedServletException: Request processing failed; nested exception is org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain**: econo.webper.server.directory.Directory["childDirectories"]->org.hibernate.collection.internal.PersistentBag[0]->econo.webper.server.directory.Directory["parentDirectory"]->econo.webper.server.directory.Directory["childDirectories"]->org.hibernate.collection.internal.PersistentBag[0]->econo.webper.server.directory.Directory["parentDirectory"]->econo.webper.server.directory.Directory["childDirectories"]->org.hibernate.collection.internal.PersistentBag[0]->econo.webper.server.directory.Directory["parentDirectory"]->econo.webper.server.directory.Directory["childDirectories"]-> ....**(계속 반복)**

위의 로그(log)를 간단히 설명하면!! **ResponseEntity는 기본적으로 Jackson 라이브러리를 사용해서 Object를 JSON데이터로 변환해주는데, 위의 Directory 클래스의 객체를 JSON데이터로 변환하기 위해 <u>해당 객체의 childDirectories필드를 참조</u>하게 되고 이는 <u>Hibernate가 제공하는 원본 컬렉션을 감싼 래퍼 컬렉션인 PersistentBag</u>을 가리킨다. -> 다음으로는 첫 번째 원소인 <u>PersistentBag[0](Directory 클래스를 감싼 래퍼 클래스 객체)를 참조</u>하고 -> 이후 래퍼 클래스 객체인 <u>PersistentBag[0](Directory 클래스 객체)의 필드인 ParentDirectory(Directory 클래스 객체)를 참조</u>하게 된다. -> 다시 <u>parentDirectory(Directory 객체)의 childDirectores필드를 참조</u>하게 되며 -> 로그에서 보여진 것처럼 위의 프로세스가 <u>무한루프</u>의 형태로 발생해 StackOverflowError가 발생한다..**

#### JPA Entity 간에 양방향 관계가 존재할 때만 Infinite Recursion이 생기는 줄 알았는데, 위의 Entity 클래스와 같이 JPA Entity가 Self Reference를 하는 경우도 Infinite Recursion이 발생할 수 있다는 사실을 알게 되었다.

우리는 위의 **Infinite Recursion 이슈를 해결하기 위해 <u>3가지 방법</u>**을 사용할 수 있다.
1. **\[사용\] JPA Entity 클래스 내에 Jackson 애노테이션을 위치시키는 방법.** 
 - @JsonIgnore, @JsonManagedReference, @JsonBackReference, @JsonIdentityInfo, ...

2. **ResponseDTO를 만들어 Persistent Object의 데이터를 주입시키는 방법**
 - Self Reference하는 Persistent Object를 JSON으로 Serialize할 때 Infinite Recursion이 발생하는 것이기 때문에 이를 해결하기 위해 Response용 DTO를 생성

3. **Custom Serializer를 만들어 사용하는 방법.**
 - ResponseEntity에 기본으로 제공되는 Jackson 라이브러리의 Databinding을 사용하지 않고 직접 Serializer를 만들어 Infinite Recursion이 일어나지 않도록 Serialize하는 방법

위의 방법 이외에도 다양한 방법들이 많겠지만, 나는 **1번<u>(JPA Entity 클래스 내에 Jackson 애노테이션을 위치시키는 방법)</u>을 사용**했다.

이번 포스팅에서는 내가 사용한 방법인 JPA Entity 클래스 내에 Jackson 애노테이션을 위치시켜 Infinite Recursion을 해결하는 방법에 대해서 주로 설명하겠다.

---

### JPA Entity내에 Jackson 애노테이션을 위치시켜 Infinite Recursion Issue를 해결하는 방법

---

### 1. @JsonIgnore를 사용해서 Infinite Recursion 해결하는 방법

**@JsonIgnore 란?**

- Serialization(Object -> JSON) 및 Deserialization(JSON -> Object)에 사용되는 논리적 속성을 무시하는데 사용된다.
- 보통 field, getter, setter에 사용하는 애노테이션이다.
- @JsonIgnore를 활성화시킬 수 있는 Value를 속성으로 가지고 있으며, Default는 true이고 false일 시 @JsonIgnore가 비활성화 된다.

`@JsonIgnore`를 Directory 클래스의 **parentDirectory 또는 childDirectories필드에 다음과 같이 적용**하면 Infinite Recursion을 끊을 수 있다.
```java
@Entity @Builder @Getter
@NoArgsConstructor @AllArgsConstructor
public class Directory {

    @Id
    @GeneratedValue
    private Integer id;

    @NotNull
    private String title;

    @NotNull
    private DirectoryCategory category;

    @ManyToOne(fetch = FetchType.LAZY)
    @JsonIgnore
    private Directory parentDirectory;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Directory> childDirectories = new ArrayList<>();

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Component> components = new ArrayList<>();
}
```

Directory Test Object를 생성해 JSON Serialize한 결과는 다음과 같다.

<img src="/assets/spring/Jackson-Infinite-Recursion-Issue With-JPA-Entity-1.png" style="width:40%">

**이 애노테이션은 Infinite Recursion 문제를 쉽게 해결하지만, Directory 객체의 parentDirectory라는 필드를 Serialize 및 Deserialize하지 않기 때문에 parentDirectory필드 데이터를 Client가 요구할 시 응답할 수 없다.**

### 2. @JsonManagedReference, @JsonBackReference를 사용해서 Infinite Recursion 해결하는 방법

**@JsonManagedReference, @JsonBackReference란?**

- 서로 대응되는 엔티티간의 match되는 properties 쌍의 부모/자식 관계를 표현하고 처리하기위해 사용되어지는 **Pair of Annotation**이다.

- `@JsonManagedReference` 애노테이션이 붙여진 필드는 **the forward part of the reference(parent)**라고 하며 JSON으로 Serialize되고, `@JsonBackReference` 애노테이션이 붙여진 필드는 **the back part of the reference(child)**라고 하며 JSON으로 Selialize되지 않습니다. **즉, the back part of the reference가 Serialize되지 않음으로 Infinite Recursion이 발생하지 않습니다.**

`@JsonManagedReference, @JsonBackReference`를 Directory 클래스의 **parentDirectory 또는 childDirectories필드에 다음과 같이 적용**하면 Infinite Recursion을 끊을 수 있다.

```java
@Entity @Builder @Getter
@NoArgsConstructor @AllArgsConstructor
public class Directory {

    @Id
    @GeneratedValue
    private Integer id;

    @NotNull
    private String title;

    @NotNull
    private DirectoryCategory category;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JsonBackReference
    private Directory parentDirectory;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JsonManagedReference
    private List<Directory> childDirectories = new ArrayList<>();

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Component> components = new ArrayList<>();
}
```

Directory Test Object를 생성해 JSON Serialize한 결과는 다음과 같다.

<img src="/assets/spring/Jackson-Infinite-Recursion-Issue With-JPA-Entity-1.png" style="width:40%">

Directory Entity가 양방향 관계라면 2개의 Entity의 모든 필드 중에서 하나의 필드만 Serialize되지 않겠지만, 우리는 하나의 Entity내에 Self Reference가 존재하기 때문에 `@JsonBackReference`와 `@JsonManagedReference`의 조합이 Serialize측면에서 `@JsonIgnore`과 동일한 역할을 하게된다. **(즉 Entity내의 하나의 필드는 무조건 Serialize되지 않는다.)**

따라서 `@JsonBackReference`가 붙여진 필드(parentDirectory)의 데이터가 필요한 Client의 입장에서는 데이터를 사용할 수 없음으로 제한이 걸린다.

### 3. @JsonIdentifyInfo를 사용해서 Infinite Recursion을 해결하는 방법

**@JsonIdentifyInfo 란?**

- Serialize / Deserialize 할 때 객체 대신 객체 ID가 사용됨을 나타내는데 사용되어지는 class/property 애노테이션이다.

- 이 애노테이션은 cyclic object graphs와 directed-acyclic graphs를 올바르게 처리하는데 사용되어질 수 있다.

`@JsonIdentifyInfo`를 Directory 클래스에 다음과 같이 적용하면 Infinite Recursion을 끊을 수 있다.

object identifier를 property에 대한 id값을 사용해서 생성하고 싶다면, 다음과 같이 generator속성 값에 ObjectIdGenerators.PropertyGenerator.class를 주어야하고, property속성 값에 id를 주어야 한다.

```java
@Entity @Builder @Getter
@NoArgsConstructor @AllArgsConstructor
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
public class Directory {

    @Id
    @GeneratedValue
    private Integer id;

    @NotNull
    private String title;

    @NotNull
    private DirectoryCategory category;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Directory parentDirectory;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Directory> childDirectories = new ArrayList<>();

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @Builder.Default
    private List<Component> components = new ArrayList<>();
}
```

Directory Test Object를 생성해 JSON Serialize한 결과는 다음과 같다.

<img src="/assets/spring/Jackson-Infinite-Recursion-Issue With-JPA-Entity-3.png" style="width:40%">

Infinite Recursion Issue가 해결 되었고, parentDirectory는 해당 객체의 Id Property값을 통해서 Directory와의 연결관계도 식별 가능하다!!!

**ParentDirectory의 Field 전부를 Json으로 Serialize해서 사용할 이유가 없기 때문에 @JsonIdentifyInfo 애노테이션이 가장 적합하고, 추가적으로 ParentDirectory의 필드 전부를 JSON Serialize해서 사용할 필요가 있다면, Response DTO를 만들어줘도 좋을 것 같다.**


#### Entity를 JSON으로 Serialize하는데 생기는 Infinite Recursion 이슈를 해결하는 더 많은 방법이 궁금하다면 [https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion](https://www.baeldung.com/jackson-bidirectional-relationships-and-infinite-recursion) 를 참조하면 더 많은 키워드 및 설명을 얻을 수 있다.

이상 글을 마치며, 부족한 글임에도 끝까지 봐준 독자들에게 감사를 표한다.

---

**참조 사이트**

- [http://keenformatics.blogspot.com/2013/08/how-to-solve-json-infinite-recursion.html](http://keenformatics.blogspot.com/2013/08/how-to-solve-json-infinite-recursion.html)
-  [https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/web.html](https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/web.html)
- [https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations](https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations)