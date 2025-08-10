---
layout: post
title: "[SpringBoot] 쿠키와 세션"
date: '2025-08-08 23:35:12 +0900'
description: cookies vs sessions
image: /assets/img/content/2025-08-10/image.png
category: [BootCamp, TIL]
tags: [springboot]
---

웹 개발에서 **쿠키(Cookie)**와 **세션(Session)**은 사용자의 상태를 저장하고 유지하는 핵심 기술이다.
HTTP는 Stateless(상태를 기억하지 않는) 프로토콜이라 요청과 응답이 끝나면 서버는 사용자를 기억하지 않는다. 그래서 쿠키와 세션이 필요한 것이다.

### 1. 쿠키(Cookie)란?

- 정의: 사용자의 브라우저에 저장되는 작은 데이터 조각

- 저장 위치: 클라이언트(브라우저)

- 형태: `key=value` 형태

**예시**:
```
userId=alpha123; expires=Sat, 10 Aug 2025 12:00:00 GMT; path=/; secure; HttpOnly
```

**쿠키 동작 방식**

1. 클라이언트가 서버에 요청

2. 서버가 Set-Cookie 헤더를 응답에 포함해 보냄

3. 브라우저는 해당 쿠키를 저장

4. 이후 같은 도메인에 요청할 때 자동으로 쿠키를 Cookie 헤더에 포함

### 2. 세션(Session)이란?

- 정의: 서버에 저장되는 사용자 정보

- 저장 위치: 서버 (메모리, DB, 캐시 등)

- 식별 방법: 세션 ID(Session ID)를 쿠키나 URL에 담아 전달

**예시**:

```
sessionId=3f0a9d72-ff19-4baf-90c1-9e3ab0ac21f0
```

**세션 동작 방식**

1. 클라이언트가 서버에 요청

2. 서버가 세션 ID를 생성 후 저장소에 사용자 데이터 저장

3. 세션 ID를 쿠키(Set-Cookie)로 클라이언트에 전달

4. 이후 요청에서 브라우저가 세션 ID를 보내면 서버가 해당 사용자를 식별

### 3. 쿠키와 세션 차이

| 구분    | 쿠키                 | 세션                 |
| ----- | ------------------ | ------------------ |
| 저장 위치 | 클라이언트(브라우저)        | 서버                 |
| 보안    | 낮음(데이터가 클라이언트에 있음) | 높음(데이터가 서버에 있음)    |
| 용량    | 최대 약 4KB           | 서버 메모리나 DB 용량에 의존  |
| 속도    | 빠름(서버 부하 적음)       | 느림(서버 부하 있음)       |
| 유지 기간 | 만료 시간까지 유지         | 브라우저 종료 시 기본적으로 삭제 |

### 4. 언제 쿠키를 쓰고 언제 세션을 쓸까?

- 쿠키: 로그인 상태 기억, 자동완성, 다크모드 같은 UI 설정

- 세션: 로그인 인증, 장바구니, 결제 과정 등 민감한 데이터

### 5. 보안 고려 사항

**쿠키**

- HttpOnly: 자바스크립트로 접근 불가

- Secure: HTTPS에서만 전송

**세션**

- 세션 하이재킹 방지 → 세션 ID를 자주 변경

- 세션 타임아웃 설정

### 6. 간단한 코드 예시 (Spring Boot)

``` java
// 쿠키 설정
Cookie cookie = new Cookie("userId", "alpha123");
cookie.setMaxAge(60 * 60); // 1시간
cookie.setHttpOnly(true);
response.addCookie(cookie);

// 세션 설정
HttpSession session = request.getSession();
session.setAttribute("userName", "Alpha");
session.setMaxInactiveInterval(30 * 60); // 30분
```

### 💡 정리

- 쿠키: 클라이언트에 저장 (간단한 정보, 장기 유지 가능)

- 세션: 서버에 저장 (민감한 정보, 보안 중요)

실제 서비스에서는 쿠키+세션을 함께 쓰는 경우가 많다. 예를 들어 로그인 시 쿠키에는 세션 ID만 저장하고 나머지 정보는 서버 세션에 보관한다.