---
layout: post
title: "[SpringBoot] Spring Boot와 Servlet Filter"
date: '2025-08-11 09:50:15 +0900'
description: servlet filter
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot]
---

Spring Boot에서 **필터(Filter)**는 HTTP 요청과 응답을 컨트롤러가 처리하기 전과 후에 가로채서 원하는 로직을 실행할 수 있는 기능이다.  
필터는 Servlet API 기반으로 동작하며, 순수 Java EE 환경에서도 동일하게 사용할 수 있다.

---

## 1. Servlet Filter란?

Servlet Filter는 `javax.servlet` 또는 `jakarta.servlet` 패키지에 포함된 인터페이스로,  
웹 애플리케이션에서 요청과 응답을 가공하거나 흐름을 제어하는 데 사용된다.

### 특징
- 요청이 서블릿(또는 Spring 컨트롤러)에 도달하기 전에 동작
- 요청이나 응답을 변경 가능
- 요청을 아예 차단 가능
- 체인 방식으로 여러 필터를 순서대로 실행 가능

### 생명주기
1. `init(FilterConfig config)` – 애플리케이션 시작 시 필터 초기화
2. `doFilter(ServletRequest request, ServletResponse response, FilterChain chain)` – 필터 핵심 로직 실행
3. `destroy()` – 애플리케이션 종료 시 필터 정리

---

## 2. Servlet Filter 예제

```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import java.io.IOException;

public class MyServletFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        System.out.println("필터 실행: " + req.getRequestURI());

        chain.doFilter(request, response); // 다음 단계로 진행
    }
}
```

### Spring Boot에서 필터 등록

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {
    @Bean
    public FilterRegistrationBean<MyServletFilter> loggingFilter(){
        FilterRegistrationBean<MyServletFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new MyServletFilter());
        registrationBean.addUrlPatterns("/*");
        return registrationBean;
    }
}

```

## 3. Spring Boot에서의 필터

Spring Boot는 기존 Servlet Filter를 그대로 사용하면서도 다음과 같은 편의 기능을 제공한다.

- Bean으로 쉽게 등록 가능

- 실행 순서를 `@Order` 또는 `setOrder()`로 설정 가능

- Spring Security 필터 체인과 함께 동작 가능

## 4. 필터 활용 사례

- 요청/응답 로깅

- CORS 설정

- 인증 및 권한 체크

- 응답 데이터 압축

- 요청 데이터 전처리

이렇게 필터는 **웹 애플리케이션의 입구**에서 흐름을 제어할 수 있는 강력한 도구다.
Spring Boot에서는 간단하게 등록하고 다양한 시나리오에 적용할 수 있기 때문에,
보안, 로깅, 성능 모니터링 등 여러 상황에서 적극적으로 활용할 수 있다.