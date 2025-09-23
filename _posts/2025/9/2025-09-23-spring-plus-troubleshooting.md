---
layout: post
title: "[Project] Spring Plus 트러블슈팅"
date: '2025-09-23 13:23:03 +0900'
description: spring plus troubleshooting
image: /assets/img/content/2025-09-23/image.png
category: [BootCamp, TIL]
tags: [springboot, project, troubleshooting]
---

## 문제 해결 과정

### 문제 1: 회원가입 시 nickname 필드 검증 오류

``` bash
2025-09-16T11:24:44.654+09:00  WARN 24552 --- [expert] [nio-8080-exec-4] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.example.expert.domain.auth.dto.response.SignupResponse org.example.expert.domain.auth.controller.AuthController.signup(org.example.expert.domain.auth.dto.request.SignupRequest): [Field error in object 'signupRequest' on field 'nickname': rejected value [null]; codes [NotBlank.signupRequest.nickname,NotBlank.nickname,NotBlank.java.lang.String,NotBlank]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [signupRequest.nickname,nickname]; arguments []; default message [nickname]]; default message [공백일 수 없습니다]] ]
```

오류 로그에서 `SignupRequest`의 `nickname` 필드가 `null` 값을 받았지만 `@NotBlank` 어노테이션으로 인해 검증에 실패한 것을 확인할 수 있습니다. 이는 프론트엔드에서 `nickname` 필드를 요청에 포함하지 않았음을 의미합니다.

**문제점:** 프론트엔드에서 회원가입 요청 시 nickname 필드를 전송하지 않았지만, 해당 필드가 `@NotBlank`로 필수값으로 설정되어 있었습니다.

**해결방법:**
1. `SignupRequest`의 `nickname` 필드에서 `@NotBlank` 어노테이션 제거
2. `AuthService`에 fallback 로직 추가:
   - nickname이 제공되지 않거나 빈 값일 경우
   - email을 기본 nickname으로 사용하도록 설정

### 문제 2: TodoControllerTest 테스트 실패

![image1](/assets/img/content/2025-09-23/image2.png){: .shadow }

**문제점:** `TodoControllerTest`에서 예상했던 상태 코드 대신 400 Bad Request가 반환되는 문제가 발생했습니다.

**원인 분석:**
1. 테스트에 필요한 설정들(`WebConfig`, `GlobalExceptionHandler`)이 포함되지 않음
2. 테스트에서 200 OK 상태를 예상했지만 `InvalidRequestException`은 400 BAD_REQUEST를 반환함

**해결방법:**
1. 필요한 임포트 및 설정 추가:
   ```java
   @Import({WebConfig.class, GlobalExceptionHandler.class})
   ```

2. 테스트에서 예상하는 상태 코드 수정:
   ```java
   // 수정 전: .andExpected(status().isOk())
   // 수정 후: .andExpected(status().isBadRequest())
   ```

3. JSON 응답의 예상 값들 업데이트:
   ```java
   .andExpected(jsonPath("$.status")
   .value(HttpStatus.BAD_REQUEST.name()))
   
   .andExpected(jsonPath("$.code")
   .value(HttpStatus.BAD_REQUEST.value()))
   ```

### 기능 구현 1: JPA Cascade를 활용한 자동 담당자 등록

**구현 내용:**

1. **JPA Cascade 설정**
   ```java
   @OneToMany(mappedBy = "todo", cascade = CascadeType.PERSIST)
   private List<Manager> managers = new ArrayList<>();
   ```
   - `CascadeType.PERSIST`: Todo가 저장될 때 연관된 Manager도 함께 저장됩니다.

2. **담당자 추가 메서드**
   ```java
   public void addManager(User user) {
       Manager manager = new Manager(user, this);
       this.managers.add(manager);
   }
   ```
   - Todo 엔티티에 담당자를 추가하는 메서드를 제공합니다.

3. **서비스 로직 수정**
   ```java
   // 할 일을 생성한 유저를 담당자로 자동 등록
   newTodo.addManager(user);
   ```
   - Todo 생성 후 생성한 유저를 담당자로 자동 등록합니다.

### 문제 3: N+1 쿼리 문제 해결

![image2](/assets/img/content/2025-09-23/image3.png){: .shadow }

**문제점:** 현재 `CommentRepository`의 쿼리에서 N+1 문제가 발생하고 있습니다. `findByTodoIdWithUser` 쿼리에서 `JOIN c.user`만 사용하고 있어서 User 엔티티가 제대로 페치되지 않습니다.

**해결방법:**

- **수정 전 코드**
  ```java
  @Query("SELECT c FROM Comment c JOIN c.user WHERE c.todo.id = :todoId")
  List<Comment> findByTodoIdWithUser(@Param("todoId") Long todoId);
  ```

- **수정 후 코드**
  ```java
  @Query("SELECT c FROM Comment c JOIN FETCH c.user WHERE c.todo.id = :todoId")
  List<Comment> findByTodoIdWithUser(@Param("todoId") Long todoId);
  ```

**개선 효과:**
- `JOIN FETCH`를 사용하여 Comment와 연관된 User 엔티티를 한 번의 쿼리로 함께 조회
- N+1 쿼리 문제를 완전히 해결하여 데이터베이스 호출 횟수를 대폭 감소
- 애플리케이션 성능 향상 및 응답 속도 개선