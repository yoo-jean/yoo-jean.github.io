---
title: 객체지향, 다시 생각하다 ⑤ 테스트 가능한 코드
date: 2025-12-27 17:00:00 +0900
categories: [CS, OOP]
tags: [test, oop, tdd, design]
---

## 개요
테스트하기 어려운 코드는  
대부분 설계가 잘못된 경우가 많다.

이 글에서는  
객체지향 설계가 왜 테스트 가능성을 높이는지 정리한다.

---

## 테스트하기 어려운 코드의 특징
- 하나의 클래스가 책임을 너무 많이 가짐
- 외부 시스템(DB, API)에 강하게 의존
- 생성자에서 객체를 직접 생성

---

## 객체지향 설계와 테스트
객체지향 설계는 자연스럽게 테스트 친화적인 구조를 만든다.

- SRP → 테스트 대상이 명확해짐
- DIP → Mock 객체 사용 가능
- 인터페이스 → 대체 구현 쉬움

---

## 예시
```java
class PaymentService {
    private final PayClient payClient;

    public PaymentService(PayClient payClient) {
        this.payClient = payClient;
    }
}
```

이 구조는 테스트에서 `PayClient`를  
Mock으로 쉽게 교체할 수 있다.

---

## 정리
- 테스트는 설계의 결과물이다
- 객체지향은 테스트를 위한 최고의 기반이다
- 테스트가 쉬운 코드는 유지보수가 쉽다

