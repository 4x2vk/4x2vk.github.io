---
layout: post
title: "[SpringBoot] 독립 실행형 서블릿 애플리케이션: Containerless로 된다"
date: '2025-08-04 20:34:54 +0900'
description: servlet application
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot, project, troubleshooting]
---

웹 서버 설치나 배포 없이 코드만으로 톰캣을 띄우고 서블릿을 등록하는 방법을 설명한다. Spring 없이도 충분히 된다!

### 1. Spring Boot 지우면 된다

- @SpringBootApplication 지우면 된다

- SpringApplication.run(...); 지우면 된다

- public static void main(String[] args) { /* 여기에 코드 쓰면 된다 */ }만 남기면 된다

### 2. 내장 톰캣 띄우면 된다

``` java
ServletWebServerFactory factory = new TomcatServletWebServerFactory();
WebServer server = factory.getWebServer();
server.start();
```
- spring-boot-starter-web만 넣으면 내장 톰캣 라이브러리가 자동 포함된다

- 위 세 줄만으로 톰캣이 켜진다

- 번거로운 설치나 설정이 필요 없다

### 3. 서블릿 등록하면 된다

``` java
ServletWebServerFactory factory = new TomcatServletWebServerFactory();
factory.getWebServer(servletContext -> {
    ServletRegistration.Dynamic reg = 
        servletContext.addServlet("helloServlet", new HelloServlet());
    reg.addMapping("/hello");
}).start();
```
- ServletContextInitializer를 람다로 구현하면 된다

- addServlet + addMapping으로 URL 매핑이 간단하게 된다

### 4. HttpServlet 작성하면 된다

``` java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req,
                         HttpServletResponse resp) throws IOException {
        String name = req.getParameter("name");
        String message = "Hello, " + (name != null ? name : "World");
        resp.setStatus(HttpServletResponse.SC_OK);
        resp.setContentType("text/plain");
        resp.getWriter().println(message);
    }
}
```
- HttpServlet만 상속하면 된다

- doGet()에 요청 처리 로직을 넣으면 된다

- 응답 상태, 헤더, 본문까지 모두 직접 제어 가능하다

### 5. 매핑·바인딩 공통화하면 된다

- URI와 메서드 비교로 매핑 처리하면 된다

- 파라미터 추출 후 바인딩만 공통화하면 된다

- 반복 코드를 리플렉션이나 어노테이션으로 어쩌면 더 편해진다