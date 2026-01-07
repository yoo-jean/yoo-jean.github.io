---
title: JVM 정리 노트 ⑧ JVM과 멀티스레딩
date: 2026-01-06 16:00:00 +0900
categories: [CS, JVM]
tags: [jvm, java, runtime, gc, memory]
---

Java의 멀티스레딩 모델은 JVM 내부 구조와 운영체제(OS) 스레드 모델을 함께 이해해야 정확히 파악할 수 있다.

---

## Java Thread ↔ OS Thread

Java에서 생성한 `Thread`는 JVM 내부에서 관리되지만,  
실제 실행은 **운영체제(OS)의 네이티브 스레드**에 의해 수행된다.

- Java Thread는 OS Thread와 **1:1 매핑**됨
- 스레드 생성, 스케줄링, 실행은 OS가 담당
- JVM은 스레드 실행을 위한 추상화 계층 역할 수행

이 구조 덕분에 Java는 플랫폼 독립성을 유지하면서도  
운영체제의 스레드 성능을 그대로 활용할 수 있다.

---

## Stack과 Thread의 관계

각 Java Thread는 **자신만의 Stack 영역**을 가진다.

- Thread 생성 시 Stack 메모리도 함께 생성
- Stack에는 메서드 호출 정보가 **Stack Frame** 형태로 저장됨
  - 지역 변수
  - 매개변수
  - 반환 주소

특징:
- Stack 영역은 **스레드 간 공유되지 않음**
- 메서드 종료 시 Stack Frame은 자동 제거
- Stack 크기 초과 시 `StackOverflowError` 발생

---

## Context Switching

**Context Switching**은 CPU가 실행 중인 스레드를 교체하는 과정이다.

- 현재 스레드의 상태를 저장
- 다음 실행할 스레드의 상태를 복원
- OS 스케줄러에 의해 관리

Context Switching은 필수적인 과정이지만,
- 빈번하게 발생하면 성능 저하 원인이 될 수 있음
- 과도한 스레드 생성은 오히려 처리 속도를 떨어뜨릴 수 있음

---

## Thread Safety

멀티스레드 환경에서는 **동시 접근으로 인한 데이터 불일치** 문제가 발생할 수 있다.

Thread Safety란:
- 여러 스레드가 동시에 접근해도
- 데이터의 일관성과 정확성이 보장되는 상태

대표적인 해결 방법:
- `synchronized`
- `Lock` (ReentrantLock 등)
- 불변 객체(Immutable Object)
- 스레드 안전한 컬렉션 사용

---

## JVM 메모리 가시성 (Memory Visibility)

JVM에서는 각 스레드가 **자신의 작업 메모리**를 사용한다.
이로 인해 한 스레드의 변경 사항이 다른 스레드에 즉시 보이지 않을 수 있다.

이를 해결하기 위한 방법:
- `volatile` 키워드
  - 변수 변경 사항을 메인 메모리에 즉시 반영
- `synchronized`
  - 락 획득 및 해제 시 메모리 동기화 보장

> Thread Safety 문제는  
> 단순한 동기화 문제가 아니라 **메모리 가시성 문제**에서 시작되는 경우가 많다.

---

## 정리

- Java Thread는 OS Thread와 1:1로 매핑
- Thread마다 독립적인 Stack 영역을 가짐
- Context Switching은 필수지만 비용이 존재
- Thread Safety와 메모리 가시성은 함께 고려해야 함

---

> 멀티스레딩을 이해한다는 것은  
> 스레드를 많이 쓰는 것이 아니라  
> **안전하게 쓰는 방법을 아는 것**이다.
