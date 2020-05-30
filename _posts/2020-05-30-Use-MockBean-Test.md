---
layout: post
title:  "@MockBean을 사용한 통합(Controller)테스트"
date:   2020-05-30
author: Green Frog Developer
categories: Spring
tags: Spring Mockito Integration Test
---

**이 글에서는 `@MockBean`을 사용한 통합테스트에 관한 이슈에 대해서 다루어 보도록 하겠다.**

> **이 이슈는 Google OAuth를 사용하는 Service레이어를 의존하는 Controller를 테스트하는 과정에서 발생되었다.**

Controller 코드는 다음과 같다.
```java
@RestController
public class GoogleLoginController {

    GoogleLoginService googleLoginService;

    UserService userService;

    public GoogleLoginController(GoogleLoginService googleLoginService, UserService userService) {
        this.googleLoginService = googleLoginService;
        this.userService = userService;
    }

    @PostMapping("/login/google")
    public ResponseEntity loginByGoogleOAuth(@RequestBody String requestBody) {
        String accessToken = (String) JsonExtractor.getValueByKey(requestBody, "access_token");
        ResponseEntity responseEntity = googleLoginService.authenticate(accessToken);
        if (responseEntity.getStatusCode().is2xxSuccessful()) {
            userService.saveUser((GoogleUserinfoDTO) responseEntity.getBody());
        }
        return responseEntity;
    }
}
```
위 코드에 대해서 간략히 설명하자면

`/login/google` 앤드포인트로 구글 인증서버에서 받은 Access_Token을 넘겨주면 핸들러 메소드가 `GoogleLoginService`를 사용해서 해당 Access_Token이 유효한지 아닌지를 체크하는 코드이다.<u>(Access_Token이 유효한지 아닌지에 대한 검증은 GoogleLoginService의 RestTemplate이 구글 인증 서버에 요청을 보내 진행)</u>

위의 컨트롤러를 테스트하기 위해 **2가지의 경우의 수**를 나누었다.
1. 유효하지 않은 Access_Token을 가진 요청을 보내 4xx 상태코드를 가진 응답 테스트
2. 유효한 Access_Token을 가진 요청을 보내 2xx 상태코드를 가진 응답 테스트

---

### 유효하지 않은 Access_Token을 가진 요청을 보내 4xx 상태코드를 가진 응답 테스트

유효하지 않은 Access_Token을 테스트하는 작업은 매우 수월했다.

내가 임의로 만든 Access_Token(구글의 Access_Token 형식을 알기 때문에 절대 겹칠일이 없다는 가정하에 진행)을 가진 요청을 `/login/google`에게만 보내면 되었다.

```java
@Test
public void GoogleLoginWithWrongAccessToken() throws Exception {
    // Given
    String GoogleAccessToken =  "basdhifbasduiofbasdohiufsaoi";

    // When & Then
    this.mockMvc.perform(post("/login/google")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"access_token\" : \"" + GoogleAccessToken + "\"}"))
            .andDo(print())
            .andExpect(status().isBadRequest());
}
```

위의 결과는 당연히 성공이었다!!!
<img src="/assets/spring/Use-MockBean-Test-1.png" style="width:30%">

하지만 문제는 두번째 경우 테스트 이었다....

---

### 유효한 Access_Token을 가진 요청을 보내 2xx 상태코드를 가진 응답 테스트

유효한 Access_Token을 가진 요청을 테스트 하기 위해서는 유효한 Access_Token을 구글 인증 서버로부터 받아야 하는데 매번 테스트 할 때마다 유효한 Access_Token을 받아올 수는 없는 노릇이었다. 

그렇다면 이전에 사용해 보았던 **Mock객체를 활용해 GoogleLoginService의 메소드 return 값을 내가 설정해 보면 어떨까??** 라는 생각이 들었다.

