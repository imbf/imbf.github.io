---
layout: post
title:  "Spring Security + JWT를 통해 프로젝트에 인증 구현하기"
date:   2020-06-29
author: Green Frog Developer
categories: Spring
tags: Spring Security JWT 
---

Spring Security와 JWT를 활용해서 프로젝트에 인증을 어떻게 구현했는지에 대해서 포스팅 하려고 한다.

토큰을 사용한 인증은 처음이기도 하고, Google OAuth를 함께 사용하는 바람에... 개념이 해깔려 며칠간 계속 해맸던 것 같다.

이제서야 조금 감을 잡으며 프로젝트에 인증 구현을 마쳤는데.. 아직 구현되지 않은 기능들이 매우 많다 ㅎㅎ 언능 구현해야 겠다!!!

**서론이 길었다!!! 지금부터 Spring Security란 무엇이고, Spring Security와 JWT를 사용해서 어떻게 인증을 구현했는지에 대해서 포스팅 하겠다.**

---

## Spring Security란?

---

**Spring Security란 인증과, 권한부여, 일반적인 공격에 대한 보호의 기능을 제공하는 프레임워크이다.** 즉, Spring Security를 사용하면 보안 관련 처리를 자체적으로 구현 할
필요 없이 쉽고 안전하게 필요한 기능을 구현할 수 있다.

Spring Security의 Servlet 보안 지원은 **Servlet Filter(이하 Filter)**를 기반으로 한다. 클라이언트가 어플리케이션으로 request를 보내면, Container는 요청 URI의 경로에 따라 어떤 Filter와 어떤 Servlet을 적용할 것인지 결정한다.

Spring은 여러 Filter중 Servlet Container lifecycle과 ApplicationContext사이에 bridging할 수 있는 `DelegatingFilterProxy`라는 Filter를 제공한다.

Spring Security의 Servlet 보안 지원은 `DelegatingFilterProxy`가 감싸고 있는 **`FilterChainProxy`**에 의해 수행되며, `FilterChainproxy`는 **`SecurityFilterChain`**을 통해 많은 쟉업을 **Security Filter** 인스턴스에 위임한다.

위 과정을 그림으로 나타내면 다음과 같다.

<img src="/assets/spring/Spring-Security-With-JWT-1.png" style="width:70%">

SecurityFilterChain은 스프링에서 보안과 관련된 여러 Security Filter List를 갖고 있는 객체로 Security Filter List를 순회하면서 필터링을 실시한다.

SecurityFilterChain에 존재하는 Security Filter순서는 다음과 같다.

1. ChannelProcessingFilter
2. ConcurrentSessionFilter
3. WebAsyncManageIntegrationFilter
4. SecurityContextPersistenceFilter
5. HeaderWriterFilter
6. CorsFilter
7. CsrfFilter
8. LogoutFilter
9. OAuth2AuthorizationRequestRedirectFilter
10. Saml2WebSsoAuthenticationRequestFilter
11. X509AuthenticationFilter
12. AbstractPreAuthenticatedProcessingFilter
13. CasAuthenticationFilter
14. OAuth2LoginAuthenticationFilter
15. Saml2WebSsoAuthenticationFilter
16. **UsernamePasswordAuthenticationFilter** 
17. ConcurrentSessionFilter

**... (총 33개의 Spring Security Filter가 존재한다.)**

위의 Security Filter중 **Security Authentication Filter**(UsernamePasswordAuthentication, OAuth2LoginAuthenticationFilter와 같은 인증 필터)는 `AuthenticationManager`를 통해 인증을 수행한다.

**AuthenticationManager는 어떻게 Spring Security의 필터들이 인증을 수행하는지에 대한 명세를 정의해 놓은 인터페이스이다.**
이 `AuthenticationManager`는 일반적으로 **`ProviderManager`**로 구현되며, `ProviderManager`는 여러 **`AuthenticationProvider`**에 인증을 위임한다.

