---
layout: post
title: "[Spring] 자바 예외 처리와 JPA"
date: '2025-08-01 20:30:23 +0900'
description: spring exception
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [java, spring]
---

이번엔 자바에서 예외(Exception)를 어떻게 처리하는지부터 JPA랑 스프링에서 DB 예외를 어떻게 다루는지까지 쭉 정리해본다. 예외가 뭐고 왜 생기고 어떻게 잡아야 하는지 헷갈렸다면 이 글로 한 방에 정리하면 된다.

### 1. 예외(Exception)란?
예외는 프로그램이 정상적으로 흐르는 걸 방해하는 사건이다. 예를 들어 계산 도중 0으로 나눈다든지 없는 파일을 연다든지 DB랑 연결이 안 된다든지 그런 게 예외다.

#### 예외와 에러(Error)는 뭐가 다를까?
- Error
  
    시스템에서 진짜 심각한 문제가 생겼을 때 발생한다.
    보통 JVM에서 발생시키고, 우리가 코드로 고치거나 복구하는 건 거의 불가능하다.

- Exception
    
    우리가 코드로 어느 정도 복구 가능한 예외들이다. 대부분의 예외는 이쪽에 속한다.

### 2. Exception의 종류: 체크 예외 vs 언체크 예외

#### 체크 예외 (Checked Exception)
- 애플리케이션에서 복구 가능성이 있다고 본다.

- 컴파일러가 try-catch 하든가, throws 하든가 처리하라고 강제한다.

- 예: IOException, SQLException

#### 언체크 예외 (Unchecked Exception)
- RuntimeException을 상속받은 예외들이다.

- 명시적인 예외 처리를 하지 않아도 컴파일은 된다.

- 예: NullPointerException, IndexOutOfBoundsException

체크 예외는 반드시 처리해야 되고 언체크 예외는 선택이다. 하지만 언체크라고 그냥 방치하면 런타임에 터진다. 그러니까 예외는 항상 신중하게 다뤄야 된다.

### 3. JPA를 이용한 Order 저장
자바에서 RDBMS를 쓸 때는 JDBC, JPA, MyBatis 같은 기술을 쓴다. 이런 기술들은 모두 **DB랑 연결하는 정보(DataSource)**를 기반으로 돌아간다.

#### EmbeddedDatabaseBuilder
- H2, HSQL, Derby 같은 내장형 DB를 빠르게 셋업할 수 있다.

- 개발 중 테스트 DB 쓸 때 진짜 편하다.

#### LocalContainerEntityManagerFactoryBean
- EntityManagerFactory를 XML 설정 없이 간단히 만들 수 있게 도와준다.

- 스프링에서 JPA 설정할 때 자주 쓴다.

### 4. EntityManager와 트랜잭션
JPA는 EntityManagerFactory가 있어야 EntityManager를 만들 수 있다. 그리고 트랜잭션은 이렇게 처리하면 된다:

``` java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = null;

try {
    tx = em.getTransaction();
    tx.begin();

    // 실제 작업 수행
    // ...

    tx.commit();
} catch (RuntimeException e) {
    if (tx != null && tx.isActive()) tx.rollback();
    throw e; // 혹은 메시지를 출력한다
} finally {
    em.close();
}
```
트랜잭션은 항상 begin → 작업 → commit 순서로 간다. 예외 터지면 rollback 해줘야 안전하게 된다.

### 5. Order 리포지토리와 예외
JDBC 쓰다 보면 SQLException이 자주 발생한다. 이건 체크 예외라서 반드시 처리해줘야 된다.

#### JPA의 예외는?
JPA도 내부적으로 JDBC를 쓰기 때문에, 예외가 나면 SQLException을 감싸서 PersistenceException으로 만들어서 던진다. 그리고 그 안에는 JPA 구현체(Hibernate 등)의 예외도 래핑돼 있다.

- 실제로는 SQLException → HibernateException → PersistenceException 이런 식으로 감싸진다.

### 6. DataAccessException 추상화
스프링은 DB 기술마다 다르게 발생하는 예외들을 하나로 감싸서 일관되게 다룰 수 있게 해준다. 그게 바로 **DataAccessException**이다.

#### 이게 왜 좋냐면?
- JDBC의 SQLException

- JPA의 PersistenceException

- 다 DataAccessException으로 변환해서 처리하게 해준다.

덕분에 우리가 어떤 기술을 쓰든 **예외 처리 방식은 동일하게 가져갈 수 있다.** 실무에선 진짜 유용하다. 예외의 종류가 바뀌어도 try-catch 안의 코드는 거의 안 바뀐다. 이게 스프링이 예외를 추상화한 이유다.

### 마무리
자바에서 예외는 그냥 무시하면 안 되는 중요한 기능이다. 특히 데이터베이스처럼 외부 시스템이랑 통신할 땐 **예외가 일상**이다. 예외를 잘 잡고 관리하는 게 **프로답게 개발하는 첫걸음**이다.

- 예외는 Error, Exception으로 나뉜다

- Exception은 Checked / Unchecked로 나뉜다

- JPA는 예외를 감싸서 PersistenceException으로 던진다

- 스프링은 다시 DataAccessException으로 감싸준다

이 흐름만 이해하고 있으면 어떤 예외든 쉽게 다룰 수 있게 된다.