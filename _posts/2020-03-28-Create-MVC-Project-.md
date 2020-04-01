---
layout: post
title:  "Intellij에서 스프링 MVC Project 생성하기"
date:   2020-03-28
author: Green Frog Developer
categories: Spring
tags: Spring Java Project MVC intellij
---

이 글에서는 **Intellij를 활용해서 Spring MVC Project를 생성 및 설정**하는 방법에 대해 소개할 것이다.

> **[사용한 개발 환경]**
>
> 자바 : Java8
>
> 운영체제: macOS Catalina
>
> 개발 도구: IntelliJ IDEA ULTIMATE
>
> 프레임워크: Spring MVC + Maven
>
> 서버: Tomcat 8.5.53

---

## 1. Project 생성하기

---

Intellij를 실행시켜서 가장 첫 번째 요소인 **Create New Project**를 클릭한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-1.png" style="width:50%">

**Maven을 프로젝트 관리 도구**로써 사용하기 위해 클릭하면 
다음과 같은 화면이 나오는데 Project SDK에 **프로젝트에서 사용할 자바 버전**(필자는 자바8 사용)을 입력하고 Next를 누른다.

<img src="/assets/spring/Spring-MVC-Basic-Project-2.png" style="width:70%">

**자신의 프로젝트를 식별해주는 고유 아이디인 GroupId**를 프로젝트에서 컨트롤하는 도메인 이름과 동일하게 입력.

**버전 정보를 생략한 이름(jar)인 ArtifactId**를 프로젝트 이름과 동일하게 입력.

모든 정보를 입력 하였으면 Next를 클릭한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-3.png" style="width:90%">

프로젝트가 생성되면 자동으로 Import를 활성화 해주는 기능인 **Enable Auto-Import**를 클릭한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-4.png" style="width:90%">

---

## 2. Spring MVC 설정하기

---

Spring MVC를 사용하기위해 루트 디렉토리에서 우클릭을 한 후 **Add Framework Support...**를 클릭한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-5.png" style="width:90%">

**Spring MVC(5.2.3.RELEASE)**를 선택하고 OK 클릭

<img src="/assets/spring/Spring-MVC-Basic-Project-6.png" style="width:80%">

그러면 다음과 같이 Web 디렉토리와 파일들이 자동으로 생긴 것을 볼 수 있다.
- **applicationContext.xml** : 빈(Bean) 설정
- **dispatcher-servlet.xml** : 내부 웹 관련 처리 설정
- **web.xml** : Web Application의 설정을 위한 deployment descriptor

<img src="/assets/spring/Spring-MVC-Basic-Project-7.png" style="width:50%">

---

## 3. Tomcat 설정하기

---

SpringBoot에는 내장 Tomcat이 있지만 Spring에는 존재하지 않는다. 따라서 우리는 프로젝트 서버인 Tomcat을 설정해줘야 한다.

우측 상단 툴바에서 **Add Configuration...** 클릭

<img src="/assets/spring/Spring-MVC-Basic-Project-8.png" style="width:90%">

**+** 버튼을 눌러서 **Local Tomcat Server** 를 선택하고 생성한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-9.png" style="width:90%">

상단에 **적절한 이름**을 입력한 후 **톰캣 Fix 설정**을 해주기 위해 Fix를 클릭한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-10.png" style="width:90%">

Deployment 화면으로 이동되면 **Application context**를 **/**로 입력하면 톰캣 Fix 설정이 완료된다.

<img src="/assets/spring/Spring-MVC-Basic-Project-11.png" style="width:90%">

---

## 4. Library 설정하기

---

Spring MVC를 선택하여 프로젝트를 생성하였고 Tomcat 서버도 연동하였지만 서버를 시작하면 에러가 발생한다. 이유가 무엇일까?

그렇다. **Spring, Spring MVC library**를 사용하기 위해서는 <u>추가적인 작업</u>이 필요하다. 

**Project Structure** 창을 띄워서 **Artifacts 탭**을 클릭하면 아래와 같은 화면을 볼 수 있다.

<img src="/assets/spring/Spring-MVC-Basic-Project-12.png" style="width:90%">

우측에 **Available Elements** 위치한 것들이 **이용 가능한 Library** 들인데 사용하기 위해서는 
double-click 클릭 혹은 drag-and-drop으로 **좌측의 WEB-INF/lib/** 위치로 다음과 같이 옮겨야 한다.

<img src="/assets/spring/Spring-MVC-Basic-Project-13.png" style="width:70%">

이제 라이브러리 설정도 완료 되었으니 Spring MVC 프로젝트를 시작할 준비는 끝났다!! 한번 서버를 시작 해 보자.

실행결과는 다음과 같이 index.jsp 파일이 브라우저에 보여지는 것을 알수 있다.

<img src="/assets/spring/Spring-MVC-Basic-Project-14.png" style="width:100%">

---

## 5. Spring MVC 프로젝트 활용

---

잘 구성된 Spring MVC 프로젝트를 간단하게 활용해 보자.

### 5-1. 자바 컨트롤러 파일 생성

```java
package controller;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Handles requests for the application home page.
 */