`ProviderManager`에 설정된 `AuthenticationProvider`중 어느 것도 성공적으로 인증을 수행할 수 없다면, 인증은 실패할 것이고 알맞는 예외가 ProviderManager에게 건내질 것이다.

**여러 `AuthenticationProvider`중 하나라도 인증에 성공한다면 `ProviderManager`에게 인증된 `Authentication`객체를 반환하고 이는 event 기반으로 `AuthenticationFilter`에 전송된다.**

**`AuthenticationFilter`는 `SecuritycontextHolder`의 `SecurityContext`에 인증된 Authentication을 저장할 수 있도록 한다.**

### 아래의 그림은 Username and Password 인증 방식의 아키텍처이다.

<img src="/assets/spring/Spring-Security-With-JWT-2.png" style="width:100%">

위 그림의 `AuthenticationFilter`의 역할은 `UsernamePasswordAuthenticationFilter`가 수행하고 **전체적인 프로세스**는 다음과 같다.

1. Client가 어플리케이션에 요청을 보내면, Servlet Filter에 의해서 Security Filter로 Security 작업이 위임되고 여러 Security Filter Chain 중에서 `UsernamePasswordAuthenticationFilter`**(Username and Password Authentication 방식에서 사용하는 AuthenticationFilter)**에서 인증을 처리한다.

2. `AuthenticationFilter`**(UsernamePasswordAuthenticationFilter인데 지금부터 AuthenticationFilter라고 부름)**는 요청에서 username과 password를 추출해 `UsernameAuthenticationToken`**(이하 인증 객체)**을 생성한다.

3. `AuthenticationFilter`는 `AuthenticationManager`**(구현체 : ProviderManager)**에게 인증 객체를 전달한다.

4. `ProviderManager`는 인증을 위해 `AuthenticationProvider`에게 인증 객체를 전달한다.

5. `AuthenticationProvider`는 전달받은 인증 객체의 정보(일반적으로 사용자 아이디)를 `UserDetailsService`에 넘겨준다.

6. `UserDetailsService`는 전달 받은 사용자 정보를 통해 DB에서 알맞는 사용자를 찾고 이를 기반으로 `UserDetails`객체를 만듭니다.

7. 사용자 정보와 일치하는 `UserDetails`객체를 `AuthenticationProvider`에 전달합니다.

8. `AuthenticationProvider`은 전달받은 `UserDetails`를 인증해 성공하면 `ProviderManager`에게 권한을 담은 검증된 인증 객체를 전달합니다.

9. `ProviderManager`는 검증된 인증 객체를 `AuthenticationFilter`에게 전달합니다. **(event 기반)**

10. `AuthenticationFilter`는 검증된 인증 객체를 `SecurityContextHolder`의 `SecurityContext`에 저장합니다.

Spring Security 전체를 설명하기에는 하나의 포스팅으로 부족하기 때문에 아주 기본적인 개념만 설명했고, 이제 메인 주제인 **나의 프로젝트에 어떻게 Spring Security + JWT를 적용**했는지에 대해서 알아보도록 하겠다.

---

## Spring Security + JWT를 프로젝트에 어떻게 적용 했는가?

---

클라이언트가 Spring Security를 적용한 어플리케이션에 리소스를 요청할 때 접근 권한이 없는 경우 기본적으로 **Username and Password Authentication Mechanism**을 사용해 **로그인 폼**으로 보내지게 되는데 그 역할을 하는 필터는 `UsernamePasswordAuthenticationFilter`이다.

내가 개발하고 있는 Rest API Server는 **Username and Password Authentication Mechanism**을 사용하지 않을 것이기 때문에 `UsernamePasswordAuthenticationFilter`이전에 사용자 정의 필터인 `JwtAuthenticationFilter`에서 인증 및 권한처리가 필요했다.

그래서 **`JwtAuthenticationFilter`**를 다음과 같이 만들어서 **`UsernamePasswordAuthenticationFilter`**이전에 등록했다.

