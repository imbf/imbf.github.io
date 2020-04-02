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

<img src="/assets/spring/Spring-MVC-Noticeboard-1.jpg" style="width:70%">

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

다음으로는 HSQL을 사용하기 위해 **스프링 프레임워크의 데이터베이스 지원 라이브러리의 의존성**을 다음과 같이 추가한다.
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```

HSQL을 Bean으로 등록하기 위해 다음의 태그를 applicationContext.xml에 추가해 준다. 추가적으로 태그의 속성으로 두 개의 SQL파일 정보를 추가한다.
```xml
<jdbc:embedded-database id="dataSource" type="HSQL">
    <jdbc:script location="classpath:BoardSchema.sql"></jdbc:script>
    <jdbc:script location="classpath:BoardData.sql"></jdbc:script>
</jdbc:embedded-database>
```

두 개의 SQL파일 정보는 다음과 같다.

/src/main/resources/BoardSchema.sql
```sql
CREATE TABLE BOARD {
    seq INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY ,
    title VARCHAR(255) NOT NULL ,
    content VARCHAR(1000) NOT NULL ,
    writer VARCHAR(10) NOT NULL ,
    password INT NOT NULL ,
    regDate TIMESTAMP NOT NULL ,
    cnt INT NOT NULL
};
```

/src/main/resources/BoardData.sql
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

참고로 **DTO(Data Transfer Object)란 VO(Value Object)로 바꿔 말할 수 있는데 계층간 데이터 교환을 위한 Java Beans를 말합니다.**

<img src="/assets/spring/Spring-MVC-NoticeBoard-2.jpg" style="width:100%">

DTO는 일반적으로 **데이터베이스의 테이블 구조를 그대로 반영**한다. BoardDTO라는 이름으로 클래스를 생성하자.

**BoardDTO.java** (Data Transfer Object)
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

이제 MyBatis를 사용하기 위해 MyBatis의 SqlMapClient에 PSA를 적용한 어댑터를 Bean으로 등록해보자.

sqlSessionFactory Bean에 MyBatis 설정 파일, SQL 스크립트 파일을 속성으로 추가.

**applicationContext.xml** (스프링 설정 파일)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">

    <!-- java database connection -->
    <jdbc:embedded-database id="dataSource" type="HSQL">
        <jdbc:script location="classpath:BoardSchema.sql"></jdbc:script>
        <jdbc:script location="classpath:BoardData.sql"></jdbc:script>
    </jdbc:embedded-database>

    <!-- sqlSessionFactory 인터페이스를 구현하는 sqlSessionFactoryBean클래스 Bean 생성 (다양한 속성을 채워서 생성)-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:sqlmap/config/mybatis-config.xml"/>
        <property name="mapperLocations">
            <list>
                <value>classpath:sqlmap/sqlmap-board.xml</value>
            </list>
        </property>
    </bean>

    <!-- sqlSessionTemplate을 Spring Bean으로 등록 -->
    <!-- sqlSessionTemplate객체 생성시 sqlSessionFactory속성에 sqlSessionFactoryBean객체로 채워진다. (Singleton) -->
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
        <setting name="cacheEnabled" value="false" />
        <setting name="useGeneratedKeys" value="false" />
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>

    <typeAliases>
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

    <select id="select" parameterType="int" resultType="boardDTO">
        SELECT * FROM
        BOARD WHERE seq=#{seq}
    </select>

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

확장성을 고려해 BoardDao 인터페이스와 이를 구현하는 BoardDaoMyBatis 클래스를 만들어 보자.

**BoardDao.java** (Data Access Object Interface) : 확장성을 위함
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

@Repository
public class BoardDaoMyBatis implements BoardDao{
    private SqlSessionTemplate sqlSessionTemplate;

    @Autowired // sqlSessionTemplate Bean주입 (Sql맵핑 클라이언트를 설정하는 주입 Setter이다.)
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



---

> 참조 : 스프링 입문을 위한 자바 객체 지향의 원리와 이해(저자: 김종민)