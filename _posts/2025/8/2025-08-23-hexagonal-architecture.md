---
layout: post
title: "[SpringBoot] 헥사고날 아키텍처"
date: '2025-08-23 23:48:26 +0900'
description: hexagonal architecture
image: /assets/img/content/2025-08-26/image.png
category: [BootCamp, TIL]
tags: [springboot, architecture]
--- 

개발을 하다 보면 프로젝트는 처음에는 단순하게 보이지만 시간이 지나면 점점 복잡해진다.  
초반에는 **UI → Service → Database** 구조만으로도 충분하지만, 나중에 REST API, 외부 결제 서비스 메시지 큐(Kafka) 같은 요소들이 추가되면 코드가 뒤엉키고 유지보수가 어려워진다.  

헥사고날 아키텍처(또는 **Ports & Adapters**)는 이러한 문제를 해결하기 위해 고안된 아키텍처 패턴이다.  

---

## 1. 헥사고날 아키텍처란 무엇인가  

핵심 아이디어는 단순하다.  

- **Core(비즈니스 로직)**: 서비스의 핵심 규칙과 로직만 담는다.  
- **Port(포트)**: 코어가 외부와 통신하기 위해 제공하는 인터페이스다.  
- **Adapter(어댑터)**: 실제 구현체로, DB, 외부 API, 웹 UI, CLI 등 모든 외부 요소를 표현한다.  

즉, 코어는 외부 기술을 전혀 알 필요가 없고, 오직 포트를 통해서만 소통한다.  

---

## 2. 장점  

- **테스트 용이성**  
  DB 없이도, 외부 API 없이도 비즈니스 로직을 독립적으로 테스트할 수 있다.  

- **유연성**  
  Stripe 대신 PayPal, MySQL 대신 PostgreSQL로 교체할 때도 코어 코드는 수정할 필요가 없다.  

- **코드의 가독성과 유지보수성**  
  핵심 로직은 깔끔하게 유지되고, 인프라 관련 코드는 각 어댑터에만 위치한다.  

---

## 3. Spring Boot 예제

### (1) Core + Port  

```java
public interface PaymentPort {
    boolean pay(Long orderId, int amount);
}

public class OrderService {
    private final PaymentPort paymentPort;

    public OrderService(PaymentPort paymentPort) {
        this.paymentPort = paymentPort;
    }

    public String createOrder(Long orderId, int amount) {
        boolean paid = paymentPort.pay(orderId, amount);
        return paid ? "주문 " + orderId + " 결제 완료" : "결제 실패";
    }
}
```

### (2) Adapter 구현체
``` java
import org.springframework.stereotype.Component;

@Component
public class StripePaymentAdapter implements PaymentPort {
    @Override
    public boolean pay(Long orderId, int amount) {
        System.out.println("Stripe로 " + amount + "원 결제 시도...");
        return true;
    }
}
```

### (3) 입력 어댑터 (Controller)
``` java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(PaymentPort paymentPort) {
        this.orderService = new OrderService(paymentPort);
    }

    @PostMapping("/{id}")
    public String createOrder(@PathVariable Long id, @RequestParam int amount) {
        return orderService.createOrder(id, amount);
    }
}
```

### (4) 테스트 어댑터
``` java
public class FakePaymentAdapter implements PaymentPort {
    @Override
    public boolean pay(Long orderId, int amount) {
        return amount < 1000; // 테스트에서는 1000원 미만만 성공
    }
}

```

## 4. 언제 쓰면 좋은가

- 프로젝트가 장기간 운영될 예정일 때

- 여러 클라이언트(Web, App, 외부 API 등)가 붙을 가능성이 높을 때

- 인프라 교체 가능성이 있을 때 (DB, 결제 모듈 등)

헥사고날 아키텍처는 유지보수성을 크게 높여준다.
하지만 단기 프로젝트나 간단한 MVP에서는 오히려 복잡도가 늘어날 수 있으니 상황에 맞게 선택하는 것이 좋다.

## 마무리
헥사고날 아키텍처는 단순히 멋져 보이는 패턴이 아니다.
비즈니스 로직을 깨끗하게 보호하고 외부 기술의 변화에 유연하게 대응할 수 있게 해주는 구조다.

모든 프로젝트에 무조건 적용할 필요는 없지만, 비즈니스와 기술을 분리한다는 핵심 원칙만 기억해도 큰 도움이 된다.