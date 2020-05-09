---
layout: post
title:  "@RequestParam, @RequestBody에 관한 이슈 (Conversion 관점)"
date:   2020-05-08
author: Green Frog Developer
categories: Spring
tags: Spring Jackson Formatter converter
---

**Conversion관점의 @RequestParam와 @RequestBody에 관한 이슈**에 대해서 다루어 보도록 하겠다.

> **이 이슈는 서버로 Form Data를 보내서 Request handler 메소드의 인자로서 setter가 없는 클래스의 객체를 받고 싶은 욕구에서 시작 되었다.**

**Form Data**
> writer=abc&title=Spring&contents=Johnson

**domain**
```java
@Getter
@Entity
public class Question {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String writer;
    private String title;
    private String contents;
    private String createTime;

    public void update(QuestionUpdateDTO questionUpdateDTO) {
        this.writer = questionUpdateDTO.getWriter();
        this.title = questionUpdateDTO.getTitle();
        this.contents = questionUpdateDTO.getContents();
    }

}
```

**controller 코드 일부**
```java
@PostMapping("/questions")
public String registerQuestion(Question question) {
    questionService.registerQuestion(question);
    return "redirect:/";
}
```

적절한 Form Data를 가진 요청을 `/questions`에게 보내니 Question 클래스에 setter가 없어 <u>기본 생성자에 의해 Question객체</u>가 생성된 것을 볼 수 있었다.

나는 예상치 못한 결과에 다음과 같은 **궁금증**을 가지게 되었다.

1. 우리가 원하는건 요청한 Form 데이터를 가진 객체가 생성되길 원하는데 안되는 이유가 뭘까?
2. 분명히 Form에 적절히 값을 채워서 요청을 보냈는데?
3. 왜 기본 생성자에 의해서만 객체가 생성되는 걸까?
4. Form의 요청 파라미터에 맞는 인자를 가진 생성자를 만든다면 생성자에 의해서 적절한 객체가 잘 생성될 수 있을것인가?

위의 궁금증을 해결하기 위해서 나는 정말 헤매고 헤매다가 **@RequestParam** 이라는 애노테이션을 키워드로 찾을 수 있었다.

>**@RequestParam이란?**
>
>-  HTTP Request 파라미터를 Handler메소드의 파라미터 타입에 전달받을 때 사용하는 애노테이션이다.
>
>- Handler 메소드의 파라미터 타입이 Simple Type이고, 지원하는 인자 타입이 아닌 경우 @RequestParam으로 먼저 resolve되고 이후에 @ModelAttribute로 resolve된다.
>
>- @RequestParam이 붙여진 인자의 타입이 String이 아닌 경우 [Type Conversion](https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/web.html#mvc-ann-typeconversion)이 자동으로 적용된다. (사용자 정의 가능)

@RequestParam가 붙여진 인자의 타입이 String이 아닌 경우 Type Conversion이 자동으로 적용된다는데... **이 Type Conversion의 로직 때문에 setter없이 생성자로써 객체를 생성할 수 없는게 아닐까?** 라는 생각이 들었다.

그렇다. **Type Conversion에서 제공해주는 DataBinder가 이슈의 시작점 이었다!!!** (DataBinder를 이용해서 Type Conversion을 수행)

요청시 WebDataBinder클래스 내의 doBind()메소드의 braeak point에 걸리는 사진.
<img src="/assets/spring/RequestParam-RequestBody-Conversion.png" style="width:100%">

**@RequestParam이 String이 아닐 경우 Object로 변경 해주는건 Formatter도 아니고 Converter도 아니고 <u>DataBinder</u>이었다. 이 DataBinder가 사용하는 <u>PropertyEditor가 Java Bean spec</u>을 따르기 때문에 이를 사용해서 바인딩을 하려는 클래스는 무조건 Java Bean spec을 충족했어야만 했고 즉, <u>setter가 존재</u>해야했다.**

> **Java Bean** : Java Bean API 명세를 따르는 클래스를 의미한다.
> 1. 기본 생성자를 가지고 있어야 한다. 
> 2. 인스턴스 변수는 private 접근 지정자여야 한다.
> 3. public의 getter와 setter를 가지고 있어야 한다.
> 4. Serializable 인터페이스를 구현해야 한다. (선택사항)

그렇다면 DataBinder를 이용해서 Binding되는 객체는 무조건 setter를 필요로 하는가??

**아니다. 다음의 코드를 컨트롤러 내부에 위치시키면 DataBinder가 Reflection을 사용해서 필드에 직접 Access할 수 있도록 만들어 준다.**

```java
@InitBinder
public void initBinder(WebDataBinder binder) {
    binder.initDirectFieldAccess();
}
```

---

#### @RequestBody를 이용하는 방법

Form Data가 아닌 JSON 형식의 데이터를 가진 요청이 들어왔을 경우 Handler method는 **@RequestBody**애노테이션이 붙여진 인자를 **JSON 형태의 데이터에서 setter없는 클래스의 객체로 만들어 준다.**

>#### **@RequestBody란?**
>- @RequestBody애노테이션은 HTTP Request body를 읽고 [HttpMessageConverter](https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/web.html#mvc-config-message-converters)를 통해서 deserialized시켜 Handler 메소드의 특정 인자 타입의 객체로 변환하기 위한 애노테이션이다.
>
>- MVC Config의 Message Converters 옵션을 사용함으로써 message conversion을 설정 및 customizing할 수 있다.
>
>- javax의 @valid, spring의 @validated 애노테이션을 사용함으로써 유효성 검사를 할 수 있다.

**JSON Data**
```json
{
	"writer" : "abc",
	"title" : "Spring",
	"contents" : "Johnson"
}
```

**Handler Method**
```java
@PostMapping("/questions")
public String registerQuestion(@RequestBody Question question) {
    questionService.registerQuestion(question);
    return "redirect:/";
}
```

왜 @RequestBody는 아무런 설정 없이 JSON 형식의 데이터에서 setter가 존재하지 않는 클래스의 객체로 변환할 수 잇었을까?

이에 대한 답은 **MappingJackson2HttpMessageConverter**에 있다.

**MappingJackson2HttpMessageConverter는 ObjectMapper를 사용해서 Setter가 존재하지 않아도 객체의 필드 자체에 데이터를 전달할 수 있기 때문에 위와 같은 빈 객체가 생성되는 이슈가 발생하지 않는것이다.**

기본적으로 동작하는 DataBinder와 MappingJackson2HttpMessageConvert가 맘에 들지 않는다면 커스터마이징 하는것도 아주 좋은 선택이다.
커스터마이징 하는 방법과 등록하는 방법은 Doc에 아주 친절히 나와 있으니 참고 바란다.


















