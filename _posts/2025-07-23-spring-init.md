---
layout: post
title: "[Intellij] Spring Boot 프로젝트 생성해보기"
date: '2025-07-23 11:04:17 +0900'
description: how to create a spring boot project
image: /assets/img/content/2025-07-23/image.png
category: [BootCamp, TIL]
tags: [intellij, spring]
---

Spring Boot로 웹 백엔드 개발을 시작하고 싶을 때 환경 설정부터 프로젝트 생성까지 처음엔 막막할 수 있습니다. 이 글에서는 IntelliJ를 활용해 Spring Boot 프로젝트를 만드는 방법을 정리해봤어요.

### 📁 프로젝트 생성
- **IntelliJ** 실행 후 **New Project** 선택
- **Spring Initializr**(Spring Boot)를 선택하고 아래와 같이 설정

![image1](/assets/img/content/2025-07-23/image2.png)

- Language: Java
- Build Tool: Gradle (Groovy)
- JDK: 17
- Packaging: Jar
- **Spring Boot 버전 선택**

![image2](/assets/img/content/2025-07-23/image3.png)

- 3.1 이상 권장
- 의존성 선택
- 아직 선택하지 않아도 되지만 이후에 필요한 웹, JPA, MySQL 등을 추가 가능
- **Finish** 클릭 시 프로젝트 생성 완료!

![image3](/assets/img/content/2025-07-23/image4.png)

### 마무리

이제 기본 Spring Boot 프로젝트가 IntelliJ 안에 설치되었습니다! 앞으로는 application.yml 혹은 application.properties 설정, 디렉토리 구조 이해 등을 통해 본격적인 개발을 시작하실 수 있어요.