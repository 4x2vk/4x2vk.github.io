---
layout: post
title: "[Project] 일정 관리 앱 Develop 트러블슈팅"
date: '2025-08-14 11:47:16 +0900'
description: schedule api develop troubleshooting
image: /assets/img/content/2025-08-14/image.png
category: [BootCamp, TIL]
tags: [springboot, project, troubleshooting]
---

> 오늘은 업그레이드된 일정 관리를 위한 **Schedule API Develop** 프로젝트를 진행하면서 배우고 경험한 내용을 정리해본다.
{: .prompt-tip }

### Session 기반 **로그인/회원가입/스케줄 관리**

#### 1\. ERD 설계 과정

![image1](/assets/img/content/2025-08-14/image1.png){: .shadow }

  * 처음에는 `User` 테이블과 `Schedule` 테이블을 분리하여 설계했다.
  * 이후 두 엔티티를 **1:N 관계**로 연결했다. `Schedule` 엔티티에 `user_id`를 외래 키로 저장하여, 어떤 사용자가 어떤 스케줄을 만들었는지 추적 가능하게 했다.

![image2](/assets/img/content/2025-08-14/image2.png){: .shadow }

-----

#### 2\. 로그인 기능 1차 완성 - 비밀번호 **검증 누락** 문제

![image3](/assets/img/content/2025-08-14/image3.png){: .shadow }

로그인 기능을 완성했으나 이메일만으로 로그인되는 보안 취약점을 발견했다. 비밀번호 검증 로직이 빠져있었기 때문이다.

  * **해결**:
    ```java
    if (request.getPassword() == null) {
        throw new InvalidCredentialException("Password must be provided");
    }
    ```
    위 코드를 추가하여 비밀번호 입력 여부를 먼저 체크하도록 수정했고 비밀번호가 없으면 **401 Unauthorized** 오류가 반환되도록 만들었다.

![image4](/assets/img/content/2025-08-14/image4.png){: .shadow }

-----

#### 3\. 문자열 비교로 인한 **로그인 실패**

![image5](/assets/img/content/2025-08-14/image5.png){: .shadow }

비밀번호를 `String.equals()`로 직접 비교했는데 이는 보안상 매우 위험했다. 데이터베이스에 암호화되지 않은 비밀번호(평문)가 저장되어 있었기 때문이다.

  * **문제 코드**:
    ```java
    if(!ObjectUtils.nullSafeEquals(user.getPassword(), request.getPassword())) {
        throw new InvalidCredentialException("Password mismatch");
    }
    ```
  * **해결**: `spring-security-crypto`의 `BCryptPasswordEncoder`를 도입했다.
      * **회원가입**: 비밀번호를 해시 암호화하여 저장하도록 수정.
      * **로그인**: `passwordEncoder.matches(raw, encoded)` 메서드를 사용하여 평문 비밀번호와 해시된 비밀번호를 안전하게 비교하도록 수정.

-----

#### 4\. Spring Security 도입 후 회원가입 **401** 에러

![image6](/assets/img/content/2025-08-14/image6.png){: .shadow }

Spring Security를 추가하자 기본 설정에 의해 모든 요청이 인증 필요 상태가 되어 회원가입 요청까지 차단되는 문제가 발생했다.

  * **해결**: 특정 경로를 인증 없이 접근할 수 있도록 **`permitAll()`** 설정을 추가했다.
    ```java
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/", "/users/signup", "/login", "/users/logout").permitAll()
        .anyRequest().authenticated())
    ```
    `/signup`, `/login`, `/logout` 등 필요한 경로들을 화이트리스트로 지정하여 해결했다.

-----

#### 5\. 로그아웃 **403** 에러

![image7](/assets/img/content/2025-08-14/image7.png){: .shadow }

`@PostMapping("/logout")`으로 로그아웃 기능을 구현했더니 **403 Forbidden** 오류가 발생했다. Spring Security에 이미 GET 요청 방식의 `/logout`이 내장되어 있어 경로와 HTTP 메서드가 충돌한 것이 원인이었다.

  * **해결**:
    ```java
    @PostMapping("/users/logout")
    ```
    API 경로를 `/users/logout`으로 변경하여 충돌을 피하고, 정상적으로 세션을 `invalidate()`할 수 있었다.

-----

### 최종 **ERD**

![image8](/assets/img/content/2025-08-14/image8.png){: .shadow }

### 배운 점

  * **로그인 로직**: 항상 비밀번호 검증부터 시작해야 한다. 이메일만으로 인증이 되면 치명적인 보안 취약점이 된다.
  * **비밀번호 저장**: 평문 저장은 절대 금지\! `BCrypt`와 같은 해시 함수를 사용해야 한다.
  * **Spring Security 설정**: 기본적으로 모든 요청을 차단하므로 필요한 경로는 `permitAll()`로 예외 처리해야 한다.
  * **API 경로 충돌**: Spring Security가 제공하는 기본 기능(예: `/logout`)과 커스텀 API의 경로 및 HTTP 메서드가 충돌하지 않도록 주의해야 한다.
  * **ERD 설계**: 처음부터 엔티티 간의 연관관계를 명확히 설계하면 이후 쿼리 설계와 API 권한 관리가 훨씬 수월해진다.


**참고 자료**

- 🔗 [Github 프로젝트](https://github.com/4x2vk/ScheduleApiDevelop)

- 🔗 [API Docs](https://documenter.getpostman.com/view/47183182/2sB3BGHVTr)

- 🔗 [ERD Diagram](https://github.com/4x2vk/ScheduleApiDevelop/issues/1#issuecomment-3186705590)