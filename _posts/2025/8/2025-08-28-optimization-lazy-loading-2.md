---
layout: post
title: "[SpringBoot] 지연 로딩과 성능 최적화 2"
date: '2025-08-28 23:35:21 +0900'
description: optimization lazy loading 2
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot, jpa]
---

# 실무 필수 최적화

## OSIV ON (`spring.jpa.open-in-view: true`, 기본값)

![image1](/assets/img/content/2025-08-28/image.png){: .shadow }

- 트랜잭션 시작부터 API 응답 완료 시점까지 영속성 컨텍스트와 DB 커넥션 유지  
- **장점**: View, Controller에서 지연 로딩 가능  
- **단점**: 커넥션을 오래 붙잡아 실시간 트래픽 환경에서는 커넥션 부족, 장애 위험. 외부 API 대기 시간만큼 커넥션 반환 지연  

---

## OSIV OFF (`spring.jpa.open-in-view: false`)

![image1](/assets/img/content/2025-08-28/image2.png){: .shadow }

- 트랜잭션 종료 시 영속성 컨텍스트 닫고 커넥션 반환, 자원 효율적 사용  
- **단점**: 지연 로딩은 트랜잭션 내부에서 미리 처리해야 함. View/Controller에서 지연 로딩 불가  

---

## 해결 전략

- **Command와 Query 분리 (CQS 원칙)**  
  - `OrderService`: 핵심 비즈니스 로직 (등록, 수정 중심)  
  - `OrderQueryService`: 조회 전용 로직, 화면 API 맞춤 쿼리 (읽기 전용 트랜잭션)  
- 유지보수성과 성능 최적화 모두 확보 가능  

---

## 실무 팁

- **실시간 고객 API**: OSIV 끄기  
- **관리용(Admin)처럼 트래픽 적은 곳**: OSIV 켜도 무방  
