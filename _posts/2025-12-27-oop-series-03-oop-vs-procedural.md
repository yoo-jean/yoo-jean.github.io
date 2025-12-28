---
title: 객체지향, 다시 생각하다 ③ 객체지향과 절차지향, 무엇이 다른가
date: 2025-12-27 13:00:00 +0900
categories: [CS, OOP]
tags: [oop, procedural, design]
---

## 개요
객체지향이 항상 정답은 아니다.
절차지향 또한 여전히 유효한 방식이다.

이 글에서는 두 패러다임의 차이를 통해
**객체지향이 필요한 순간**을 정리해본다.

---

## 절차지향 프로그래밍
데이터와 로직이 분리되어 있고,
함수를 순서대로 실행하며 문제를 해결한다.

- **데이터 중심**

### 장점
- 단순한 구조
- 빠른 구현

### 단점
- 규모가 커질수록 변경이 어려움
- 데이터 변경 규칙이 분산됨
- 디버깅이 어려움

### 대표 절차지향 언어
- C언어
- Pascal
- Fortran
- BASIC

---

## 객체지향 프로그래밍
데이터와 행위를 객체로 묶고,
객체 간 협력으로 시스템을 구성한다.

- **기능 중심**

### 장점
- 책임 단위 분리
- 변경과 확장에 강함
- 디버깅이 쉬움
- 코딩이 절차지향보다 간편함

### 단점
- 초기 설계 비용
- 과한 추상화 위험
- 처리속도가 절차지향보다 느림

### 대표 객체지향 언어
- JAVA
- C++ (절차 + 객체지향)
- C#
- Python (절차+객체+함수형)
- Ruby
- Smalltalk

---


## 절차지향 vs 객체지향 코드로 비교해보기

### 요구사항
- 주문 금액에 따라 할인 정책 적용
- VIP 고객: 20% 할인
- 일반 고객: 10% 할인

---

### 절차지향 방식

```java
public class DiscountCalculator {

    public static int calculatePrice(int price, String userType) {
        if ("VIP".equals(userType)) {
            return price - (price * 20 / 100);
        } else if ("NORMAL".equals(userType)) {
            return price - (price * 10 / 100);
        } else {
            return price;
        }
    }
}
```

---

### 객체지향 방식

```java
public interface DiscountPolicy {
    int discount(int price);
}
```

```java
public class VipDiscountPolicy implements DiscountPolicy {
    public int discount(int price) {
        return price * 20 / 100;
    }
}
```

```java
public class OrderService {
    private final DiscountPolicy discountPolicy;

    public OrderService(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }

    public int calculatePrice(int price) {
        return price - discountPolicy.discount(price);
    }
}
```

---

## 언제 객체지향이 필요한가
- 요구사항 변경이 잦을 때
- 도메인이 복잡할 때
- 팀 단위 개발일 때

---

## 정리
객체지향은 조건문을 줄이는 기술이 아니라  
**변경 가능성이 있는 책임을 객체로 분리하는 설계 방식**이다.
