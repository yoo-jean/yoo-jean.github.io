---
title: 스프링 정리 노트 ① 스프링 프레임워크란 무엇인가?
date: 2026-01-28 13:00:00 +0900
categories: [CS, Spring]
tags: [spring, framework, container, enterprise]
---


스프링 프레임워크(Spring Framework)는 **자바 기반 엔터프라이즈 애플리케이션**을 만들 때   
반복적으로 등장하는 문제(객체 관리, 트랜잭션, 웹 요청 처리, 데이터 접근 등)를 **프레임워크가 대신 처리**해   
개발자는 비즈니스 로직에 집중할 수 있도록 돕는 생태계다.

핵심은 두 가지다.

- **객체를 만드는/연결하는 책임을 개발자가 직접 하지 않는다.**
- **공통 관심사(트랜잭션, 로깅, 보안 등)를 비즈니스 코드에서 분리한다.**

---

## 스프링을 쓰면 무엇이 좋아지나

### 1) 객체 생성/연결의 표준화
- 전통적인 방식: `new`로 직접 생성 + 직접 연결 → 변경에 약함
- 스프링 방식: 컨테이너가 Bean을 생성/연결(의존성 주입) → 교체/확장에 강함

### 2) 테스트하기 쉬운 구조
- DI를 쓰면 단위 테스트에서 **가짜 객체(mock/fake)**를 쉽게 끼울 수 있다.
- 객체 생성 로직이 숨겨져 있어도 컨테이너 설정만 바꾸면 교체 가능

### 3) 생산성: “기본은 자동, 필요하면 커스터마이징”
- 스프링 생태계는 웹(MVC), 데이터(JDBC/JPA), 트랜잭션, 배치 등 모듈이 풍부
- 스프링 부트는 설정/의존성/실행을 더 단순화해 “바로 실행되는 앱”을 제공

---

## 스프링의 핵심 철학(자주 나오는 문장들)

### POJO 지향
- 특별한 상속이나 인터페이스 구현 없이도 애플리케이션을 만들 수 있어야 한다.
- 특정 기술에 강하게 묶이지 않는 설계를 권장

### OOP(객체지향) 친화
- 추상화(인터페이스) + 다형성 기반으로 구현체를 바꿔 끼울 수 있다.
- 결합도를 낮추고 응집도를 높이는 쪽으로 자연스럽게 유도

---

## 스프링을 구성하는 큰 덩어리

### Core Container
- Bean 생성/관리/주입(IoC/DI)
- 환경 설정(Profile, Property), 이벤트(ApplicationEvent) 등

### AOP
- 프록시 기반으로 공통 관심사를 분리
- 대표 사례: `@Transactional`

### Web (Spring MVC)
- HTTP 요청 → Controller → 응답까지 일관된 처리 파이프라인 제공
- 필터/인터셉터/예외처리/데이터 바인딩 등 포함

### Data Access
- JDBC Template, Spring Data, JPA/Hibernate 등과 결합
- 트랜잭션/예외 변환(PSA) 지원

---

## (짧은 예시) 스프링 이전 vs 스프링 이후

### 스프링 이전(직접 new)
```java
class OrderService {
  private final OrderRepository repo = new JdbcOrderRepository();
}
```

- 구현체 고정(교체 어려움)
- 테스트에서 repo 교체가 번거로움

### 스프링 이후(DI)
```java
class OrderService {
  private final OrderRepository repo;
  OrderService(OrderRepository repo) { this.repo = repo; }
}
```

- 구현체를 외부에서 주입 → 교체 쉬움
- 테스트에서 fake/memory repo 주입 가능

---

## 한 줄 요약
스프링은 **“객체 관리(IoC/DI) + 공통 관심사 분리(AOP) + 기술 추상화(PSA)”** 를 중심으로,   
대규모 서비스에서도 **변경에 강하고 테스트 가능한 구조**를 만들게 해주는 프레임워크다.
