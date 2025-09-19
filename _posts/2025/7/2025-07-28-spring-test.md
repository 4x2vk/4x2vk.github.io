---
layout: post
title: "[Spring] 자동화 테스트와 JUnit5"
date: '2025-07-28 20:10:46 +0900'
description: spring test
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [spring]
---

> 💡 테스트를 만들지 않을 거면 스프링을 도대체 뭐하러 쓰는 거죠?

### 자동화 테스트의 필요성

자동화 테스트는 수동 검증의 번거로움과 사람의 실수를 완전히 제거한다.
코드로 작성된 테스트는 언제 실행하더라도 동일한 결과를 보장해 개발자가 즉시 피드백을 받을 수 있게 한다. 리팩토링이나 신규 기능 추가 시에도 기존 기능이 올바르게 동작하는지 안정적으로 검증할 수 있다.

### 스프링과 테스트 지원

스프링은 의존성 주입(DI)과 AOP 기능으로 컴포넌트 간 결합도를 낮춰 테스트 코드를 쉽게 작성할 수 있게 한다.
`@SpringBootTest`로 전체 컨텍스트를 로딩하거나, `@WebMvcTest`로 웹 레이어만 분리해 빠르게 실행할 수 있다.

### JUnit과 xUnit 계열

JUnit은 Kent Beck과 Eric Gamma가 만든 자바용 테스트 프레임워크로, .NET(NUnit), Python(pyUnit) 등 다양한 언어의 xUnit 계열 원조다.
최신 버전인 JUnit 5는 Platform, Jupiter, Vintage 세 모듈로 구성돼 스프링 테스트 엔진으로 널리 사용된다.

### PaymentService 테스트 예시

``` java
@SpringBootTest
class PaymentServiceTest {

    @Autowired
    private PaymentService paymentService;

    @MockBean
    private ExternalPaymentGateway gateway;

    @Test
    void 결제_성공_테스트() {
        // given
        given(gateway.process(any())).willReturn(true);

        // when
        boolean result = paymentService.pay(1000);

        // then
        assertTrue(result);
    }
}
```

외부 시스템 호출을 `@MockBean`으로 대체해 테스트를 제어한다. 시간이나 랜덤 값 같은 매번 달라지는 요소는 **Clock** 등으로 추상화해 일관된 결과를 보장해야 한다.

### TDD와 클린 코드

테스트 주도 개발(TDD)은 먼저 실패하는 테스트를 작성하고 이를 통과시키는 최소 코드를 구현한 뒤 리팩토링하는 순서로 진행된다.
클린 코드 원칙을 지키면 가독성과 유지보수성이 높아지고 잘 설계된 테스트가 코드 품질을 더욱 강화한다.

자동화 테스트를 일상적인 개발 과정에 자연스럽게 통합하면 더 안정적이고 견고한 애플리케이션이 만들어진다.
