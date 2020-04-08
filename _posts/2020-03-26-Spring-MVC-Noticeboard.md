---
layout: post
title:  "Spring MVC를 활용한 게시판 구축하기"
date:   2020-03-26
author: Green Frog Developer
categories: Spring
tags: Spring Java Project MVC
---

**이 글은 Spring MVC를 활용한 게시판 구축하는 과정을 포스팅한 글이다.**

> **[사용한 개발 환경]**
>
> 자바 : Java8
>
> 운영체제: macOS Catalina
>
> 개발 도구: IntelliJ IDEA ULTIMATE
>
> 서버: Tomcat 8.5.53

---

## Spring MVC의 Request처리 방법

---

스프링 MVC에서는 **`@Controller` 어노테이션이 붙은 클래스 안에 `@RequestMapping` 어노테이션이 붙은 메서드에서 클라이언트 요청을 처리**하게 된다.

이를 그림으로 나타내면 다음과 같다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-1.jpg" style="width:70%">

이를 기반으로 게시판을 구축해보자.

스프링 MVC의 특징을 중심으로 간단하게 만드는 게시판이기 때문에 페이징 기능과 검색 기능은 생략했다.

---

## 1. 스프링 MVC 프로젝트 생성

---

스프링 MVC 프로젝트 생성을 위해 다음의 포스팅을 참고하자.

[**Intellij에서 스프링 MVC Project 생성하기**](https://imbf.github.io/spring/2020/03/28/Create-MVC-Project.html)

---

## 2. 인메모리 DB HSQL 사용하기

---

게시판을 작성하려면 **데이터를 저장할 데이터베이스**가 필요하다. 이 프로젝트에서는 간단하게 사용해 볼 수 있는 **인메모리(in-memory) DBMS중 HSQL(HyperSQL)**을 사용할 것이다.

이 프로젝트는 생성될 때 메이븐 기반으로 생성되기 때문에 라이브러리 관리는 간단하게 **메이븐 설정 파일인 pom.xml에 HSQL에 대한 의존성만 추가**하면 된다.

**메이븐 설정 파일에 HSQL 의존성 추가하는 방법**
1. Maven Repository에 HyperSQL Database를 검색합니다.
2. pom.xml에 추가할 Maven용 텍스트 문자열을 복사합니다.
3. 프로젝트의 pom.xml의 \<dependencies>태그를 찾아 그 아래에 복사해둔 내용을 넣는다.

다음으로는 HSQL을 사용하기 위해 **스프링 프레임워크의 데이터베이스 지원 라이브러리의(jdbc) 의존성**을 다음과 같이 추가한다.
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```

HSQL을 Bean으로 등록하기 위해 다음의 태그를 applicationContext.xml에 추가해 준다. 추가적으로 태그의 속성(jdbc:script)으로 두 개의 SQL파일(스키마 정보, 데이터 정보) 정보를 추가한다.
(type="HSQL"은 Default이기 때문에 생략해도 된다.)
```xml
<jdbc:embedded-database id="dataSource" type="HSQL">
    <jdbc:script location="classpath:BoardSchema.sql"></jdbc:script>
    <jdbc:script location="classpath:BoardData.sql"></jdbc:script>
</jdbc:embedded-database>
```

두 개의 SQL파일 정보는 다음과 같다.

**resources/BoardSchema.sql**
```sql
CREATE TABLE BOARD (
  seq 		INTEGER IDENTITY PRIMARY KEY,
  title 	VARCHAR(255) NOT NULL ,
  content 	VARCHAR(1000) NOT NULL ,
  writer 	VARCHAR(10) NOT NULL ,
  password	INT NOT NULL ,
  regDate 	TIMESTAMP NOT NULL ,
  cnt 		INT NOT NULL
);
```

**resources/BoardData.sql**
```sql
INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t1', 'c1', 'w1', 1234, '2014-09-09 14:21:12', 0);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t2', 'c2', 'w1', 1234, '2014-09-09 14:21:12', 1);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t3', 'c3', 'w1', 1234, '2014-09-09 14:21:12', 2);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t4', 'c4', 'w1', 1234, '2014-09-09 14:21:12', 3);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t5', 'c5', 'w1', 1234, '2014-09-09 14:21:12', 4);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t6', 'c5', 'w1', 1234, '2014-09-09 14:21:12', 5);

INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
VALUES ('t7', 'c5', 'w1', 1234, '2014-09-09 14:21:12', 6);
```

이로써 HSQL을 데이터베이스로 활용하기 위한 준비는 모두 끝났다.

---

## 3. DTO와 MyBatis를 이용한 DAO 구현

---

데이터베이스를 다루기 위해서 **SQL Mapper인 MyBatis**를 사용해 보자.

중대형 프로젝트에서 스프링 프레임워크를 적용하는 경우 다음과 같이 모듈을 구성한다. 참고해서 DTO를 구현해 보자.

<img src="/assets/spring/Spring-MVC-NoticeBoard-2.jpg" style="width:100%">

참고로 **DTO(Data Transfer Object)란 VO(Value Object)로 바꿔 말할 수 있는데 계층간 데이터 교환을 위한 Java Beans**를 말합니다.

DTO는 일반적으로 **데이터베이스의 테이블 구조를 그대로 반영**한다. BoardDTO라는 이름으로 클래스를 생성하자.

**BoardDTO.java** (Data Transfer Object == Value Object)
```java
package board.domain;

import org.apache.ibatis.type.Alias;

import java.sql.Timestamp;

@Alias("boardDTO")
public class BoardDTO {
    private int seq;
    private String title;
    private String content;
    private String writer;
    private int password;
    private Timestamp regDate;
    private int cnt;

    public BoardDTO() {
    }

    public BoardDTO(String title, String content, String writer, int password) {
        super();
        this.title = title;
        this.content = content;
        this.writer = writer;
        this.password = password;
        this.cnt = 0;
    }

    // getter, setter 생략
}
```

이제 MyBatis를 사용하기 위해 **MyBatis의 SqlMapClient에 PSA를 적용한 어댑터를 Bean으로 등록**해보자.

sqlSessionFactory Bean에 MyBatis 설정 파일, SQL 스크립트 파일을 속성으로 추가.

**applicationContext.xml** (스프링 설정 파일)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">

    <!-- Java HSQL database connection -->
    <jdbc:embedded-database id="dataSource" type="HSQL">
        <!-- SQL 파일 정보 추가 -->
        <jdbc:script location="classpath:BoardSchema.sql"></jdbc:script>
        <jdbc:script location="classpath:BoardData.sql"></jdbc:script>
    </jdbc:embedded-database>

    <!-- sqlSessionFactory 인터페이스를 구현하는 sqlSessionFactoryBean클래스 Bean 생성 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- sqlSessionFactoryBean클래스 인스턴스 내의 속성들을 채운다. -->
        <property name="dataSource" ref="dataSource"/>  
        <property name="configLocation" value="classpath:sqlmap/config/mybatis-config.xml"/>
        <property name="mapperLocations">
            <list>
                <value>classpath:sqlmap/sqlmap-board.xml</value>
            </list>
        </property>
    </bean>

    <!-- sqlSessionTemplate을 Bean으로 등록 -->
    <!-- sqlSessionTemplate(SqlMapClient) 객체 생성시 sqlSessionFactory속성에 sqlSessionFactoryBean객체로 채워진다. (Singleton) -->
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
        <constructor-arg ref="sqlSessionFactory"/>
    </bean>
</beans>
```

