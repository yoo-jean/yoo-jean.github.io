---
title: JVM 정리 노트 ④ JVM 메모리 구조
date: 2026-01-04 19:00:00 +0900
categories: [CS, JVM]
tags: [jvm, java, runtime, gc, memory]
---

JVM은 프로그램 실행을 위해 여러 메모리 영역을 사용하며,  
각 영역은 **역할과 생명주기, 스레드 공유 여부**가 명확히 구분된다.

---

## JVM Runtime Data Area 구성

- **Method Area**
- **Heap**
- **Stack**
- **PC Register**
- **Native Method Stack**

---

## Method Area

- 클래스 메타데이터 저장
  - 클래스 구조, 메서드 정보, 필드 정보
- static 변수 및 상수 저장
- 모든 스레드가 공유

### Java 8 이후: Metaspace

- Java 8부터 Method Area 구현이 **PermGen → Metaspace**로 변경
- Metaspace는 **Native Memory(운영체제 메모리)** 를 사용
- 클래스 메타데이터 크기에 따라 **동적으로 확장 가능**

> **PermGen이 제거된 이유**  
> 고정된 크기로 인해 `OutOfMemoryError: PermGen space` 문제가 잦았고,  
> 이를 해결하기 위해 동적 확장이 가능한 Metaspace로 대체되었다.

---

##  Heap

- 객체(Object)와 배열(Array)이 생성되는 영역
- Garbage Collection 대상
- 모든 스레드가 공유
- JVM 메모리 영역 중 가장 큰 비중을 차지

---

## Stack

- 스레드별로 생성되는 메모리 영역
- 메서드 호출 시 **Stack Frame** 생성
- 지역 변수, 매개변수, 반환 주소 저장
- 메서드 종료 시 자동으로 제거

---

## PC Register

- 각 스레드가 현재 실행 중인 JVM 명령어의 주소를 저장
- 스레드별로 독립적으로 관리

---

## Native Method Stack

- Java가 아닌 네이티브 코드(C/C++) 실행 시 사용
- JNI(Java Native Interface) 호출 시 활용

---

## JVM 메모리 구조 요약

| 영역 | 저장 내용 | GC 대상 | 스레드 공유 |
|---|---|---|---|
| Method Area / Metaspace | 클래스 메타데이터, static 변수 | ❌ | 공유 |
| Heap | 객체, 배열 | ⭕ | 공유 |
| Stack | 지역 변수, 매개변수 | ❌ | 스레드별 |
| PC Register | 실행 중인 명령 주소 | ❌ | 스레드별 |
| Native Method Stack | 네이티브 메서드 정보 | ❌ | 스레드별 |

---

> JVM 메모리 구조를 이해하면  
> GC 동작 방식, 메모리 오류(OOM, StackOverflow)의 원인을 명확히 파악할 수 있다.
