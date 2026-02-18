---
title: JPA 핵심 개념 정리 ① JPA Basic
date: 2026-02-14 20:00:00 +0900
categories: [CS, JPA]
tags: [jpa, hibernate, ORM, persistence]
---
 
JPA(Java Persistence API)의 핵심 개념을 기본부터 실무 관점까지
체계적으로 정리해 보았다. 

------------------------------------------------------------------------

## 1. JPA란 무엇인가?

JPA는 자바 진영에서 객체와 관계형 데이터베이스(RDBMS) 간의 매핑을
표준화한 ORM(Object Relational Mapping) 기술이다.

즉, 개발자가 SQL 중심이 아닌 **객체 중심으로 데이터베이스를 다룰 수
있도록 지원하는 표준 인터페이스**이다.

### 1-1. ORM이란?

ORM은 객체(Object)와 테이블(Relation)을 자동으로 매핑해주는 기술이다.

| 구분    | 기존 방식  | ORM 방식 |
|-------|--------|--------|
| 개발 기준 | SQL 중심 | 객체 중심  |
| 유지보수  | 어려움    | 쉬움     |
| 확장성   | 낮음     | 높음     |

### 1-2. JPA 장점

-   SQL 의존도 감소
-   객체지향 설계 유지 가능
-   생산성 향상
-   유지보수 비용 절감
-   DB 변경에 대한 영향 최소화

### 1-3. JPA 구현체

JPA는 인터페이스이며 실제 구현체가 필요하다.

대표 구현체: - Hibernate (가장 많이 사용) - EclipseLink - OpenJPA

실무에서는 대부분 Hibernate를 사용한다.

------------------------------------------------------------------------

## 2. 영속성 컨텍스트(Persistence Context)

영속성 컨텍스트란 **엔티티를 관리하는 논리적인 공간**이다.

EntityManager가 생성될 때 함께 생성된다.

### 2-1. 엔티티 생명주기

엔티티는 다음 4가지 상태를 가진다.

| 상태  | 설명            |
|-----|---------------|
| 비영속 | JPA와 무관한 상태   |
| 영속  | 영속성 컨텍스트에 관리됨 |
| 준영속 | 관리 대상에서 분리됨   |
| 삭제  | 삭제 예약 상태      |

### 2-2. 주요 기능

#### ① 1차 캐시

같은 트랜잭션 내에서는 DB 조회 없이 캐시에서 조회

#### ② 동일성 보장

같은 엔티티 조회 시 항상 동일 객체 반환

#### ③ 트랜잭션 쓰기 지연

INSERT SQL을 즉시 실행하지 않고 모아서 처리

#### ④ 변경 감지 (Dirty Checking)

객체 값 변경 시 자동 UPDATE 발생

#### ⑤ 지연 로딩 지원

연관 객체를 필요할 때 조회

------------------------------------------------------------------------

## 3. 엔티티 매니저 팩토리(EntityManagerFactory)

EntityManager를 생성하는 객체이다.

### 특징

-   애플리케이션 전체에서 하나만 생성
-   Thread-Safe 보장
-   생성 비용이 매우 큼
-   애플리케이션 종료 시 close 필요

### 구조

Client → EntityManagerFactory → EntityManager → Persistence Context

------------------------------------------------------------------------

## 4. 엔티티 매니저(EntityManager)

엔티티의 CRUD를 담당하는 핵심 객체이다.

### 주요 역할

-   persist()
-   find()
-   remove()
-   merge()
-   createQuery()

### 특징

-   Thread-Safe 하지 않음
-   요청 단위로 생성
-   트랜잭션 단위로 관리

------------------------------------------------------------------------

## 5. 엔티티(Entity)

엔티티는 DB 테이블과 매핑되는 객체이다.

### 필수 조건

-   @Entity 선언
-   기본 생성자 필수
-   식별자(@Id) 필요
-   final 클래스 사용 불가

### 예제

``` java
@Entity
@Table(name = "member")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    private int age;
}
```

------------------------------------------------------------------------
