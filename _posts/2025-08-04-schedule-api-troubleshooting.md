---
layout: post
title: "[Project] 일정 관리 앱 개발 회고 및 트러블슈팅"
date: '2025-08-04 12:15:25 +0900'
description: schedule api troubleshooting
image: /assets/img/content/2025-08-04/image5.png
category: [BootCamp, TIL]
tags: [springboot, project, troubleshooting]
---

오늘은 일정 관리를 위한 Schedule API 프로젝트를 진행하면서 배우고 경험한 내용을 정리해본다.

### 데이터베이스와의 첫 만남

프로젝트를 진행하면서 처음으로 MySQL DB를 직접 다뤄보게 되었다.

- Schedule Table 생성 후 DB와 코드가 어떻게 연결되는지를 직접 확인

![image1](/assets/img/content/2025-08-04/image.png)

![image2](/assets/img/content/2025-08-04/image2.png)

- Comment Table을 별도로 생성하여 schedule_id를 통해 일정과 댓글 간의 외래키 관계 설정

``` sql
CREATE TABLE comment (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  schedule_id BIGINT,
  name VARCHAR(100),
  password VARCHAR(100),
  contents TEXT,
  created_at DATETIME,
  modified_at DATETIME,
  FOREIGN KEY (schedule_id) REFERENCES schedule(id)
);
```

![image3](/assets/img/content/2025-08-04/image3.png)

![image3](/assets/img/content/2025-08-04/image4.png)

DB Client를 통해 직접 테이블이 생성된 걸 확인하고 데이터가 잘 들어가는 걸 보며 처음 접한 DB가 점점 익숙해지는 순간이었다.이 경험을 통해 코드만 잘 짜는 것보다 데이터 흐름을 이해하고 구조화하는 능력이 얼마나 중요한지 배울 수 있었다.

### 구현 목표

• 	제목, 내용, 작성자, 비밀번호를 기반으로 일정 생성

• 	전체 혹은 특정 작성자의 일정 조회

• 	일정 수정 및 삭제 시 비밀번호 확인

• 	RESTful하게 API 구조화 및 응답 형식 명확화

### 구현 중 좋았던 점

• 	`@Transactional(readOnly = true)`를 통해 조회 성능 최적화

• 	`DTO`를 통해 Controller와 Entity 간의 역할 분리

• 	`ResponseStatusException`으로 에러 처리를 깔끔하게 관리

• 	`Comparator`를 활용한 최신 일정 정렬

### 트러블슈팅 사례

1. 수정 시 비밀번호 확인 누락

    문제: 초기에는 Update시 `password` 확인 로직이 빠져 있었음

    해결:  메서드 추가하여 기존 비밀번호 비교 후 예외 처리

``` java
if (!inputPassword.equals(recentPassword)) {
    throw new ResponseStatusException(HttpStatus.UNAUTHORIZED);
}
```

2. GET 요청 시 `name` 파라미터 필터링

    문제: 모든 일정이 반환되어 사용자별 필터링 어려움

    해결: `@RequestParam(required=false)`로 유연하게 적용하여 `name` 기반 조회 로직 구현

### 회고

이번 프로젝트를 통해 단순한 CRUD 구현을 넘어 API 설계의 중요성과 세심한 보안 고려가 얼마나 중요한지 체감했다. 특히, 비밀번호를 통한 일정 수정/삭제 보안 로직은 단순해 보이지만 사용자의 신뢰를 위한 핵심 기능이었다.

**참고 자료**

• 🔗 [Github 프로젝트](https://github.com/4x2vk/Schedule-API)

• 🔗 [API Docs](https://documenter.getpostman.com/view/47183182/2sB3BANDXa)

• 🔗 [ERD Diagram (확인시 로그인 필요)](https://github.com/4x2vk/ScheduleAPI/issues/6)