하지만 다시 생각해 보면 통합(Controller)테스트 인데 Spring Boot가 뜰 때 어떻게 `GoogleLoginService` Mock객체를 `Controller`에 주입해 줄 수 있을까??? 라는 또 다른 이슈가 생겼다!!!

여기 저기 참고해 보고, Sproutt스터디에 질문을 올려본 결과 으쌰으쌰 프로젝트에서 `@MockBean`을 사용하고 있는 걸 볼 수 있었고, 이에 대해서 찾아보니 나의 이슈를 해결할 수 있는 애노테이션이라는 것을 알게 되었다.

유효한 Access_Token 테스트 코드를 작성하기 이전에 `@MockBean`에 대해서 알아보자!

> ### @MockBean
> 
>  - `@MockBean`은 Spring ApplicationContext에 Mock객체를 추가하게 해주는 주석이다.
>  - `@Configuration` 또는 `@RunWith(SpringRunner.class)` 클래스내의 필드에서 사용할 수 있고, Class Level의 주석으로써도 사용할 수 있다.
>  -  **Mock객체는 타입 또는 Bean name으로써 Context에 등록할 수 있고, Context내에서 정의된 동일한 유형의 단일 Bean이 존재한다면 어플리케이션 실행시 Mock Bean으로써 대체된다.** 만약 동일한 타입의 Bean이 존재하지 않는다면 새로운 Mock Bean이 생성된다.
>  - ApplicationContext에 알려져 있지만 Bean이 아닌 의존성은 발견되지 않을 것이며 Mock객체는 존재하는 의존성과 함께 Context에 추가되어 질 것이다.

위의 @MockBean의 Spec을 참고해 작성한 **유효한 Access_Token을 가진 요청을 보내 2xx 상태코드를 가진 응답 테스트 코드**는 다음과 같다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class GoogleLoginControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    GoogleLoginService googleLoginService;

    @Autowired
    UserService userService;

    @Test
    public void GoogleLoginWithRightAccessToken() throws Exception {
        // Given
        GoogleUserinfoDTO googleUserinfoDTO = new GoogleUserinfoDTO();
        googleUserinfoDTO.setName("frog");
        googleUserinfoDTO.setEmail("frog@email.com");
        ResponseEntity<GoogleUserinfoDTO> responseEntity = ResponseEntity.ok(googleUserinfoDTO);

        // When & Then
        when(googleLoginService.authenticate(eq("Right_Access_Token")))
                .thenReturn(responseEntity);

        this.mockMvc.perform(post("/login/google")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"access_token\" : \"Right_Access_Token\"}"))
                .andDo(print())
                .andExpect(status().isOk());

        assertThat(userService.findUserByEmail("frog@email.com").getName()).isEqualTo("frog");
    }

}
```

위의 결과는 다음과 같다.

<img src="/assets/spring/Use-MockBean-Test-2.png" style="width:70%">

`@MockBean`의 개념과 사용방법에 대해서 알아보았으니까 유효하지 않은 Access_Token을 테스트하는 코드를 리팩토링 하면 다음과 같다.
```java
    @Test
    public void GoogleLoginWithWrongAccessToken() throws Exception {
        // When & Then
        when(googleLoginService.authenticate(eq("Wrong_Access_Token")))
                .thenReturn(ResponseEntity.badRequest().build());

        this.mockMvc.perform(post("/login/google")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"access_token\" : \"Wrong_Access_Token\"}"))
                .andDo(print())
                .andExpect(status().isBadRequest());
    }
```

#### @MockBean에 대해서 자세히 알아 보았고 이를 어떻게 사용하는지에 대해서도 알아 보았으니 테스트할 때 더 날아다닐 수 있을 것 같다!!! 역시 Sproutt스터디가 짱이다!! 화이팅

더 많은 정보를 얻고 싶은 독자는 다음을 참고하면 좋다.

- [https://howtodoinjava.com/spring-boot2/testing/spring-mockbean-annotation/](https://howtodoinjava.com/spring-boot2/testing/spring-mockbean-annotation/)
- [https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html)