---
layout: post
title: "[SpringBoot] 쿠키, 세션, JWT 정리"
date: '2025-08-05 15:57:00 +0900'
description: cookie session jwt
image: /assets/img/content/2025-08-05/image.png
category: [BootCamp, TIL]
tags: [springboot]
---

웹 애플리케이션을 개발할 때 피할 수 없는 두 가지 핵심 개념이 있다. 바로 **인증(Authentication)**과 **인가(Authorization)**다. 이 개념을 이해하려면 HTTP 프로토콜이 갖는 Stateless 특성을 반드시 알아야 한다. 이 글에서는 인증·인가의 정의부터 HTTP의 특징, 그리고 상태 유지를 위한 대표적 기법인 쿠키, 세션, JWT까지 차근차근 다룬다.

---

## 1. 인증(Authentication) vs 인가(Authorization)

| 구분 | 설명 | 예시 |
|------|------|------|
| 인증 | 사용자가 누구인지 확인하는 절차 | 로그인, 아이디/비밀번호 검증 |
| 인가 | 인증된 사용자가 어떤 권한을 갖는지 확인하는 절차 | 게시글 삭제 권한 확인 |

>인증이 “누구냐?”를 묻는 단계라면  
 인가는 “이 기능을 사용할 수 있느냐?”를 묻는 단계다.

---

## 2. HTTP의 기본 특성

HTTP는 다음 두 가지 특성으로 설계되었다.

- **Stateless**  
  서버가 각 요청 간에 클라이언트의 상태를 기억하지 않는다.
  
- **Connectionless**  
  요청–응답이 끝나면 연결이 끊어진다.

이 때문에 로그인 상태나 쇼핑 카트 같은 연속적인 사용자 경험을 제공하려면 별도의 상태 관리 기법이 필요하다.

---

## 3. HTTP도 완전 Stateless & Connectionless일까?

사실 HTTP/1.1부터 몇 가지 확장 기능이 도입되며 “완전” Stateless & Connectionless라는 표현이 항상 정확하지는 않다.

### 3-1. keep-alive (Persistent Connection)

- HTTP/1.1 기본 설정으로 TCP 연결을 재사용한다.

**헤더 예시**:  
``` http
  Connection: keep-alive
```

**효과**: 매번 핸드셰이크 없이 같은 TCP 연결로 요청을 전송해 오버헤드를 줄인다.

### 3-2. SSE (Server-Sent Events)

- HTTP 연결을 장시간 유지하며 서버가 클라이언트로 실시간 이벤트를 전송한다.
- **활용 예시**: 채팅, 실시간 알림 등

---

## 4. HTTP 메시지 구조와 쿠키의 위치

HTTP 메시지는 크게 **헤더(Header)**와 **본문(Body)**로 구성된다.  
쿠키는 이 중 **헤더에 포함되어** 교환된다.

---

## 5. 쿠키(Cookie)로 상태 유지하기

쿠키는 HTTP의 Stateless 특성을 보완하기 위해 고안된 가장 오래된 기법이다.

### 5-1. 서버 → 클라이언트: 쿠키 설정

``` http
HTTP/1.1 200 OK
Set-Cookie: session_id=abc123; Path=/Domain=example.com; Secure; HttpOnly; SameSite=Strict
```

- `Secure`: HTTPS 연결에서만 전송된다.  

- `HttpOnly`: 자바스크립트 접근 차단  

- `SameSite`: 동일 사이트 요청에만 쿠키 전송

> **중요**: 보안 관련 설정은 서버에서 반드시 `HttpOnly`를 걸어야 클라이언트 접근을 방지할 수 있다.
{: .prompt-info }

### 5-2. 클라이언트 → 서버: 쿠키 전송

``` http
GET /dashboard HTTP/1.1
Host: example.com
Cookie: session_id=abc123; theme=dark
```


---

## 6. 세션(Session)

### 세션 저장 흐름

- 서버에 사용자 상태(로그인 정보 등)를 저장한다.
- 클라이언트는 식별자(`session_id`)만 쿠키로 전송한다.
- 서버가 매 요청마다 해당 식별자로 상태를 조회한다.

세션 방식은 **서버 메모리나 데이터베이스에 상태(state)를 저장**하므로 **Stateful**하다.

---

## 7. JWT (JSON Web Token)

JWT는 세션의 반대 개념으로, 상태를 **서버가 아닌 클라이언트가 토큰 형태로 보유**한다.  
서버는 토큰의 유효성만 검증하고, 별도의 상태 저장소가 필요 없다.

### 7-1. JWT 구조

- **Header**
- **Payload (Claims)**
- **Signature**

각 부분은 Base64URL로 인코딩되어 **헤더-페이로드-서명** 형태의 문자열을 이룬다.

### 7-2. Base64URL vs Base64

- Base64URL: `+` → `-`, `/` → `_`, `=` 제거  
- 패딩(`=`) 제거는 Base64 특성(결과 길이가 4의 배수)으로 가능

> **페이로드**는 인코딩일 뿐 암호화가 아니다. **민감한 정보**는 절대로 포함해서는 안 된다.
{: .prompt-warning }
---

## 8. 인증 상태 저장 방식 비교

| 특징 | 세션(Session) | JWT (Stateless) |
|------|----------------|------------------|
| 저장 위치 | 서버 메모리/DB | 클라이언트(토큰) |
| 상태 | Stateful | Stateless |
| 서버 부하 | 상태 저장으로 증가 | 토큰 검증만 하므로 감소 |
| 확장성 | 세션 동기화 필요 | 별도 저장소 없이 확장 용이 |
| 보안 | 서버 권한 관리 가능 | 토큰 탈취 시 위험 |

---

## 마무리 및 실전 팁

- **인증**은 사용자가 **누구인지 확인**한다.
- **인가는** 사용자가 **무엇을 할 수 있는지 결정**한다.
- HTTP는 기본적으로 **상태를 유지하지 않으므로**, 쿠키, 세션, JWT 같은 외부 기법을 활용해야 한다.
- 보안을 위해 항상 **Least Privilege 원칙**을 적용하고, **민감 정보는 서버에만 저장**해야 한다.

