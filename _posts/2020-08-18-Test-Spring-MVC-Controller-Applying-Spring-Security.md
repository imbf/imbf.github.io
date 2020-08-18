---
layout: post
title:  "Spring Security를 적용한 Spring MVC Controller 단위 테스트 "
date:   2020-08-18
author: Green Frog Developer
categories: Spring
tags: Spring Security MVC Controller Unit Test
---

이번 포스팅에서는 **Spring Security를 적용한 Spring MVC Controller 단위 테스트**에 대해서 알아보도록 하겠다!!

Controller를 테스트하는데 사용한 테스트 프레임워크는 **JUnit 5** 이다.

프로젝트 빌드 툴로 maven을 사용하고 있으며, 테스트를 위해서 pom.xml에 추가할 의존성은 다음과 같다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

위와 같이 테스트와 관련된 프로젝트 의존성을 pom.xml에 추가 했으면 이제 Spring Security를 적용한 Spring MVC Controller 단위 테스트에 대해서 알아보자!!

---

## Controller 

--- 

```java
@RestController
public class MemberController {

    private final DirectoryService directoryService;

    private final MemberService memberService;

    private final ObjectMapper objectMapper;

    public MemberController(DirectoryService directoryService, MemberService memberService, ObjectMapper objectMapper) {
        this.directoryService = directoryService;
        this.memberService = memberService;
        this.objectMapper = objectMapper;
    }

    @GetMapping("/root-directory")
    @ApiImplicitParam(name = "Authorization", value = "Access_Token", required = true, paramType = "header")
    public ResponseEntity getRootDirectories(@AuthenticationPrincipal User user) throws JsonProcessingException {
        Member member = memberService.findMemberByEmail(user.getUsername());
        List<DirectoryDTO> directoryDtoList = directoryService.getDirectoryDTOs(member);
        if (directoryDtoList.size() == 0) {
            return ResponseEntity.status(HttpStatus.NO_CONTENT).build();
        }
        return ResponseEntity.status(HttpStatus.OK)
                .contentType(MediaType.APPLICATION_JSON)
                .body(objectMapper.writeValueAsString(directoryDtoList));
    }
}
```

내가 테스트 할 Controller의 Handler Method는 `GET /root-directory`를 처리하는 메소드이다. 

이 Handler Method를 테스트하기 위해서 생기는 Test Cases는 2개로 다음과 같다.

- Member의 Root Directory가 존재하는 Case Test

- Member의 Root Directory가 존재하지 않는 Case Test

> 여기서 Root Directory란 Member에 속해있는 가장 상위의 Directory를 의미한다.

**Mock 객체를 사용해서 Controller 단위 테스트를 진행할 것이기 때문에, 여기까지만 Controller를 설명하도록 하겠다.**

---

## Controller Test 

---

Controller Unit Test 코드는 다음과 같다. 

(Test 코드 작성시 다양한 라이브러리의 API를 사용하는데 다양한 클래스에 동일한 이름을 가진 Method가 많아 독자가 더 쉽게 해당 Static Method를 식별할 수 있도록 일부러 import Statement는 지우지 않았다)

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import econo.webper.server.directory.DirectoryCategory;
import econo.webper.server.directory.DirectoryService;
import econo.webper.server.directory.dto.DirectoryDTO;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithUserDetails;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = {MemberController.class, ObjectMapper.class})
@WebAppConfiguration
@AutoConfigureMockMvc
public class MemberControllerTest {

    private static final String EMAIL = "a@a";
    private static final Integer ID = 1;
    private static final String NAME = "NAME";
    private static final String PASSWORD = "1234";

    private static final Integer DIRECTORY_ID = 1;
    private static final String DIRECTORY_TITLE = "title";
    private static final Integer PARENT_DIRECTORY_ID = null;

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private DirectoryService directoryService;

    @MockBean
    private MemberService memberService;

