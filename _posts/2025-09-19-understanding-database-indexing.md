---
layout: post
title: "[SpringBoot] 데이터베이스 인덱스 개념 이해"
date: '2025-09-19 11:18:12 +0900'
description: understanding database indexing
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [springboot, jpa, index]
---

데이터베이스 인덱스는 도서관의 카탈로그 시스템과 같은 역할을 수행한다. 도서관에서 원하는 책을 찾기 위해 모든 책장을 뒤지지 않고 카탈로그를 먼저 확인하는 것처럼, 데이터베이스 인덱스는 테이블 전체를 스캔하지 않고도 원하는 데이터에 빠르게 접근할 수 있는 경로를 제공한다.

인덱스 생성 시 데이터베이스 엔진은 **B-tree** 또는 **hash table** 같은 자료구조를 활용하여 인덱싱 대상 컬럼의 값과 실제 레코드 위치 정보를 담은 별도의 데이터 구조를 구축한다.

## 인덱스 분류체계

데이터베이스에서 사용되는 인덱스는 용도와 구조에 따라 다음과 같이 분류된다.

### Single-column Index
한 개의 컬럼에 대해서만 생성되는 가장 기본적인 형태의 인덱스다. 특정 컬럼 값으로 검색하거나 정렬하는 작업이 빈번한 경우 효과적이다.

### Composite Index
두 개 이상의 컬럼을 결합하여 생성하는 인덱스다. 복수의 조건을 동시에 만족하는 레코드를 찾는 쿼리에서 높은 성능을 발휘한다.

### Unique Index
지정된 컬럼에 중복 데이터 입력을 차단하는 제약 조건을 포함한 인덱스다. 데이터 일관성 보장과 함께 검색 성능 향상을 동시에 제공한다.

### Clustered Index
테이블 데이터의 물리적 저장 순서를 결정하는 인덱스로, 일반적으로 primary key가 이 역할을 담당한다.

### Non-clustered Index
데이터의 물리적 배치와 무관하게 생성되는 인덱스로, 별도의 구조를 통해 데이터 위치 정보를 관리한다.

## 인덱스 도입의 필요성과 고려사항

데이터베이스 인덱스를 도입하는 핵심 목적은 **쿼리 실행 속도 개선**에 있다. 하지만 이에 따른 트레이드오프도 존재한다.

### 조회 성능 향상 효과
대용량 테이블에서 SELECT 연산의 실행 시간을 획기적으로 단축시킬 수 있다. 수초가 소요되던 쿼리가 밀리초 단위로 완료되는 경우도 흔히 발생한다.

### 데이터 변경 작업에 미치는 영향
INSERT, UPDATE, DELETE 작업 시 인덱스 구조도 함께 갱신되어야 하므로 약간의 성능 저하가 발생할 수 있다. 또한 인덱스는 추가적인 디스크 공간을 요구하므로 적절한 균형점을 찾는 것이 중요하다.

## Spring Boot 환경에서의 인덱스 구현 방법

Spring Boot 기반 애플리케이션에서는 JPA(Java Persistence API)를 통해 데이터베이스 연동 작업을 처리하는 것이 일반적이다. JPA는 어노테이션 기반으로 인덱스를 선언적으로 정의할 수 있는 기능을 제공한다.

아래는 JPA 어노테이션을 활용한 인덱스 정의 예시다:

```java
import javax.persistence.*;

@Entity
@Table(name = "members", indexes = {
    @Index(name = "idx_member_email", columnList = "email"),
    @Index(name = "idx_member_name_age", columnList = "lastName, firstName")
})
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(length = 50)
    private String firstName;
    
    @Column(length = 50)
    private String lastName;

    // 생성자, getter, setter 메서드 생략
}
```

위 코드에서는 `email` 컬럼에 단일 인덱스를, `lastName`과 `firstName` 컬럼 조합에 복합 인덱스를 각각 생성하고 있다. `@Index` 어노테이션이 인덱스 생성을 위한 메타데이터를 제공한다.

### NULL 값 처리 방식
대부분의 RDBMS에서는 기본적으로 NULL 값도 인덱스에 포함시킨다. 이러한 NULL 값들은 데이터베이스 제품에 따라 인덱스의 시작점이나 끝점에 위치하게 된다. 만약 NULL 값을 인덱스에서 제외하고 싶다면 각 데이터베이스별 고유 문법을 사용한 조건부 인덱스(Partial Index)를 생성해야 하며, 이는 일반적으로 네이티브 SQL 구문을 요구한다.

