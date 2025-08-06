---
layout: post
title: "[SpringBoot] DI와 테스트, 애노테이션 활용"
date: '2025-08-06 20:08:07 +0900'
description: di test annotation
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot]
---

스프링 프레임워크를 사용할 때 **DI(의존성 주입)**과 **테스트**, **애노테이션**을 잘 활용하면 코드가 훨씬 깔끔해지고 안정적이다. 이 글에서는 다음 **3** 가지를 쉽고 빠르게 따라본다.

1. 간단한 통합 테스트: TestRestTemplate 사용하기

2. 단위 테스트: DI 없이, DI와 함께

3. 애노테이션으로 똑똑하게: @Primary와 간단한 패턴

### 1. 통합 테스트(TestRestTemplate)

컨트롤러를 브라우저 없이 코드로 확인하는 방법이다.

``` java
@Test
void testHelloApi() {
    // 서버가 켜져 있다고 가정하고, 요청 보내기
    TestRestTemplate client = new TestRestTemplate();
    
    ResponseEntity<String> response = client.getForEntity(
        "http://localhost:8080/hello?name=Spring",
        String.class
    );
    
    // 결과 확인
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    assertThat(response.getBody()).contains("Hello Spring");
}
```
서버에 `GET /hello?name=Spring`을 보내고

상태 코드`200`와 응답 내용`Hello Spring`을 확인한다.

### 2. 단위 테스트

#### 2-1. DI 없이

의존 객체가 없으면 다음과 같이

``` java
@Test
void testSimpleService() {
    SimpleHelloService service = new SimpleHelloService();
    String result = service.sayHello("Test");
    assertThat(result).isEqualTo("Hello Test");
}
```

`new`로 객체를 생성하고 메소드 결과만 확인하면 끝난다.

#### 2-2. DI와 함께

서비스나 컨트롤러에 다른 객체가 필요할 때는 실제 대신 **테스트용 가짜(Stub)**를 주입한다.

``` java
@Test
void testHelloController() {
    // name을 그대로 반환하는 가짜 객체 주입
    HelloController controller = new HelloController(name -> name);
    
    String result = controller.hello("JUnit");
    assertThat(result).isEqualTo("JUnit");
}
```

인터페이스 구현체나 람다로 최소 기능만 제공해 테스트를 경량화한다.

### 3. 애노테이션 잘 활용

스프링 애노테이션 한 줄로 환경을 바꾸고 디자인 패턴도 쉽게 적용할 수 있다.

#### 3-1. @Primary

빈(Bean)이 여러 개일 때 어떤 것을 기본으로 사용할지 지정한다.

``` java
@Primary
@Component
class RealPaymentService implements PaymentService { ... }
```

테스트용 가짜 대신 실제 구현을 기본으로 사용할 때 표시한다.

#### 3-2. 간단한 데코레이터

추가 기능이 필요할 때 원래 서비스에 한 줄만 감싸서 주입한다.

``` java
@Component
class LoggingService implements HelloService {
    private final HelloService delegate;
    
    public LoggingService(HelloService delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public String sayHello(String name) {
        System.out.println("Hello 호출: " + name);
        return delegate.sayHello(name);
    }
}
```

기존 코드를 변경하지 않고 로그 기능만 추가한다.

### 마무리

- **TestRestTemplate**로 통합 테스트를 간편하게 실행한다.

- **단위 테스트**는 DI 없이도, DI와 함께도 쉽게 작성한다.

- **@Primary**와 데코레이터로 애노테이션을 활용한다.

이 **3** 가지만 적용하면 스프링 개발이 훨씬 쉬워진다고 한다.