---
layout: post
title: "[SpringBoot] 지연 로딩과 성능 최적화"
date: '2025-08-27 21:03:13 +0900'
description: optimization lazy loading
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot, jpa]
---

이번 글에서는 주문, 배송, 회원 정보를 함께 조회하는 API를 만들고, 지연 로딩(Lazy Loading)으로 인해 발생하는 성능 문제를 단계별로 해결하는 과정을 살펴본다. 실무에서 JPA를 효율적으로 사용하기 위해 반드시 이해해야 하는 중요한 내용이다.

--- 

## V1: 엔티티를 직접 노출하는 방식은 좋지 않다

첫 번째 방법은 Order 엔티티를 API 응답으로 직접 반환하는 방식이다.

```java
// OrderSimpleApiController.java
@GetMapping("/api/v1/simple-orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName(); // Lazy 강제 초기화
        order.getDelivery().getAddress(); // Lazy 강제 초기화
    }
    return all;
}
````

### 문제점

* **엔티티 직접 노출**: 엔티티의 모든 정보가 외부에 노출되어 API 스펙이 고정되지 않고, 보안 문제가 발생할 수 있다.
* **지연 로딩 문제**: Order와 연관된 Member, Delivery는 지연 로딩으로 설정되어 있으며, 실제 엔티티 대신 프록시 객체가 들어 있다. Jackson 라이브러리는 이 프록시 객체를 JSON으로 변환할 수 없어 오류가 발생한다.

### 해결책

* Hibernate5Module 또는 Hibernate5JakartaModule을 등록하여 프록시 객체를 JSON으로 변환할 수 있도록 한다.
* 양방향 관계에서는 무한 루프를 방지하기 위해 한쪽에 `@JsonIgnore`를 추가한다.

V1 방식은 간단한 애플리케이션이 아니라면 절대 권장하지 않는다.

---

## V2: DTO 변환 시 N+1 문제가 발생한다

두 번째 방법은 엔티티를 조회한 후 DTO로 변환하여 반환하는 방식이다.

```java
// OrderSimpleApiController.java
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());
    List<SimpleOrderDto> result = orders.stream()
            .map(SimpleOrderDto::new)
            .collect(toList());
    return result;
}

// SimpleOrderDto.java
@Data
static class SimpleOrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName(); // 여기서 지연 로딩 발생
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress(); // 여기서 지연 로딩 발생
    }
}
```

### 문제점

* **N+1 쿼리 문제**: 주문 목록을 조회하는 쿼리가 1번 실행된다. 그러나 DTO 생성 과정에서 `order.getMember().getName()`과 `order.getDelivery().getAddress()`를 호출할 때마다 지연 로딩이 발생하여 추가 쿼리가 실행된다.
* 주문이 N개라면 총 **1 + N + N번**의 쿼리가 실행되어 성능 저하가 발생한다.

---

## V3: 페치 조인을 사용해 성능을 최적화한다

V2의 N+1 문제를 해결하기 위해 \*\*페치 조인(Fetch Join)\*\*을 사용한다. 페치 조인은 연관된 엔티티를 한 번의 쿼리로 함께 조회하여 지연 로딩을 방지한다.

```java
// OrderRepository.java
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
            "select o from Order o" +
            " join fetch o.member m" +
            " join fetch o.delivery d", Order.class)
            .getResultList();
}

// OrderSimpleApiController.java
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
            .map(SimpleOrderDto::new)
            .collect(toList());
    return result;
}
```

### 장점

* 단일 쿼리로 Member와 Delivery를 함께 조회하여 쿼리가 1번만 실행된다.
* 지연 로딩이 발생하지 않아 V2의 N+1 문제를 완벽하게 해결한다.

### 권장 사항

* 항상 지연 로딩(LAZY)을 기본으로 설정하고, 성능 최적화가 필요한 경우에만 페치 조인을 사용해야 한다.
* 즉시 로딩(EAGER)은 예상치 못한 쿼리를 유발하고 성능 튜닝을 어렵게 하므로 사용하지 않는 것이 좋다.

---

## V4: DTO로 직접 조회한다

마지막 방법은 JPA에서 DTO로 직접 조회하는 방식이다. SELECT 절에서 필요한 필드만 선택해 DTO를 즉시 생성한다.

```java
// OrderSimpleQueryRepository.java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
            "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
            " from Order o" +
            " join o.member m" +
            " join o.delivery d", OrderSimpleQueryDto.class)
            .getResultList();
    }
}

// OrderSimpleApiController.java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```

### 장점

* 필요한 데이터만 선택하므로 네트워크 전송량이 줄어들어 최적화가 가능하다.
* 단 한 번의 쿼리로 모든 데이터를 조회한다.

### 단점

* API 스펙에 맞춘 코드가 리포지토리에 포함되므로 재사용성이 떨어진다.
* 리포지토리가 특정 DTO에 종속된다.


## 결론: 쿼리 방식 선택 권장 순서

엔티티를 DTO로 변환하는 방법(V2, V3)과 DTO로 직접 조회하는 방법(V4)은 각각 장단점이 있다. 실무에서는 다음 순서를 따르는 것이 좋다.

1. **엔티티를 DTO로 변환하는 방법**을 기본으로 사용한다.
2. **성능 문제가 발생하면 페치 조인**으로 최적화한다. 대부분의 성능 문제는 이 단계에서 해결된다.
3. 그래도 해결되지 않거나 특정 화면에서 극도의 쿼리 최적화가 필요할 경우에만 **DTO 직접 조회** 방식을 사용한다.

