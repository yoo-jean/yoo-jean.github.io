---
title: 동시성 정리 노트 ② Java 스레드 모델
date: 2026-01-11
categories: [CS, Concurrency]
tags: [thread, concurrency, daemon, threadlocal]
---

Java의 스레드 모델은 JVM 내부 구조와 OS 스레드 모델을 이해하는 데서 시작된다.  
Java 스레드의 동작 방식, 데몬 스레드, 그리고 ThreadLocal까지 정리한다.

---

## 2.0 OS Thread란 무엇인가

### OS Thread의 정의
**OS Thread(운영체제 스레드)** 는  
운영체제가 직접 생성·관리·스케줄링하는 **실행의 최소 단위**이다.

- CPU가 실제로 실행하는 대상
- 커널 레벨에서 관리됨

> CPU는 프로세스를 실행하지 않고, **스레드를 실행한다**

---

### 프로세스와 OS Thread의 관계
- **프로세스(Process)**
  - 자원 소유 단위 (메모리, 파일, 소켓 등)
- **스레드(Thread)**
  - 실행 단위

하나의 프로세스는 **여러 OS Thread**를 가질 수 있다.

---

### OS Thread의 특징
- 커널이 직접 관리
- 각 스레드는 다음을 독립적으로 가짐
  - Stack
  - Program Counter
  - 레지스터 상태
- 같은 프로세스 내 스레드는 다음을 공유
  - Heap
  - 전역 변수
  - 파일 디스크립터

 이로 인해 **동기화 문제**가 발생한다.

---

## 2.1 Java 스레드의 동작 방식

### Java Thread ↔ OS Thread (1:1 모델)
- Java 스레드는 **OS 네이티브 스레드와 1:1로 매핑**
- JVM은 스레드를 직접 스케줄링하지 않음
- 실제 실행과 스케줄링은 **OS가 담당**

```text
Java Thread 1개
    ↓
OS Thread 1개
```

---

### 왜 Java는 OS Thread를 사용하는가?
**장점**
- 멀티코어 CPU 활용 가능
- OS 스케줄링 정책 그대로 사용
- 성능 예측이 비교적 쉬움

**단점**
- 스레드 생성 비용 큼
- 스레드 수 증가 시 컨텍스트 스위칭 비용 증가

이 한계를 보완하기 위해
- 스레드 풀
- 비동기 모델
- 가상 스레드(Virtual Thread)가 등장

---

### JVM Stack과 Thread의 관계
- **스레드마다 독립적인 JVM Stack**을 가진다.
- Stack에는 다음 정보가 저장된다:
  - 메서드 호출 프레임(Stack Frame)
  - 지역 변수
  - 매개변수
  - 리턴 주소

 스레드 간 Stack은 **절대 공유되지 않음**  
 동시성 문제는 주로 Heap 영역에서 발생

---

### Thread 생명주기
Java 스레드는 다음 상태를 가진다:

1. **NEW**
2. **RUNNABLE**
3. **BLOCKED**
4. **WAITING**
5. **TIMED_WAITING**
6. **TERMINATED**

---

### main thread
- JVM 시작 시 자동 생성
- `main()` 메서드를 실행하는 최초의 사용자 스레드
- main thread 종료 시
  - 사용자 스레드가 남아 있으면 JVM 종료되지 않음
  - 데몬 스레드만 남아 있으면 JVM 종료

---

## 2.2 데몬 스레드 (Daemon Thread)

### 데몬 스레드란?
- 백그라운드 작업용 스레드
- JVM 종료 여부를 결정하지 않음
- 모든 사용자 스레드 종료 시 자동 종료

```java
Thread t = new Thread(() -> {
    while (true) {
        // background task
    }
});
t.setDaemon(true);
t.start();
```

`start()` 호출 전만 설정 가능

---

### 사용자 스레드와의 차이

| 구분 | 사용자 스레드 | 데몬 스레드 |
|---|---|---|
| JVM 종료 영향 | 있음 | 없음 |
| 주요 역할 | 비즈니스 로직 | 백그라운드 작업 |
| 강제 종료 가능성 | 낮음 | 높음 |

---

### 사용 사례
- Garbage Collector
- 로그 수집
- 모니터링
- 캐시 정리

⚠ 중요한 작업에는 사용 금지

---

## 2.3 ThreadLocal

### ThreadLocal이 필요한 이유
- 스레드마다 **독립적인 변수 저장 공간 제공**
- 동기화(synchronized) 없이 스레드 안전성 확보

```java
ThreadLocal<Integer> local = new ThreadLocal<>();
local.set(10);
Integer value = local.get();
```

---

### 스레드 간 데이터 격리
- ThreadLocal 값은 **Thread 내부 Map**에 저장
- 같은 ThreadLocal 객체라도 스레드마다 다른 값 유지

**대표 사용 예**
- 사용자 인증 정보
- 트랜잭션 컨텍스트
- Request ID
- 날짜 포맷터(SimpleDateFormat)

---


### 메모리 누수 주의사항
ThreadLocal은 **강한 참조 구조**로 인해 누수 위험이 있다.

**문제 원인**
- ThreadLocal key: WeakReference
- value: StrongReference
- 스레드가 종료되지 않으면 value가 GC되지 않음

**특히 위험한 경우**
- WAS 스레드 풀 환경
- ThreadLocal.remove() 미호출

```java
try {
    threadLocal.set(value);
    // logic
} finally {
    threadLocal.remove();
}
```

 반드시 `remove()` 호출 습관화

---

## 핵심 요약 
- OS Thread는 커널이 관리하는 실행 단위
- Java Thread는 OS Thread와 1:1 매핑
- Stack은 스레드마다 독립
- 데몬 스레드는 JVM 종료를 막지 않음
- ThreadLocal은 편리하지만 반드시 remove 필요