**JwtAuthenticationFilter.java** : Jwt가 유효한 토큰인지 인증하기 위한 Filter이다. 
```java
public class JwtAuthenticationFilter extends GenericFilterBean {

    private JwtTokenProvider jwtTokenProvider;

    public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    // Request로 들어오는 Jwt Token의 유효성을 검증하는 filter를 filterChain에 등록합니다.
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        String token = jwtTokenProvider.resolveToken((HttpServletRequest) servletRequest);
        if (token != null && jwtTokenProvider.validateToken(token)) {   // token 검증
            Authentication auth = jwtTokenProvider.getAuthentication(token);    // 인증 객체 생성
            SecurityContextHolder.getContext().setAuthentication(auth); // SecurityContextHolder에 인증 객체 저장
        }
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

Spring Security Filter와 통합하지 않고 **사용자가 정의한 필터(JwtAuthenticationFilter)에서 인증 및 권한 작업을 진행**할 것이기 때문에 `AuthenticationManager`를 사용하지 않고 `JwtTokenProvider`를 통해서 인증 후 `SecurityContextHolder`를 바로 사용했다.

> **Spring Security Document의 관련 글**
>
> If you are not integrating with `Spring Security’s Filters` you can set the `SecurityContextHolder` directly and are not required to use an `AuthenticationManager`.

다음으로는 생성한 `JwtAuthenticationFilter`를 Spring Security의 `UsernamePasswordAuthenticationFilter`**이전에 등록하는 설정**을 할 것이다.

**SecurityConfig.java** : Spring Security 관련 설정들을 하는 Configuration 클래스
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final JwtTokenProvider jwtTokenProvider;

    public SecurityConfig(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .httpBasic().disable()
                .csrf().disable();

        http
                .authorizeRequests()
                    .antMatchers( "/login/google").anonymous()
                    .anyRequest().authenticated()
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);
    }
}
```

Filter를 등록 했으면 Jwt Token을 생성하고, 인증 및 권한 부여 등의 기능을 제공할 **Provider**를 만들어야 한다.

**JwtTokenProvider.java** : Jwt Token을 생성, 인증, 권한 부여, 유효성 검사, PK 추출 등의 다양한 기능을 제공하는 클래스
```java
@Component
public class JwtTokenProvider {

    private final long TOKEN_VALID_MILISECOND = 1000L * 60 * 60 * 10; // 10시간

    @Value("spring.jwt.secret")
    private String secretKey;

    private final UserDetailsService userDetailsService;

    public JwtTokenProvider(@Qualifier("userUserDetailsService") UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes());
    }

    // Jwt 토큰 생성
    public String createToken(String userPk, List<UserRole> roles) {
        Claims claims = Jwts.claims().setSubject(userPk);
        claims.put("roles", roles);
        Date now = new Date();
        return Jwts.builder()
                .setClaims(claims) // 데이터
                .setIssuedAt(now)   // 토큰 발행 일자
                .setExpiration(new Date(now.getTime() + TOKEN_VALID_MILISECOND)) // 만료 기간
                .signWith(SignatureAlgorithm.HS512, secretKey) // 암호화 알고리즘, secret 값 
                .compact(); // Token 생성
    }

    // 인증 성공시 SecurityContextHolder에 저장할 Authentication 객체 생성
    public Authentication getAuthentication(String token) {
        UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUserPk(token));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    // Jwt Token에서 User PK 추출
    public String getUserPk(String token) {
        return Jwts.parser().setSigningKey(secretKey)
                .parseClaimsJws(token).getBody().getSubject();
    }

    public String resolveToken(HttpServletRequest req) {
        return req.getHeader("X-AUTH-TOKEN");
    }

    // Jwt Token의 유효성 및 만료 기간 검사
    public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwtToken);
            return !claims.getBody().getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }

}
```

인증 기능을 수행할 JwtTokenProvider를 만들었으면 JwtTokenProvider가 제공한 사용자 정보로 UserDetails를 제공할 `UserDetailsService`를 만들어야 한다.

