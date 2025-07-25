---
layout: post
title: "[Spring] Spring Boot의 BeanFactory와 ApplicationContext 이해"
date: '2025-07-25 23:12:53 +0900'
description: understanding BeanFactory and ApplicationContext
image: /assets/img/content/2025-07-25/image2.png
category: [BootCamp, TIL]
tags: [intellij, spring]
---

Spring Boot를 사용하다 보면 BeanFactory, ApplicationContext 같은 용어를 자주 접하게 된다. 이 용어들은 스프링의 핵심 개념인 제어의 역전(IoC) 컨테이너와 관련돼 있지만, Spring Boot를 처음 접하는 사람에겐 꽤나 헷갈릴 수 있다.

이 글에서는 BeanFactory와 ApplicationContext가 각각 무엇이고 어떻게 다르며, Spring Boot에서 어떤 역할을 하고 언제 사용되는지 간단하게 정리해 본다.

### 1. IoC 컨테이너가 뭐냐?

BeanFactory와 ApplicationContext를 보기 전에, IoC 컨테이너가 뭔지부터 알아야 한다.

- IoC 컨테이너는 스프링 빈(Spring Bean) 을 관리해 주는 녀석이다.
- 스프링 빈은 Spring이 직접 생성하고, 설정해 주고, 생명 주기까지 관리해 주는 객체다.
- 의존성 주입(DI), 생명 주기 관리, 컴포넌트 간 연결 등을 알아서 해준다.
- 대표적인 IoC 컨테이너 구현체는 BeanFactory와 ApplicationContext 두 가지가 있다.

### 2. BeanFactory – 최소한의 핵심 컨테이너

BeanFactory는 Spring에서 가장 단순한 IoC 컨테이너다. 말 그대로 핵심 기능만 제공된다.

#### 특징

- Lazy Initialization: getBean()을 호출할 때까지 객체를 만들지 않는다.
- Lightweight: 메모리 사용량이 적고, 성능이 민감한 환경에 적합하다.
- Basic Bean Management: 빈을 생성하고, 설정하고, 생명 주기(init, destroy 등)만 관리한다.

#### 예제

``` java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("beans.xml"));
MyBean bean = factory.getBean(MyBean.class);
```
- 요즘에는 거의 쓰지 않는다. 옛날 XML 기반 스프링 애플리케이션에서나 등장한다.

#### 언제 쓰면 되냐?

- 메모리가 부족한 환경에서 가볍게 사용하고 싶을 때
- 빈 초기화 타이밍을 직접 제어하고 싶을 때
- 테스트나 단순한 로직에서만 잠깐 쓰고 싶을 때

### 3. ApplicationContext – BeanFactory보다 강력한 컨테이너

ApplicationContext는 BeanFactory를 확장한 상위 개념이다. Spring Boot에서는 기본적으로 이걸 사용하게 된다.

#### 특징
- 즉시 초기화(Eager Initialization) 된다
    - 앱이 실행될 때 대부분의 싱글톤 빈을 미리 만들어 둔다.

#### 엔터프라이즈급 기능을 다 제공한다:

- AOP Integration: 관점 지향 프로그래밍이 가능하다.
- Event Handling: ApplicationEvent 기반으로 이벤트를 발행하고 처리할 수 있다.
- Internationalization (i18n): 다국어 메시지를 관리할 수 있다.
- Resource Loading: 파일, URL, 클래스패스 리소스를 쉽게 다룰 수 있다.
- Annotation Support: @Component, @Autowired 등.

#### 예제

``` java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
MyBean bean = context.getBean(MyBean.class);
```

#### 언제 쓰면 되냐?
- 거의 모든 Spring Boot 애플리케이션에서 사용된다
- 이벤트, AOP, 메시지 처리 등 복잡한 기능을 써야 할 때
- 빈들을 빠르게 찾아서 자동으로 연결하고 싶을 때

### 4. BeanFactory vs ApplicationContext

![image1](/assets/img/content/2025-07-25/image.png)

### Spring Boot에서 ApplicationContext를 어떻게 쓰냐?

Spring Boot는 내부적으로 ApplicationContext를 자동으로 구성해서 사용하게 해준다. 개발자는 일일이 설정할 필요가 없다.

#### 동작 방식

- 일반적으로 AnnotationConfigApplicationContext 또는 웹의 경우 AnnotationConfigServletWebServerApplicationContext를 사용한다.
- @SpringBootApplication 어노테이션을 붙이면 컴포넌트를 자동으로 스캔해서 필요한 빈들을 등록해 준다.

#### 예제

``` java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);  // 자동으로 ApplicationContext가 초기화된다
    }
}

```

### 6. BeanFactory를 굳이 쓰는 경우도 있냐?
물론 있다. 다만 매우 예외적인 상황이다.
- 테스트 환경: 아주 가볍게 컨테이너를 띄워야 할 때.
- 리소스가 극단적으로 부족한 환경: 예를 들어 임베디드 장치 같은 곳.

### 마무리
- BeanFactory는 Spring의 기본 IoC 컨테이너로 단순하고 가볍다.
- ApplicationContext는 BeanFactory보다 기능이 훨씬 많고 Spring Boot에서는 기본적으로 사용되는 컨테이너다.
- Spring Boot를 쓸 땐 대부분 ApplicationContext를 자동으로 사용하게 되며 개발자는 그 위에서 비즈니스 로직에 집중하면 된다.