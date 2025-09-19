---
layout: post
title: "[Project] Spring Advanced 트러블슈팅"
date: '2025-09-01 13:44:46 +0900'
description: spring advanced troubleshooting
image: /assets/img/content/2025-09-01/image.png
category: [BootCamp, TIL]
tags: [springboot, project, troubleshooting]
---

> 이 프로젝트는 Spring Boot 기반의 백엔드 서비스로, 초기 설정부터 테스트까지 다양한 이슈가 발생했다. 아래는 발생한 문제들과 그 해결 방법이다.
{: .prompt-tip }
---

### 애플리케이션 실행 오류

**문제**: 프로젝트 실행 시 애플리케이션이 시작되지 않음  
**원인**:  
- `application.yml` 누락 또는 JWT 키 미설정  
- 의존성 충돌 또는 누락된 라이브러리  
- Bean 주입 실패  

**해결 방법**:  
- `src/main/resources/application.yml`에 JWT 키를 설정
  ```yaml
  jwt:
    secret: 
       key: your-secret-key
  ```
- Gradle 의존성을 다시 확인하고 `./gradlew clean build` 실행

---

### ArgumentResolver 동작 안 함

**문제**: `AuthUserArgumentResolver`가 정상적으로 작동하지 않음  
**해결 방법**:  
- `WebMvcConfigurer`를 구현한 클래스에서 `addArgumentResolvers()` 메서드에 `AuthUserArgumentResolver`를 등록했는지 확인
- `@Component` 또는 `@Bean`으로 등록되어 있는지 확인

---

### 비즈니스 로직 오류

**문제**: `signup()` 메서드에서 불필요한 `passwordEncoder.encode()` 실행  
**해결 방법**:  
- 이메일 중복 여부를 먼저 검사하고, 조건에 맞지 않으면 즉시 리턴하는 방식으로 리팩토링

---

### N+1 문제 발생

**문제**: `TodoService.getTodos()`에서 N+1 문제가 발생  
**해결 방법**:  
- 기존 JPQL `fetch join` 대신 `@EntityGraph`를 사용하여 연관 데이터를 한 번에 조회
  ```java
  @EntityGraph(attributePaths = {"user"})
  List<Todo> findByUserId(Long userId);
  ```

---

### 테스트 코드 실패

**문제**: 테스트가 실패하거나 의도한 예외를 던지지 않음  
**해결 방법**:  
- 테스트 메서드명이 실제 동작과 일치하는지 확인
- `@Test` 어노테이션과 `assertThrows()` 또는 `assertTrue()` 등의 검증 로직이 정확한지 점검
- 예외 타입이 변경되었을 경우, 테스트 코드도 함께 수정

---

### 비밀번호 유효성 검사

**문제**: `changePassword()`에서 유효성 검사가 서비스 로직에 포함되어 있음  
**해결 방법**:  
- DTO 클래스에서 `@Size`, `@Pattern`, `@NotBlank` 등의 어노테이션을 사용하여 유효성 검사 수행
- `@Valid` 어노테이션을 컨트롤러에서 사용하여 자동 검증

### 참고 자료

- 🔗 [Github 프로젝트](https://github.com/4x2vk/spring-advanced)