**UserDetailsService.java :** JwtTokenProvider가 제공한 사용자 정보로 DB에서 알맞은 사용자 정보를 가져와 UserDetails 생성

```java
@Service
public class UserUserDetailsService implements UserDetailsService {

    private final UserService userService;

    public UserUserDetailsService(UserService userService) {
        this.userService = userService;
    }

    //
    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return userService.findUserByEmail(email);
    }
}
```

UserDetailsService까지 만들었으면 UserDetails를 상속받는 **도메인 객체**를 만들고 사용자를 식별할 JwtToken을 생성할 Controller까지 만들면 Production 코드는 끝이다.

**(User 클래스와 Controller 클래스의 코드를 첨부하고 싶었으나, 많은 배경지식이 필요한 코드들이 존재해 첨부하지 않았다.)**

그렇다면 위의 생성된 코드들을 테스트 할 수 있는 Test Code를 만들면 다음과 같다.

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

    @Autowired
    JwtTokenProvider jwtTokenProvider;

    @Test
    public void GoogleLoginWithWrongAccessToken() throws Exception {
        // When & Then
        when(googleLoginService.authenticate(eq("Wrong_Access_Token")))
                .thenReturn(ResponseEntity.badRequest().build());

        // Then
        this.mockMvc.perform(post("/login/google")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"access_token\" : \"Wrong_Access_Token\"}"))
                .andDo(print())
                .andExpect(status().isBadRequest());
    }

    @Test
    public void GoogleLoginWithRightAccessToken() throws Exception {
        GoogleUserinfoDTO googleUserinfoDTO = new GoogleUserinfoDTO();
        googleUserinfoDTO.setName("frog");
        googleUserinfoDTO.setEmail("frog@email.com");
        ResponseEntity<GoogleUserinfoDTO> responseEntity = ResponseEntity.ok(googleUserinfoDTO);

        // When
        when(googleLoginService.authenticate(eq("Right_Access_Token")))
                .thenReturn(responseEntity);

        MvcResult mvcResult = this.mockMvc.perform(post("/login/google")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"access_token\" : \"Right_Access_Token\"}"))
                .andDo(print())
                .andExpect(status().isOk())
                .andReturn();

        String contentAsString = JsonPath.read(mvcResult.getResponse().getContentAsString(), "$.X-AUTH-TOKEN");
        assertThat(jwtTokenProvider.getUserPk(contentAsString)).isEqualTo("frog@email.com");

        mockMvc.perform(get("/test")
            .header("X-AUTH-TOKEN",contentAsString))
        .andDo(print());
    }
}

```

이를 돌려보면 적절한 Jwt Token으로 인증을 완료하고 SecurityContextHolder에서 알맞는 사용자의 **Principal**을 가져오는 걸 볼 수 있다.


<img src="/assets/spring/Spring-Security-With-JWT-3.png" style="width:100%">

글이 아주 길고 장황해 독자들이 이를 읽고 얼마나 많은 도움이 될려는 지는 모르겠지만, Spring Security 아키텍처 부터 프로젝트 적용까지 한 포스팅으로 전부 설명한다는 것이 정말 쉽지 않은 일이었던 것 같고 아주 좋은 경험인 것 같다.

또한 나의 지식이 늘어감에 또는 나의 프로젝트가 더 커짐에 따라서 지속적으로 업데이트 해야 겠다.

---

참고한 사이트

- [https://docs.spring.io/spring-security/site/docs/5.3.3.BUILD-SNAPSHOT/reference/html5/](https://docs.spring.io/spring-security/site/docs/5.3.3.BUILD-SNAPSHOT/reference/html5/)
- [https://daddyprogrammer.org/post/636/springboot2-springsecurity-authentication-authorization/](https://daddyprogrammer.org/post/636/springboot2-springsecurity-authentication-authorization/)
- [https://jeong-pro.tistory.com/205](https://jeong-pro.tistory.com/205)
- [https://www.baeldung.com/spring-security-custom-filter](https://www.baeldung.com/spring-security-custom-filter)