@Controller
public class HomeController {

    private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

    /**
     * Simply selects the home view to render by returning its name.
     */
    @RequestMapping(value = "/", method = RequestMethod.GET)
    public String home(Locale locale, Model model) {
        logger.info("Welcome home! The client locale is {}.", locale);

        Date date = new Date();
        DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

        String formattedDate = dateFormat.format(date);

        model.addAttribute("serverTime", formattedDate );

        return "home";
    }

    @RequestMapping(value = "/show")
    @ResponseBody
    public String show() {
        return "<h1>show</h1>";
    }

}
```
- **`@Controller`** 
    - `@Component`보다 더 전문화된 역할을 수행한다. (실제로 @Controller는 @Component 애노테이션을 포함한다.)
    - 이 어노테이션을 붙이면 dispatcher-servlet.xml에서 이것을 인식하여 **컨트롤러로 등록**함.
    - 클래스 경로 스캔을 통해 구현 클래스를 autoscan 할 수 있습니다.
    - 일반적으로 `@RequestMapping` 애노테이션을 기반으로 어노테이션이있는 Handler 메소드와 함께 사용됩니다.
- `@RequestMapping` : 스프링은 **HandlerMppaing에 의해 컨트롤러가 결정**된다. 이 **컨트롤러에서 HandlerAdapter에 의해 실행 메서드가 결정**되는 데 **@RequestMapping 어노테이션이 그 정보를 제공해 준다.** value에 해당하는 url이 GET 방식으로 요청이 들어올 때 해당 메서드를 실행한다.
- home 메서드는 serverTime이라는 속성을 Model에 추가하고 이 값은 formattedDate 변수 안에 담긴 현재 날짜 정보를 담고있다. 이 정보는 **JSP에서 클라이언트에게 전달할 HTML문서를 만들 때 쓰인다. 여기서 모델은 어떤 구조화된 데이터를 담는 객체**라고 보면 된다.
- 마지막으로 "home" 문자열을 반환하는 데 이 문자열은 나중에 **servlet-context.xml에 설정된 prefix와 suffix 정보를 참조하여 /WEB-INF/home.jsp 파일을 찾는 정보를 제공**한다.

### 5-2. 메이븐 설정 파일(pom.xml) 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>groupId</groupId>
    <artifactId>mvc</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>

</project>
```
- Java Logging API인 slf4j를 사용하기 위해 라이브러리 의존성을 추가해준다.

### 5-3. web.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```
- **DispatcherServlet 클래스를 서블릿으로 정의**하며 클라이언트가 **/로 요청**을 하면 모두 **DispatcherServlet 클래스로 랩핑**하도록 정의하고 있다.
- ContextLoaderListener 클래스는 스프링 설정 파일(디폴트에서 파일명 applicationContext.xml)을 로드하면 ServletContextListener 인터페이스를 구현하고 있기 때문에 ServletContext 인스턴스 생성 시(톰켓으로 어플리케이션이 로드된 때)에 호출된다. **즉, ContextLoaderListener 클래스는 DispatcherServlet 클래스의 로드보다 먼저 동작하여 <u>비즈니스 로직층을 정의한 스프링 설정 파일을 로드</u>한다.**


### 5-4. Dispatcher-servlet.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="controller"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```
- 매번 스프링 설정 파일에 Bean으로 등록하기 어려우니까 base-package인 controller 패키지를 기준으로 **@Component애노테이션을 가진 속성을 찾아서 Bean으로 등록**해주기 위한 코드가 component-scan 태그이다. ()
- **이름으로부터 실제로 사용할 뷰 객체를 결정해 주는 ViewResolver가 필요**하기 때문에 dispatcherServlet은 기본 뷰 리졸버인 InternalResourceViewResolver를 사용한다.
InternalResourceViewResolver는 JSP를 뷰로 사용할 때 쓰인다. 컨트롤러에서 리턴하는 뷰 이름에 접두,접미어를 붙여서 JSP페이지의 경로를 찾는다.
**즉, 뷰 리졸버는 "/WEB-INF/"라는 위치의 이름.JSP 뷰를 참고해서 모델을 이용하여 페이지를 만든다.**

### 5-5. jsp(View) 페이지 작성

jsp란 Java 언어를 기반으로 하는 Server Side 스크립트 언어이다. 이 프로젝트에서 작성할 home.jsp는 다음과 같다.

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Home</title>
</head>
<body>
<h1>
    Hello world!
</h1>

<P>  The time on the server is ${serverTime}. </P>
</body>
</html>
```
- serverTime이라는 속성에서 값을 가져와 html페이지를 만들어서 client에게 전달한다.

### 5-6. 실행 결과

위의 프로젝트를 실행한 결과는 다음과 같다.

<a href="/2020-03-26-Spring-MVC-Noticeboard.md" >
</a>

---
> 참조 
> - https://engkimbs.tistory.com/688
> - https://whitepaek.tistory.com/41