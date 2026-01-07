---
title: 객체지향, 다시 생각하다 ⑩ GRASP 설계원칙
date: 2026-01-07 19:00:00 +0900
categories: [CS, OOP]
tags: [domain, service, architecture]
---

##  GRASP란?
**GRASP (General Responsibility Assignment Software Patterns)** 는  
객체지향 설계에서 *“이 책임은 어떤 객체가 가져가야 하는가?”* 를 판단하기 위한 **책임 할당 원칙 모음**이다.

목표는 다음과 같다.
- 객체가 자신의 역할에 맞는 책임을 갖도록 설계
- 결합도(Low Coupling)는 낮추고, 응집도(High Cohesion)는 높이기
- Service에 로직이 과도하게 몰리는 구조 방지

---

##  GRASP 9가지 핵심 원칙

### 1) Information Expert
- 필요한 정보를 가장 많이 아는 객체가 책임을 가진다.

**예시**
- 주문 금액 계산 → `Order`

---

### 2) Creator
- A가 B를 포함하거나 많이 알고 있으면 A가 B를 생성한다.

**예시**
- `Order` → `OrderItem` 생성

---

### 3) Controller
- 외부 요청을 받아 도메인에 위임하는 진입점 역할

**주의**
- Controller는 흐름 제어만 담당, 비즈니스 로직 금지

---

### 4) Low Coupling
- 객체 간 의존성을 최소화하여 변경 영향을 줄인다.

---

### 5) High Cohesion
- 객체는 하나의 책임에 집중해야 한다.

---

### 6) Polymorphism
- 조건문(if/else) 대신 다형성을 사용한다.

---

### 7) Pure Fabrication
- 도메인에 어울리지 않는 책임은 별도 객체로 분리한다.

**예시**
- Repository, Mapper, Validator

---

### 8) Indirection
- 중간 객체를 두어 직접적인 결합을 줄인다.

---

### 9) Protected Variations
- 변경 가능성이 높은 부분을 인터페이스로 감싼다.

---

## 3. 적용 예제

### ❌ 잘못된 예 (Service에 로직 집중)

```java
public class OrderService {
    public int calculateTotalPrice(List<OrderItem> items) {
        int sum = 0;
        for (OrderItem item : items) {
            sum += item.getPrice() * item.getQuantity();
        }
        return sum;
    }
}
```

문제점
- Service가 도메인 로직을 침범
- 응집도 낮음
- 테스트 및 확장 어려움

---

### ✅ GRASP 적용 예

#### Information Expert 적용

```java
public class Order {
    private List<OrderItem> items;

    public int calculateTotalPrice() {
        return items.stream()
                .mapToInt(OrderItem::calculatePrice)
                .sum();
    }
}
```

```java
public class OrderItem {
    private int price;
    private int quantity;

    public int calculatePrice() {
        return price * quantity;
    }
}
```

**적용 원칙**
- Information Expert
- High Cohesion
- Low Coupling

---

### ❌ if/else 남발

```java
public int pay(String type) {
    if (type.equals("CARD")) return cardPay();
    if (type.equals("BANK")) return bankPay();
    return 0;
}
```

---

### ✅ Polymorphism 적용

```java
public interface Payment {
    void pay();
}
```

```java
public class CardPayment implements Payment {
    public void pay() {
        // 카드 결제
    }
}
```

```java
public class BankPayment implements Payment {
    public void pay() {
        // 계좌 결제
    }
}
```

---

## 4. 실무 기준 요약

- Service에 로직이 많다 → **Information Expert / High Cohesion**
- if/else가 많다 → **Polymorphism**
- 도메인에 안 맞는 책임 → **Pure Fabrication**
- 변경이 잦다 → **Protected Variations**

---

## 5. 면접용 한 문장
> GRASP는 객체가 자신의 데이터와 역할에 맞는 책임을 갖도록 하는 객체지향 설계 원칙으로,   
> 결합도는 낮추고 응집도는 높이기 위한 기준입니다.

---

### Reference
- Craig Larman, *Applying UML and Patterns*
