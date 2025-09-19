---
layout: post
title: "[SpringBoot] 웹 애플리케이션 보안 - Spring Security"
date: '2025-08-13 09:57:41 +0900'
description: spring security
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot]
---

웹 개발자라면 반드시 알아야 할 **Spring Security**에 대해 깊이 있게 다루어 본다. Spring Security는 Spring 기반 애플리케이션의 보안을 책임지는 가장 강력하고 표준적인 프레임워크다. 단순히 로그인 기능을 구현하는 것을 넘어 현대 웹 애플리케이션이 직면하는 다양한 보안 위협으로부터 시스템을 보호하는 데 필수적인 역할을 한다.

### 1\. Spring Security의 핵심 개념

Spring Security는 크게 두 가지 핵심 개념을 기반으로 동작한다.

  * **인증(Authentication)**: 사용자가 누구인지, 즉 신원을 확인하는 과정이다. 흔히 말하는 '로그인'이 바로 인증 과정이다.

      * **Principal**: 인증을 시도하는 주체를 나타낸다. 보통 사용자의 '아이디'가 여기에 해당된다.
      * **Credential**: 주체의 신원을 증명하는 정보다. '비밀번호'가 대표적인 예다.
      * Spring Security는 데이터베이스, LDAP, OAuth2 등 다양한 인증 방식을 지원하며, 사용자의 Principal과 Credential을 검증하여 `Authentication` 객체를 생성한다.

  * **인가(Authorization)**: 인증된 사용자가 특정 리소스(예: URL, 메서드)에 접근하거나 특정 작업을 수행할 수 있는 권한이 있는지 확인하는 과정이다.

      * **Role (역할)**: 사용자에게 부여된 추상적인 권한 그룹이다. 예를 들어 `ROLE_ADMIN`이나 `ROLE_USER` 같은 역할이 있다.
      * **Authority (권한)**: 더 세부적이고 구체적인 권한을 나타낸다. 예를 들어 `DELETE_POST`, `READ_COMMENTS`와 같이 특정 행위에 대한 권한을 의미한다. Spring Security에서는 일반적으로 `GrantedAuthority` 인터페이스를 통해 관리된다.

인증이 성공해야 인가 과정으로 넘어갈 수 있으며 인가가 실패하면 **403 Forbidden** 에러가 발생한다. 반대로 인증이 실패하면 **401 Unauthorized** 에러가 발생한다.

### 2\. Spring Security의 동작 원리: 필터 체인(Filter Chain)

Spring Security의 가장 중요한 특징은 **서블릿 필터(Servlet Filter)** 기반으로 동작한다는 점이다. 애플리케이션으로 들어오는 모든 HTTP 요청은 `FilterChainProxy`라는 특별한 필터를 통해 여러 보안 필터들의 연속적인 체인(Chain)을 거치게 된다.

이 필터 체인에는 `UsernamePasswordAuthenticationFilter`, `ExceptionTranslationFilter` 등 다양한 역할을 가진 필터들이 미리 정의된 순서대로 존재한다. 각 필터는 자신의 역할을 수행하며, 요청을 다음 필터로 전달하거나 필요에 따라 요청을 중단하고 응답을 반환한다.

### 3\. 주요 구성 요소

Spring Security의 유연하고 강력한 기능을 가능하게 하는 몇 가지 핵심 인터페이스와 클래스가 있다.

  * **`SecurityContextHolder`**: 인증된 사용자의 정보를 저장하고 있는 `SecurityContext`를 담는 컨테이너다. 기본적으로 `ThreadLocal` 방식을 사용하여 요청이 끝날 때까지 해당 스레드에서 인증 정보를 공유한다.
  * **`AuthenticationManager`**: 인증을 총괄하는 핵심 인터페이스다. 실제 인증 로직은 `AuthenticationProvider`에게 위임하여 처리한다.
  * **`UserDetailsService`**: 인증 과정에서 사용자의 Principal(아이디)을 받아, 해당 사용자의 상세 정보(`UserDetails` 객체)를 로드하는 인터페이스다. 개발자가 직접 구현하여 데이터베이스 등에서 사용자 정보를 가져오도록 할 수 있다.

### 4\. 최신 버전(Spring Boot 3.x)에서의 변화

최신 Spring Security 6.x 버전(Spring Boot 3.x)에서는 설정 방식에 큰 변화가 있었다. 이전의 `WebSecurityConfigurerAdapter`는 더 이상 사용되지 않으며, `SecurityFilterChain` Bean을 직접 등록하는 방식으로 바뀌었다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/public/**").permitAll() // 'public' 경로는 모두 허용
                .anyRequest().authenticated() // 그 외 모든 요청은 인증 필요
            )
            .formLogin(Customizer.withDefaults()); // 기본 폼 로그인 설정 사용

        return http.build();
    }
}
```

이러한 변화는 람다식과 빌더 패턴을 적극적으로 활용하여 설정의 가독성과 유연성을 높인다.

-----

Spring Security를 제대로 이해하고 활용하면 애플리케이션의 보안을 견고하게 구축할 수 있다.