    @DisplayName("멤버에 Root Directory가 존재하는 경우 테스트")
    @WithUserDetails
    @Test
    public void getRootDirectoryTest() throws Exception {
        DirectoryDTO directoryDTO = new DirectoryDTO(DIRECTORY_ID, DIRECTORY_TITLE, DirectoryCategory.BLOG, PARENT_DIRECTORY_ID);
        Member member = new Member().builder()
                .id(ID)
                .email(EMAIL)
                .name(NAME)
                .password(PASSWORD)
                .roles(Collections.singletonList(MemberRole.USER))
                .build();

        when(memberService.findMemberByEmail(any())).thenReturn(member);
        when(directoryService.getDirectoryDTOs(member))
                .thenReturn(Collections.singletonList(directoryDTO));

        mockMvc.perform(get("/root-directory"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$[0].id").value(DIRECTORY_ID))
                .andExpect(jsonPath("$[0].title").value(DIRECTORY_TITLE))
                .andExpect(jsonPath("$[0].category").value(DirectoryCategory.BLOG.name()))
                .andExpect(jsonPath("$[0].parentDirectoryId").value(PARENT_DIRECTORY_ID));
    }

    @DisplayName("멤버에 Root Directory가 존재하지 않는 경우 테스트")
    @WithUserDetails
    @Test
    public void getNonRootDirectoryTest() throws Exception {
        Member member = new Member().builder()
                .id(ID)
                .email(EMAIL)
                .name(NAME)
                .password(PASSWORD)
                .roles(Collections.singletonList(MemberRole.USER))
                .build();
        List<DirectoryDTO> nonElementList = new ArrayList<>();

        when(memberService.findMemberByEmail(any())).thenReturn(member);
        when(directoryService.getDirectoryDTOs(member)).thenReturn(nonElementList);
        mockMvc.perform(get("/root-directory"))
                .andExpect(status().isNoContent())
                .andDo(print());
    }

}
```
#### ControllerTest 클래스에 붙은 애노테이션 설명

- **@ExtendWith(SpringExtension.class)**
    - Spring TestContext Framework를 JUnit5의 jupiter programming model에 통합해주는 애노테이션이다.
    - RestController와 같은 Spring Bean을 테스트하기 위해서는 꼭 필요한 애노테이션이다.
    - 위의 애노테이션 이외에도 SpringExtension을 JUnit에 적용하려면 **@SpringJUnitConfig** 또는 **@SpringJUnitWebConfig** 와 같은 애노테이션을 붙여주면 된다.

- **@ContextConfiguration**
    - 테스트를 위해 `ApplicationContext`를 설정하고 로드하는 방법을 정의하는데 사용되는 메타데이터를 정의한다.
    - **@ContextConfiguration**은 Context를 로드하는데 사용되는 Application Context Resource `locations` 또는 `classes` component를 선언한다.
        - ex) `@ContextConfiguration(classes = {MemberController.class, ObjectMapper.class})`
        - ex) `@ContextConfiguration(locations="applicationContext.xml")`
    - **@ContextConfiguration**은 ContextLoader 전략도 선언할 수 있습니다.
        - ex) `@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)`

- **@WebAppConfiguration**
    - 테스트를 위해 로드된 `ApplicationContext`가 `WebApplicationContext`가 되어야 함을 선언하는 애노테이션 입니다.
        - WebApplicationContext란 ApplicationContext를 상속하는 Context이며, 주로 web application에서 사용된다. 주로 web과 연관된 구성요소들(controller, view resolver, ...)등을 다루는 기능을 제공한다.
    - 테스트 클래스에 `@WebAppConfiguration`이 있을 경우, WebApplicationContext가 테스트를 위해 로드되었음을 보장합니다.
    - 웹 어플리케이션에 대한 디폴트 루트 경로는 `file:src/main/webapp`입니다.

- **@AutoConfigureMockMvc**
    - 자동으로 MockMvc를 사용해서 테스트할 수 있게끔 설정해주는 애노테이션
        - MockMvc는 브라우저에서 요청과 응답을 의미하는 객체로서 Controller 테스트를 용이하게 해주는 라이브러리이다.

ControllerTest Class에 붙여진 애노테이션에 대해서 알아보았음으로, 구체적으로 어떻게 Controller Unit Test를 작성했는지에 대해서 알아보자!

**MemberController Bean은 DirectoryService Bean, MemberService Bean, ObjectMapper Bean으로 이루어져 있다.** 나는 MemberController Bean 자체만 테스트 하고 싶기 때문에, DirectoryService Bean과 MemberService Bean을 테스트 할 필요가 없다.

그래서 DirectoryService Bean과 MemberService Bean은 MemberController Bean생성시 @MockBean을 통해서 Mock객체로써 받았다.

그렇다면 **왜 ObjectMapper Bean은 @MockBean을 통해서 Mock 객체로써 받지 않았는가?** 라는 질문을 할 수 있다.

이에 대한 나의 답변은 **Mock 객체는 일반적으로 테스트를 위해 메소드의 리턴 값을 내가 정의한대로 사용하기 위해 사용하는데, ObjectMapper Bean과 같은 경우에는 해당 Bean 자체의 기존 메소드 Return값을 사용해야 했기 때문에 @MockBean으로써 의존성을 받지 않았고, ApplicationContext내에서 관리하는 실제 ObjectMapper Bean 객체를 주입 받았다.**

이로써 MemberController 구성을 완료 했고, 이를 실제 테스트 해 보도록 하겠다.

---

#### 1. 멤버에 Root Directory가 존재하는 Case Test
```java
@DisplayName("멤버에 Root Directory가 존재하는 경우 테스트")
@WithUserDetails
@Test
public void getRootDirectoryTest() throws Exception {
    DirectoryDTO directoryDTO = new DirectoryDTO(DIRECTORY_ID, DIRECTORY_TITLE, DirectoryCategory.BLOG, PARENT_DIRECTORY_ID);
    Member member = new Member().builder()
            .id(ID)
            .email(EMAIL)
            .name(NAME)
            .password(PASSWORD)
            .roles(Collections.singletonList(MemberRole.USER))
            .build();

    when(memberService.findMemberByEmail(any())).thenReturn(member);
    when(directoryService.getDirectoryDTOs(member))
            .thenReturn(Collections.singletonList(directoryDTO));

    mockMvc.perform(get("/root-directory"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$[0].id").value(DIRECTORY_ID))
            .andExpect(jsonPath("$[0].title").value(DIRECTORY_TITLE))
            .andExpect(jsonPath("$[0].category").value(DirectoryCategory.BLOG.name()))
            .andExpect(jsonPath("$[0].parentDirectoryId").value(PARENT_DIRECTORY_ID));
}
```

MockMvc를 통해서 MemberController 테스트를 진행했으며, Spring Security Filter로부터 인증을 통과하기 위해 `@WithUserDetails` 애노테이션을 사용했다.

> If you want to see the Detaild Information of `@WithUserDetails`, See [Here](https://docs.spring.io/spring-security/site/docs/5.3.5.BUILD-SNAPSHOT/reference/html5/#test-method-withuserdetails)

처음에는 `@WithUserDetails`를 사용하는 주 목적인 인증된 User별로 요청에 응답을 보내는 Controller를 테스트 하면 어떨까? 생각 했지만, 이는 Security 뿐만 아니라 DB와도 관련된 테스트를 하는 것이기 때문에, 점점 Controller 단위 테스트가 통합 테스트로 변화하고 있다는 느낌을 받아서, **테스트 대상 User를 인증된 사용자(User)라고 가정하**고 Controller 테스트를 진행했다.

멤버에 Root Directory가 존재하는 경우 Controller의 Handler Method는 response로 body에 Root Directory의 json 데이터가 담겨져 있음으로 이를 테스트 하기위해서 jsonPath를 사용했다.

#### 2. 멤버에 Root Directory가 존재하지 않는 Case Test

```java
@DisplayName("멤버에 Root Directory가 존재하지 않는 경우 테스트")
@WithUserDetails
@Test
public void getNonRootDirectoryTest() throws Exception {
    Member member = new Member().builder()
            .id(ID)
            .email(EMAIL)
            .name(NAME)
            .password(PASSWORD)
            .roles(Collections.singletonList(MemberRole.USER))
            .build();
    List<DirectoryDTO> nonElementList = new ArrayList<>();

    when(memberService.findMemberByEmail(any())).thenReturn(member);
    when(directoryService.getDirectoryDTOs(member)).thenReturn(nonElementList);
    mockMvc.perform(get("/root-directory"))
            .andExpect(status().isNoContent())
            .andDo(print());
}
```

멤버에 Root Directory가 존재하지 않는 Case의 경우도 1번 Case와 동일하게 MockMvc와 @WithUserDetails를 사용해서 테스트를 진행하였다.

멤버에 Root Directory가 존재하지 않는 경우 Handler Method는 Response Body에 어떠한 Json Data도 보내지않고 Http 상태 코드로 204번(NO CONTENT)를 보내주기 때문에 이를 테스트 했다.

---

### 테스트 결과

---

위의 방식대로 테스트 한 결과 2개의 Test Cases가 멋지게 **pass**된 것을 볼 수 있다.

<img src="/assets/spring/Test-Spring-MVC-Controller-Applying-Spring-Security-1.png" style="width:100%">

#### 이 테스트를 기반으로 JUnit5를 활용한 테스트에 대해 전체적인 개념을 알게 되었고, 통합 테스트와 단위 테스트의 차이에 대해서 잘 알게 되었다.

---

#### Reference

- [Spring Security Doc](https://docs.spring.io/spring-security/site/docs/5.3.3.BUILD-SNAPSHOT/reference/html5/)

- [Spring Framework Testing Doc](https://docs.spring.io/spring/docs/5.2.3.RELEASE/spring-framework-reference/testing.html)