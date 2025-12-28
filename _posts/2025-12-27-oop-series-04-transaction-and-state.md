---
title: 객체지향, 다시 생각하다 ④ 트랜잭션과 상태 관리
date: 2025-12-27 15:00:00 +0900
categories: [CS, OOP]
tags: [transaction, state, spring, oop]
---

## 개요
객체지향 설계를 하다 보면 결국 마주하게 되는 문제가 있다.  
바로 **상태(state)와 트랜잭션(transaction)** 이다.

이 글에서는 객체지향 관점에서  
왜 상태 관리가 중요한지,  
그리고 트랜잭션은 어디까지 책임져야 하는지 정리해본다.

---

## 상태란 무엇인가
상태란 객체가 가지고 있는 **변할 수 있는 값**이다.

```java
class Order {
    private OrderStatus status;
}
```

상태가 많아질수록 객체는 복잡해지고  
잘못 다루면 버그의 원인이 된다.

---

## 상태 변경의 책임
객체지향에서는  
**상태를 변경하는 책임은 객체 자신에게 있어야 한다.**

```java
order.complete();
```
외부에서 직접 상태 값을 변경하기보다는  
의미 있는 메시지를 통해 변경하는 것이 좋다.

---

## 트랜잭션의 위치
트랜잭션은 보통 **Service 계층**에서 관리한다.

- 도메인 객체는 비즈니스 규칙에 집중
- 트랜잭션은 흐름 제어 역할

```java
@Transactional
public void completeOrder(Long orderId) {
    Order order = orderRepository.findById(orderId);
    order.complete();
}
```

---

## 정리
- 상태는 객체 내부에서 관리한다
- 트랜잭션은 객체의 책임이 아니다
- 역할 분리가 객체지향 설계를 지켜준다