## Spring Data JPA를 통한 인덱스 활용

Spring Data는 개발자가 직접 구현체를 작성하지 않아도 자동으로 리포지토리 클래스를 생성해주는 기능을 제공한다. 다음은 인덱싱된 컬럼을 효율적으로 활용하는 리포지토리 인터페이스 예시다:

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Member findByEmail(String email);
    
    List<Member> findByLastNameAndFirstName(String lastName, String firstName);
    
    @Query("SELECT m FROM Member m WHERE m.lastName = ?1 ORDER BY m.firstName")
    List<Member> findMembersByLastNameSorted(String lastName);
}
```

이 리포지토리에서 `findByEmail(String email)` 메서드는 email 컬럼의 단일 인덱스를, `findByLastNameAndFirstName(...)` 메서드는 복합 인덱스를 각각 활용한다. 사용자 정의 쿼리 역시 정렬과 필터링 과정에서 복합 인덱스의 혜택을 받게 된다.

### 성능 개선 사례 분석
수만 건의 회원 데이터를 보유한 Member 테이블에서 email 컬럼에 인덱스가 없는 상황을 가정해보자. 특정 이메일로 회원을 검색하는 쿼리는 전체 테이블 스캔으로 인해 수 초의 시간이 소요될 수 있다. 반면 email 컬럼에 적절한 인덱스를 생성하면 동일한 쿼리가 수 밀리초 내에 완료된다.

성능 차이는 다음 SQL 명령을 통해 확인할 수 있다:

```sql
EXPLAIN ANALYZE SELECT * FROM members WHERE email = 'user@example.com';
```

> 인덱스 적용 후에는 실행 계획과 소요 시간에서 현저한 개선 효과를 관찰할 수 있다.
{: .prompt-info }

## 효과적인 인덱싱 전략

성공적인 인덱싱을 위해서는 체계적인 접근 방법이 필요하다.

### 1. 선택적 인덱싱 원칙
WHERE 조건절, JOIN 연산, ORDER BY 정렬에서 빈번하게 사용되는 컬럼을 우선적으로 인덱싱 대상으로 선정해야 한다.

### 2. 적정 수준 유지
과도한 인덱스 생성은 데이터 변경 작업의 성능을 저하시키고 불필요한 저장 공간을 소모할 수 있으므로 신중한 판단이 요구된다.

### 3. Composite Index 활용 최적화
다중 컬럼을 동시에 사용하는 쿼리가 많은 경우, 개별 Single-column Index보다는 Composite Index가 더 효율적인 성능을 제공할 수 있다.

### 4. 쿼리 패턴 분석 기반 설계
애플리케이션에서 가장 빈번하게 실행되고 성능에 민감한 쿼리들을 식별하여, 이들에게 최대 효과를 제공할 수 있는 인덱스를 설계해야 한다.

### 5. 통계 정보 관리
대부분의 RDBMS는 쿼리 최적화를 위해 테이블과 인덱스에 대한 통계 정보를 활용한다. 따라서 이러한 통계 정보를 주기적으로 갱신하여 최적의 실행 계획이 수립되도록 해야 한다.

### 6. 인덱스 사용률 모니터링
생성된 인덱스 중 실제로 활용되는 것과 그렇지 않은 것을 구분하여 파악하고, 이를 바탕으로 인덱싱 전략을 지속적으로 개선해나가야 한다.

## 결론

데이터베이스 인덱싱은 단순한 설정 작업이 아니라 지속적인 관리와 최적화가 필요한 영역이다. 애플리케이션의 성장과 변화에 맞춰 인덱싱 전략도 함께 진화해야 하며, 이러한 노력을 통해 얻을 수 있는 성능 향상 효과는 투입된 시간과 비용을 충분히 정당화한다.

올바른 인덱싱 전략 수립과 운영을 통해 Spring Boot 애플리케이션의 데이터베이스 성능을 획기적으로 개선할 수 있으며, 이는 궁극적으로 사용자 경험 향상과 시스템 안정성 확보로 이어진다.

---

**참고 자료:**  
[Indexing to improve query performance in spring boot](https://medium.com/@LynneMunini/indexing-to-improve-query-performance-in-spring-boot-173723b5f906)