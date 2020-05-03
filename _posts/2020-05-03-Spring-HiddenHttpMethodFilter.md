---
layout: post
title:  "Spring의 HiddenHttpMethodFilter에 관한 이슈"
date:   2020-05-03
author: Green Frog Developer
categories: Spring
tags: Spring Data
---

이 포스팅에서는 **HTTP Method의 처리 프로세스**에 대한 서버단의 이슈에 관해서 설명하고자 한다.

HTML Form에서는 GET과 POST방식의 Methods만 지원한다. HTML Form이 다른 Methods를 지원하지 않는 이유는 **Form의 역할과는 거리가 멀기 때문**이다.

왜 HTML Form이 GET과 POST방식의 Methods만 지원하는지에 대해서 궁금한 독자는 다음의 블로그[(http://haah.kr/2017/05/23/rest-http-method-in-html-form/)](http://haah.kr/2017/05/23/rest-http-method-in-html-form/) 를 읽어보자.

이제부터 HTTP Method에 관해서 나에게 어떠한 **이슈**가 있었고 어떻게 해결했는지에 대해서 알려줄 것이다.

---

## 이슈(에러) 발생

---

아래의 폼은 사용자를 id로 식별하여 정보를 수정하기 위해 수정할 정보를 서버로 전송하는 폼이다.

**updateForm.html**
```html
<!-- ... -->
<form name="question" method="post" action="/users/{user.id}/update">
    <input type="hidden" name="_method" value="put"/>
    <div class="form-group">
        <label>사용자 아이디 : {user.userId}</label>
    </div>
<!-- ... -->
```

사용자의 정보를 수정하기 위해서 클라이언트는 수정할 데이터를 **PUT 형식으로 서버에 요청**해야 한다.

하지만 PUT형식을 제공해 주지 않는 HTML 때문에 우리는 나름 꼼수(?)를 사용해서 Spring이 알아먹ㄴ을 수 있도록 `<input type="hidden" name="_method" value="put"/>`를 다음과 같이 폼 태그 내부에 위치시켜야 한다.

나는 이렇게만 하면 Controller의 `@PutMapping("/users/{id}/update")`애노테이션을 갖은 핸들러 메서드가 해당 요청에 대해서 잘 처리할 줄 알았다.

하지만 누가 알았을까..? 정말 이제 시작이란것을..

다음은 위의 폼의 요청에 따른 서버에서 보내는 응답이다.

<img src="/assets/spring/Spring-HiddenHttpMethodFilter-1.png" style="width:100%">

보이는가..? 서버는 이 요청을 POST로 해석하고 있다... 어찌된 일인가 도통 모르겠다... 일단 해결해 보자!!

---

## 해결 방안

---

정말 이 이슈를 해결하는데 생각보다 오래걸렸고, 생각보다 힘들었다.

이제와서 생각이 드는 거지만, 이러한 **이슈들을 해결하는데 오래걸리고 힘든 이유**는 기존 **내 머리속의 지식 안에서만 문제를 해결**하려고 하니까 힘들고 오래걸리는 듯 하다.

말이 길었다. 어떻게 이러한 문제를 해결할 수 있었는지에 대해서 설명하겠다. (Spring Web MVC Document에 친절하게 나와있다.)

아까 Spring에서 지원해주는 꼼수라는 말을 기억하는가??? 그렇다. Spring Web MVC는 `<input type="hidden" name="_method" value="put"/>`를 사용해서 POST를 PUT으로 해석한다.

하지만, Spring은 신이 아니다... 당연히 이러한 역할을 해주는 Filter가 Bean으로 등록되어야 한다. 이러한 역할을 해주는 Filter는 **HiddenHttpMethodFilter**이다. 이름 그대로 숨겨진 HTTP Method를 필터하는 클래스이다.

> 정확히 말해서 HiddenHttpMethodFilter는 Hidden 타입의 input 태그의 속성들을 읽어서 HttpServletRequestWrapper.getMethod() 반환 값을 변경해 요청된 HTTP 메소드의 타입을 PUT, DELETE, PATCH로 변경해주는 필터이다.

**HiddenHttpMethodFilter**클래스를 Bean으로 등록하면 위의 이슈(문제는) 해결된다.

다음의 코드는 HiddenHttpMethodFilter를 Bean으로 등록하는 코드이다.

```java
@Configuration
public class MvcConfig extends WebMvcConfigurationSupport {

    @Bean
    public HiddenHttpMethodFilter httpMethodFilter() {
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return hiddenHttpMethodFilter;
    }

}
```

이로써 이번 포스팅은 마친다.

---
> 참조
> - [Spring Web MVC Document](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)
> - [Spring API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/HiddenHttpMethodFilter.html)















