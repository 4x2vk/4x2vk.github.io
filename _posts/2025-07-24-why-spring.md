---
layout: post
title: "[Spring] 왜 스프링인가?"
date: '2025-07-24 18:22:34 +0900'
description: why spring
image: /assets/img/content/2025-07-23/image.png
category: [BootCamp, TIL]
tags: [intellij, spring]
---

프로그래밍에서 디자인 패턴은 객체 간 유연한 관계 설정과 유지보수를 위해 필수적이다. 그러나 순수 Java로 디자인 패턴을 직접 구현하면 여러 불편함이 발생한다. Spring 프레임워크는 이러한 과정을 자동화하여 생산성을 크게 높여 준다.

### 🤔 순수 Java에서의 짐작 가능한 불편함

**객체 생성과 연결**
- 개발자가 new로 객체를 일일이 생성하고 의존성을 설정해야 한다.

**생명주기 관리 부재**
- 객체의 생성과 소멸 시점을 개발자가 직접 제어해야 한다.

**테스트의 어려움**
- 강한 결합 구조에서는 Mock 객체로 대체하기가 번거롭다.

**자동화 부족**
- 로깅, 트랜잭션, 보안, 캐싱 등을 매번 수작업으로 작성해야 한다.

### Spring을 사용하면 쉬워지는 이유
Spring의 IoC(Inversion of Control) 컨테이너는 여러 디자인 패턴을 내장하여 개발자의 반복 작업을 줄여 준다.

| 디자인 패턴              | 순수 Java 구현 예시                                        | Spring 적용 방식                                   |
| ------------------- | ---------------------------------------------------- | ---------------------------------------------- |
| **DI (의존성 주입)**     | `UserService svc = new UserService(new UserRepo());` | `@Component` + `@Autowired`로 자동 주입             |
| **Singleton**       | `private static instance` 직접 관리                      | 모든 Bean이 기본적으로 싱글톤 스코프                         |
| **Factory Method**  | 별도 팩토리 클래스 작성                                        | `@Configuration`의 `@Bean` 메서드 사용               |
| **Proxy / AOP**     | 수작업으로 프록시 클래스 작성                                     | `@Aspect`, `@Around`로 자동 프록시 생성                |
| **Template Method** | 추상 클래스 상속 후 구현                                       | `JdbcTemplate`, `RestTemplate` 등 제공            |
| **Observer**        | 이벤트 처리 로직 직접 구현                                      | `ApplicationEventPublisher` + `@EventListener` |

### 예시: 트랜잭션 처리 비교
- 순수 Java 방식

``` java
Connection conn = dataSource.getConnection();
try {
  conn.setAutoCommit(false);
  withdraw(conn, from, amount);
  deposit(conn, to, amount);
  conn.commit();
} catch (Exception e) {
  conn.rollback();
  throw e;
} finally {
  conn.close();
}
```

- Spring + AOP 방식

``` java
@Service
public class AccountService {
    @Transactional
    public void transferMoney(Account from, Account to, int amount) {
        withdraw(from, amount);
        deposit(to, amount);
    }
}
```
@Transactional만 선언하면 트랜잭션 시작, 커밋·롤백, 자원 해제를 Spring이 자동으로 처리한다.

### 결론: Spring을 선택해야 하는 이유
 - 보일러플레이트 코드 감소
 - 테스트 용이성 향상
 - 로깅, 트랜잭션, 보안 등 공통 기능 자동화
- 확장성 있고 유연한 아키텍처 구축

이미 검증된 Spring의 추상화를 활용하면, 반복 작업을 줄이고 핵심 비즈니스 로직에만 집중할 수 있다. 개발 생산성과 유지보수성이 모두 향상된다.