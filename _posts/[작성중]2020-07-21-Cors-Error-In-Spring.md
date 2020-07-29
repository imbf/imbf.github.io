---
layout: post
title:  "Cors Error In Spring"
date:   2020-07-21
author: Green Frog Developer
categories: Spring
tags: Spring Jpa Serialize Infinite Recursion
---


Spring MVC Cors 적용
```java
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedOrigins("http://webper.net:3000")
                        .allowedMethods("*")
                        .allowCredentials(false)
                        .maxAge(3000);
            }
        };
```

Spring Security Cors 적용
```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .httpBasic().disable()
                .csrf().disable();

        http
                .authorizeRequests()
                .antMatchers("/login/google").anonymous()
                .anyRequest().authenticated()
                .and()
                .cors()
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        configuration.addAllowedOrigin("http://www.webper.net:3000");
        configuration.addAllowedHeader("*");
        configuration.addAllowedMethod("*");
        configuration.setAllowCredentials(false);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
```



spring mvc 의 CORS 를 사용하고 있는 경우, 기존 mvc config 에 세팅된 configuration 들을 활용하기 위해서
spring security 레벨에서 enable cors 만 해주면 mvc config 에 세팅된 구성 정보들을 security 에서 활용할 수 있다는 말이지?

<img src="https://files.slack.com/files-pri/TG2QNEQN7-F017SCP0Q67/image.png">
