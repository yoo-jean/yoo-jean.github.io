---
title: JVM 정리 노트 ⑤ Heap 메모리 상세
date: 2026-01-05 15:00:00 +0900
categories: [CS, JVM]
tags: [jvm, java, runtime, gc, memory]
---

Heap 영역은 **객체(Object)가 생성되고 소멸되는 공간**으로,  
JVM 메모리 관리와 GC 동작을 이해하는 핵심 영역이다.

---

## Heap 메모리 구조

Heap은 크게 **Young Generation**과 **Old Generation**으로 나뉜다.

### Young Generation
- 새로 생성된 객체가 저장되는 영역
- 대부분의 객체는 이 영역에서 생성되고 빠르게 소멸됨
- 다음 세부 영역으로 구성된다
  - **Eden**
  - **Survivor (S0, S1)**

### Old Generation
- Young 영역에서 오래 살아남은 객체가 이동되는 영역
- 크기가 크고 생명주기가 긴 객체 저장
- Major GC(또는 Full GC) 대상

---

## 객체 생성 → 이동 → 소멸 과정

객체의 생명주기는 다음과 같은 흐름을 따른다.

1. **객체 생성**
  - 객체는 대부분 **Eden 영역**에 생성된다.

2. **Minor GC 발생**
  - Eden 영역이 가득 차면 Minor GC가 발생한다.
  - 살아있는 객체는 **Survivor 영역(S0 또는 S1)** 으로 이동한다.
  - Eden 영역은 비워진다.

3. **객체 이동 (Promotion)**
  - Survivor 영역을 여러 번 거친 객체는  
    일정 조건을 만족하면 **Old Generation**으로 이동한다.

4. **객체 소멸**
  - 참조되지 않는 객체는 GC 대상이 되어 메모리에서 제거된다.

---

## Survivor 영역 역할

- Survivor 영역은 객체의 **생존 횟수(age)** 를 관리한다.
- GC가 발생할 때마다 객체의 age가 증가한다.
- 특정 age 이상이 되면 Old Generation으로 이동한다.

> Survivor 영역을 두 개(S0, S1) 사용하는 이유는  
> **Copying 방식 GC**를 통해 빠르고 효율적인 메모리 정리를 위함이다.

---

## Minor GC / Major GC 개념

### Minor GC
- **Young Generation**을 대상으로 수행
- 발생 빈도가 높지만 속도가 빠름
- 애플리케이션에 미치는 영향이 상대적으로 적음

### Major GC (또는 Full GC)
- **Old Generation**을 대상으로 수행
- 수행 시간이 길고 Stop-The-World 영향이 큼
- 성능 저하의 주요 원인이 될 수 있음

---

## TLAB (Thread Local Allocation Buffer)

**TLAB**는 멀티스레드 환경에서 객체 생성 성능을 높이기 위한 메커니즘이다.

- 각 스레드에 **개별적인 Eden 영역 일부를 할당**
- 객체 생성 시 동기화(lock) 비용 감소
- 빠른 객체 할당 가능

### TLAB의 장점
- 멀티스레드 환경에서 객체 생성 성능 향상
- Heap 할당 시 경쟁 최소화

> 대부분의 Java 애플리케이션은  
> TLAB을 통해 매우 빠른 객체 생성을 수행한다.

---

## 정리

- Heap은 객체의 **생성 → 이동 → 소멸** 전 과정을 담당
- Young / Old 영역 분리는 GC 효율을 극대화하기 위한 설계
- Minor GC는 빈번하지만 빠르고  
  Major GC는 드물지만 비용이 크다
- TLAB은 실무 성능 튜닝에서 중요한 역할을 한다

---

> Heap 구조와 객체 생명주기를 이해하면  
> GC 튜닝과 성능 문제의 원인을 훨씬 쉽게 파악할 수 있다.
