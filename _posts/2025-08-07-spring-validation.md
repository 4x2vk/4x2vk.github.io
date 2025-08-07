---
layout: post
title: "[SpringBoot] Validation"
date: '2025-08-07 23:09:43 +0900'
description: spring validation
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot]
---

### @Valid vs @Validated

| 구분 | 설명 | 특징 |
| --- | --- | --- |
| @Valid | javax.validation 기반 단순 검증 | 그룹 기능 없음 |
| @Validated | Spring 기반 확장 기능 제공 (validation 그룹 지원) | 그룹별 조건 가능 |

### 사용 예시

build.gradle

```java
 implementation 'org.springframework.boot:spring-boot-starter-validation'
```

```java
@RestController
@RequestMapping("/members")
public class MemberController {
    @PostMapping
    public ResponseEntity<String> register(@Valid @RequestBody MemberDto dto) {
        return ResponseEntity.ok("등록 완료");
    }
}
```

```java
@Getter
public class MemberDto {
    @NotBlank(message = "이름은 필수입니다")
    private String name;

    @Email(message = "올바른 이메일 형식이어야 합니다")
    private String email;
}
```

### 유효성 검증 실패 시 예외

- `MethodArgumentNotValidException` 발생
- 이를 `@ControllerAdvice`에서 핸들링 가능

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<String> handleValidation(MethodArgumentNotValidException ex) {
    String msg = ex.getBindingResult().getAllErrors().get(0).getDefaultMessage();
    return ResponseEntity.badRequest().body("입력값 오류: " + msg);
}

```

---

## 유용한 유효성 검증 어노테이션

| 어노테이션                 | 설명                                      |
|--------------------------|-------------------------------------------|
| `@NotNull`               | null이 아니어야 함                        |
| `@NotEmpty`              | null 또는 빈 문자열이 아니어야 함         |
| `@NotBlank`              | null, 빈 문자열, 공백 불허                |
| `@Size(min, max)`        | 문자열/컬렉션의 길이 범위 지정            |
| `@Pattern`               | 정규표현식 일치 여부 검사                 |
| `@Email`                 | 이메일 형식 검사                          |
| `@Min` / `@Max`          | 최소 / 최대 숫자 값 지정                  |
| `@Positive` / `@Negative`| 양수 / 음수 검사                          |