**mybatis-config.xml** (MyBatis 설정 파일)
```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTO config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 마이바티스 작동 규칙 정의 -->
    <settings>
        <!-- 설정에서 각 매퍼에 설정된 캐시를 전역적으로 사용할지 말지에 대한 여부 -->
        <setting name="cacheEnabled" value="false" />
        <!-- 생성키에 대한 JDBC 지원을 허용. 지원하는 드라이버가 필요하다. true로 설정하면 생성키를 강제로 생성한다. -->
        <setting name="useGeneratedKeys" value="false" />
        <!-- 전통적인 데이터베이스 칼럼명 형태인 A_COLUMN을 CamelCase형태의 자바 프로퍼티명 형태인 aColumn으로 자동으로 매핑하도록 함 -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>

    <!-- 타입 별칭은 자바 타입에 대한 짧은 이름이다. 오직 XML 설정에서만 사용되며, 타이핑을 줄이기 위해 존재한다.-->
    <typeAliases>
        <!-- 이 설정에서 “boardDTO”는 여러군데에서 “board.domain.BoardDTO” 대신 사용할 수 있다. -->
        <typeAlias alias="boardDTO" type="board.domain.BoardDTO"/>
    </typeAliases>
</configuration>
```

**sqlmap-board.xml** (SQL 스크립트 파일)
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTO Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="boardDAO">
    <select id="list" resultType="boardDTO">
        SELECT * FROM BOARD
        ORDER BY seq
    </select>

    <!-- 해당 쿼리문은 int인자 타입을 받아서 boardDTO라는 별칭의 인자 타입을 리턴한다. -->
    <select id="select" parameterType="int" resultType="boardDTO">
        SELECT * FROM
        BOARD WHERE seq=#{seq}
    </select>

    <!-- 들어오는 인자는 boardDTO라는 별칭을 가진 인자이다.  -->
    <insert id="insert" parameterType="boardDTO">
        INSERT INTO BOARD(title, content, writer, password, regDate, cnt)
        VALUES (${title}, ${content}, ${writer}, #{password}, SYSDATE, 0);
        <selectKey keyProperty="seq" resultType="Integer">
            SELECT NVL(MAX(seq), 0) FROM BOARD
        </selectKey>
    </insert>

    <update id="update" parameterType="boardDTO">
        UPDATE BOARD SET title = #{title}, content = #{content}, writer = ${writer}
        WHERE seq = #{seq}
        AND password = #{password}
    </update>

    <update id="updateCount" parameterType="int">
        UPDATE BOARD SET
        cnt = cnt + 1
        WHERE seq = #{seq}
    </update>

    <delete id="delete" parameterType="boardDTO">
        DELETE FROM BOARD
        WHERE seq = #{seq}
        AND password = #{password}
    </delete>

    <delete id="deleteAll">
        DELETE FROM BOARD
    </delete>
</mapper>
```

MyBatis에 관한 설정 파일 및 스크립트 파일을 완성했다면 **database의 data에 접근하기 위한 DAO(Data Access Object)를 만들어 보자**.

다음의 그림을 다시 한번 참조 해 보자!!! ( 여러 번 반복 할 것이다. )

<img src="/assets/spring/Spring-MVC-NoticeBoard-2.jpg" style="width:100%">

확장성을 고려해 BoardDao 인터페이스와 이를 구현하는 BoardDaoMyBatis 클래스를 만들어 보자.

**BoardDao.java** (Data Access Object Interface)
```java
package board.domain;

import java.util.List;

public interface BoardDao {
    public abstract List<BoardDTO> list();

    public abstract int delete(BoardDTO boardDTO);

    public abstract int deleteAll();

    public abstract int update(BoardDTO boardDTO);

    public abstract int insert(BoardDTO boardDTO);

    public abstract BoardDTO select(int seq);

    public abstract int updateReadCount(int seq);
}
```

**BoardDaoMyBatis.java** (Dao를 구현하는 클래스)
```java
package board.domain;

import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.List;
/*
@Repository 애노테이션은 DAO클래스를 Bean으로 등록하기 위해 사용 
(DAO와 DDD 스타일 저장소의 차이점을 이해하고 사용해야 한다.) 
@Repository가 달린 클래스는 PersistenceExceptionTranslationPostProcessor와 함께 사용
되어질 때 Spring DataAccessException 변환에 적합하다. */
@Repository
public class BoardDaoImpl implements BoardDao{

    private SqlSessionTemplate sqlSessionTemplate;

    @Autowired // sqlSessionTemplate 인스턴스 주입 (Sql맵핑 클라이언트를 설정하는 주입 Setter이다.)
    public void setSqlMapClient(SqlSessionTemplate sqlSessionTemplate) {
        this.sqlSessionTemplate = sqlSessionTemplate;
    }

    @Override
    public List<BoardDTO> list() {
        // 인자로써 SQL 스크립트 파일에 명시된 id를 준다.
        return sqlSessionTemplate.selectList("list");
    }

    @Override
    public int delete(BoardDTO boardDTO) {
        // 인자로써 SQL 스크립트 파일에 명시된 id와 parameterType의 객체를 준다.
        return sqlSessionTemplate.delete("delete", boardDTO);
    }

    @Override
    public int deleteAll() {
        // 인자로써 SQL 스크립트 파일에 명시된 id와 parameterType의 객체를 준다.
        return sqlSessionTemplate.delete("deleteAll");
    }

    @Override
    public int update(BoardDTO boardDTO) {
        return sqlSessionTemplate.update("update", boardDTO);
    }

    @Override
    public int insert(BoardDTO boardDTO) {
        return sqlSessionTemplate.insert("insert", boardDTO);
    }

    @Override
    public BoardDTO select(int seq) {
        return sqlSessionTemplate.selectOne("select", seq);
    }

    @Override
    public int updateReadCount(int seq) {
        return sqlSessionTemplate.update("updateCount", seq);
    }
}
```

---

## 4.서비스 구현

---

일반적으로 DAO는 데이터베이스 테이블당 하나를 만들게 된다. 하지만 사용자에게 제공되는 서비스는 **여러 테이블의 정보를 조합해서 제공하는 경우**가 많다. 따라서 하나의 서비스에서
다수의 DAO를 사용하기도 하고 때로는 다수의 서비스가 하나의 DAO를 사용하기도 한다. 물론 대부분의 경우 하나의 서비스가 하나의 DAO와 관계를 맺는다. <u>게시판을 작성하다 보면
서비스가 단순히 DAO에게 위임하는 형태로 구성된 경우가 많다.</u> 이런 경우 서비스의 필요성에 대해 의구심을 갖는 개발자들이 있는데, 확장성과 유연성을 고려하면 서비스를 작성하는
것이 실보다 득이 더 많다. **또한 서비스는 DAO와의 연동뿐만 아니라 서버 기술(웹, 클라이언트/서버)이나 각 벤더별 데이터베이스에 종속되지 않은 로직을 구현하는 곳이기도 하기에**
**반드시 서비스를 작성하는 습관을 들이자.**

다음의 그림을 참고해서 BoardService 인터페이스와 BoardServiceImpl 클래스를 만들어 보자.

<img src="/assets/spring/Spring-MVC-NoticeBoard-2.jpg" style="width:100%">

**BoardService.java**
```java
package board.service;

import board.domain.BoardDTO;

import java.util.List;

public interface BoardService {
    public abstract List<BoardDTO> list();

    public abstract int delete(BoardDTO boardDTO);

    public abstract int edit(BoardDTO boardDTO);

    public abstract void write(BoardDTO boardDTO);

    public abstract BoardDTO read(int seq);
}
```

**BoardServiceImpl.java**
```java
package board.service;

import board.domain.BoardDTO;
import board.domain.BoardDao;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

@Service
public class BoardServiceImpl implements BoardService {
    @Resource
    private BoardDao boardDao;

    public BoardDao getBoardDao() {
        return boardDao;
    }

    public void setBoardDao(BoardDao boardDao) {
        this.boardDao = boardDao;
    }

    @Override
    public List<BoardDTO> list() {
        return boardDao.list();
    }

    @Override
    public int delete(BoardDTO boardDTO) {
        return boardDao.delete(boardDTO);
    }

    @Override
    public int edit(BoardDTO boardDTO) {
        return boardDao.update(boardDTO);
    }

    @Override
    public void write(BoardDTO boardDTO) {
        boardDao.insert(boardDTO);
    }

    @Override
    public BoardDTO read(int seq) {
        return boardDao.select(seq);
    }
}
```

---

## 5.목록 구현 (Controller 구현)

---

다음의 그림을 참조해 사용자가 브라우저 주소창에 /board/list라고 하는 URL을 요청했을 때 이를 처리하는 메서드(RequestMapping Method)를 구현하자.

<img src="/assets/spring/Spring-MVC-NoticeBoard-2.jpg" style="width:100%">

**BoardController.java**
```java
package board.controller;

import board.service.BoardService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class BoardController {

    @Autowired
    private BoardService boardService;

    @RequestMapping(value = "/board/list")
    @ResponseBody
    public String list() {
        return boardService.list().toString();
    }
}
```

Controller를 작성하고 서버를 실행해 localhost:8080/board/list에 요청하면 다음과 같은 결과가 뜬다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-3.png" style="width:100%">

위의 결과가 실행되지 않고 다음과 같은 에러가 뜬다면

<img src="/assets/spring/Spring-MVC-NoticeBoard-4.png" style="width:100%">

이것은 **프로젝트 라이브러리의 의존성간의 충돌**이니 pom.xml을 활용해서 라이브러리 의존성을 다음과 같이 설정해주면 정상 결과가 뜰것이다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-5.png" style="width:70%">

이제 MVC에서 모델(Model)에 해당하는 부분을 구현해 보자. **모델은 컨트롤러에서 뷰로 전달해주는 정보다.** 
스프링 MVC에서 모델을 생성하는 것은 **DispatcherServlet**의 역할이다.

DispatcherServlet이 생성한 모델에 대한 참조 변수는 **@RequestMapping 어노테이션이 붙은 메서드에서 인자를 선언하기만 하면 자동으로 받을 수 있다.**
모델을 사용할 수 있도록 list() 메서드에 Model 타입 인자를 만들어 주고 MVC 모델의 마지막 요소인 뷰를 사용하도록 코드를 변경해 보자.

**BoardController.java의 list 메서드에 추가할 코드**
```java
@RequestMapping(value = "/board/list")
public String list(Model model) { // 모델에 대한 참조변수를 인자로써 추가
    // 모델에 속성을 추가
    model.addAttribute("boardList", boardService.list());
    return "WEB-INF/board/list";   // DispatcherServlet이 뷰를 선정하는 힌트
}
```
DispatcherServlet은 dispatcher-servlet.xml설정 파일을 참고하여 `web/WEB-INF/board/list.jsp`를 **뷰**로써 보여준다.

이제 list.jsp를 생성하자.

**web/WEB-INF/board/list.jsp**
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Insert title here</title>
</head>
<body>
    <table border="1">
        <tr>
            <th>NO</th>
            <th>제목</th>
            <th>작성자</th>
            <th>작성일</th>
            <th>조회수</th>
        </tr>
        <c:forEach var="board" items="${boardList}" varStatus="loop">
            <tr>
                <td>${board.seq}</td>
                <td><a href="<c:url value="/board/read/${board.seq}" /> ">${board.title}</a></td>
                <td>${board.writer}</td>
                <td>${board.regDate}</td>
                <td>${board.cnt}</td>
            </tr>
        </c:forEach>
    </table>
    <a href="<c:url value="/board/write"/> ">새글</a>
</body>
</html>
```

위의 코드에서 볼 수 있는 것처럼 컨트롤러에서 모델에 담아서 보내준 정보인 `boardList`는 **EL 표기법인** `${boardList}`를 이용해서 쉽게 사용할 수 있다.

모든 내용을 저장하고 서버를 다시 시작해 `/board/list`로 요청을 보내면 다음과 같은 화면을 브라우저에서 볼 수 있다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-6.png" style="width:70%">

---

## 6. 읽기 구현

---

완성된 List 화면에 코드 보기를 하면 각 글의 읽기 페이지 링크가 `/board/read/글번호` 형식으로 돼 있음을 확인할 수 있다.

스프링 MVC에서는 이처럼 SEO(Search Engine Optimization, 검색 엔진 최적화)에 최적화된 URL을 처리할 수 있는 메커니즘을 제공한다.
이 때 `@PathVariable` 어노테이션을 사용한다.

BoardController.java에 다음과 같은 메소드를 추가하자.
```java
@RequestMapping(value = "/board/read/{seq}") // {seq}는 경로 변수라고 한다.
public String read(Model model, @PathVariable int seq) { //seq인자에는 @PathVariable 애노테이션으로 인하여 값이 자동 바인딩 된다.
    model.addAttribute("boardDTO", boardService.read(seq));
    return "/WEB-INF/board/read";
}
```

**경로 변수({seq})를 메서드의 인자로 사용하려면 @PathVariable 애노테이션을 인자에 지정하면 된다.**

위의 Request를 처리하기 위해 DispatcherServlet이 찾을 WEB-INF/board/read.jsp는 다음과 같다.

**WEB-INF/board/read.jsp**
```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Insert title here</title>
</head>
<body>
<table border="1">
    <tr>
        <th>제목</th>
        <td>${boardDTO.title}</td>
    </tr>
    <tr>
        <th>작성자</th>
        <td>${boardDTO.writer}</td>
    </tr>
    <tr>
        <th>작성일</th>
        <td>${boardDTO.regDate}</td>
    </tr>
    <tr>
        <th>조회수</th>
        <td>${boardDTO.cnt}</td>
    </tr>
</table>
<div>
    <a href="<c:url value="/board/edit/${boardDTO.seq}"/>">수정</a>
    <a href="<c:url value="/board/list"/>">목록</a>
</div>
</body>
</html>
```

서버를 다시 실행해 해당 URL에 요청을 보내면 다음과 같은 결과를 화면에서 볼 수 있다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-7.png" style="width:50%">

---

## 7. 새 글 구현

---
**Controller 요구사항**
1. `@RequestMapping`에서 호출 방식이 **GET 방식이냐 POST 방식이냐를 구분**해서 같은 URL로 요청이 들어와도 별개의 **메서드가 처리**할 수 있도록 지원하는 글쓰기 요청을 처리하는 메서드를 구현해 보자.
2. `BindingResult` 클래스의 인스턴스를 이용해서 **바인딩 에러 처리** 코드를 메서드에 구현해 보자.
3. 하이버네이트의 Validator를 사용해서 **유효성 검사**를 위한 `@Valid` 애노테이션을 사용해 보자.

Controller 요구사항을 구현하기에 앞서 `dispatcher-servlet.xml`파일에 ViewResolver의 `prefix`속성을 다음과 같이 수정했고, `@Valid` 애노테이션을 사용하기 위해서 `<mvc:annotation-driven/>` 코드를 추가하였다.

```xml
<mvc:annotation-driven/>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF"/>
    <property name="suffix" value=".jsp"/>
</bean>
 
```

요구사항을 구현하기 위해 다음과 같은 메서드를 BoardController.java 파일에 추가하고 중복된 메서드는 지우자.

```java
@RequestMapping(value = "/board/write", method = RequestMethod.GET)
public String write(Model model) {
    model.addAttribute("boardDTO", new BoardDTO());
    return "/board/write";
}

@RequestMapping(value = "/board/write", method = RequestMethod.POST)
public String write(@Valid BoardDTO boardDTO, BindingResult bindingResult) { // 바인딩 결과 인자
    if (bindingResult.hasErrors()) { // 바인딩 에러 처리 코드
        return "/board/write";
    }
    boardService.write(boardDTO);
    return "redirect/board/list";   // PRG 패턴
}
```
위의 코드에서 주목할 점은 **스프링 MVC는 form 태그에서 전송된 input 태그의 name 속성과 BoardDTO 인스턴스의 속성 이름을 비교해 자동으로 그 값을 바인딩해 준다.**

스프링 MVC를 사용하지 않았다면 사용자가 입력 태그를 통해 입력한 내용을 HttpServletRequest의 인스턴스(request)에서 getParameter() 메서드를 이용해
가져온 후 BoardDTO 인스턴스의 속성에 넣어주기 위해 형변환까지 해야 했을 것이다.

> ### **PRG 패턴 (POST - Redirect - GET)**
>
> Post Request후 Redirect를 하지 않고 그냥 뷰만 바꾸어준다면 URL은 POST 요청했을 때와 동일하고 그냥 보여지는 View만 바뀔 것이다.
>
> 이러한 경우에 새로고침을 하게되면 이전의 POST Request가 다시 한번 서버로 요청되어서 데이터베이스에 중복된 값이 저장되는 사태가 발생한다.
>
> **그래서 꼭 POST Request 후에 Redirect를 이용해서 GET Request해야 한다.**

실제 유효성 검증을 위해 BoardDTO를 다음과 같이 수정하자.
```java
package board.domain;

import org.apache.ibatis.type.Alias;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotEmpty;

import java.sql.Timestamp;

@Alias("boardDTO")
public class BoardDTO {
    private int seq;

    @Length(min = 2, max = 5, message = "제목은 2자 이상, 5자 미만 입력하세요.")
    private String title;

    @NotEmpty(message = "내용을 입력하세요.")
    private String content;

    @NotEmpty(message = "작성자를 입력하세요.")
    private String writer;

    private int password;
    private Timestamp regDate;
    private int cnt;

    // 이하 생략
```

다음으로 뷰 역할을 할 JSP 파일을 구현하자.

**/WEB-INF/board/write.jsp**

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Insert title here</title>
</head>
<body>

<!-- form 태그에 action 정보가 없어도 Spring이 브라우저 창을 참고해서 자동으로 action정보를 설정 -->
<form:form commandName="boardDTO" method="post">
    <table border="1">
        <tr>
            <th><form:label path="title">제목</form:label></th>
            <td><form:input path="title"/>
                <form:errors path="title"/></td>
        </tr>
        <tr>
            <th><form:label path="content">내용</form:label></th>
            <td><form:input path="content"/>
                <form:errors path="content"/></td>
        </tr>
        <tr>
            <th><form:label path="writer">작성자</form:label></th>
            <td><form:input path="writer"/>
                <form:errors path="writer"/></td>
        </tr>
        <tr>
            <th><form:label path="password">비밀번호</form:label></th>
            <td><form:input path="password"/>
                <form:errors path="password"/></td>
        </tr>
    </table>
    <div>
        <input type="submit" value="등록">
        <a href="<c:url value="/board/list"/>">목록</a>
    </div>
</form:form>
</body>
</html>

```



모든 코드를 구현해서 새로운 글을 등록한 후 /board/list로 요청을 보낸 응답은 다음과 같다.

<img src="/assets/spring/Spring-MVC-NoticeBoard-8.png" style="width:50%">

---

## 8. 수정 구현

---

글 수정 기능을 구현하기 위해 BoardController.java에 다음의 코드를 추가한다.

```java
@RequestMapping(value = "/board/edit/{seq}", method = RequestMethod.GET)
public String edit(@PathVariable int seq, Model model) {
    BoardDTO boardDTO = boardService.read(seq);
    model.addAttribute("boardDTO", boardDTO);
    return "/board/edit";
}

@RequestMapping(value = "/board/edit/{seq}", method = RequestMethod.POST)
public String edit(@Valid BoardDTO boardDTO, BindingResult result, int pwd, Model model) {
    if(result.hasErrors()) {
        return "/board/edit";
    } else {
        if(boardDTO.getPassword() == pwd) {
            boardService.edit(boardDTO);
            return "/board/list";
        }
    }

    model.addAttribute("msg", "비밀번호가 일치하지 않습니다.");
    return "/board/edit";
}
```



---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)