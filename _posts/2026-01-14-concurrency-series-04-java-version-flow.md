---
title: 동시성 정리 노트 ④ Java 버전 흐름
date: 2026-01-14 13:00:00 +0900
categories: [CS, Concurrency]
tags: [java, thread, executor, completablefuture]
---


이 글은  
**“Java는 왜 이렇게 스레드 사용 방식이 바뀌어 왔는가?”**를 이해하는 데 목적이 있다.

핵심 질문은 하나다.

> **스레드를 직접 관리하는 책임을  
> 개발자가 질 것인가, JVM/프레임워크에 맡길 것인가?**

---

## 4.1 Java 5 이전 – 스레드를 직접 다루던 시기

### Thread 상속

```java
class MyThread extends Thread {
    public void run() {
        // 작업 내용
    }
}
```

- 스레드 생성과 작업 정의를 동시에 수행
- “이 작업은 스레드다”라는 강한 결합
- `Thread` 클래스를 상속하여 스레드 생성
- `run()` 메서드 오버라이딩

**문제점**
- 상속 제한으로 구조 유연성 부족
- 작업 재사용 어려움
- 스레드 개수 제어 어려움

> 스레드를 *직접* 만드는 방식은  
> 규모가 커질수록 위험해진다.

---

### Runnable 구현

```java
Runnable task = () -> {
    // 작업 내용
};
new Thread(task).start();
```

- 작업(Runnable)과 실행(Thread)을 분리
- 구조는 좋아졌지만 근본 문제는 남아 있음
- `Runnable` 인터페이스 구현
- `run()` 메서드에 작업 정의

**장점**
- 작업과 스레드 분리
- 상속 제약 없음

**여전히 불편한 점**
- 반환값 없음
- 예외 처리 불편
- 스레드 생성/종료를 개발자가 직접 관리

> “작업은 늘어나는데,  
> 스레드는 어떻게 관리해야 하지?”라는 문제가 발생

---

## 4.2 Java 5 – 스레드 관리의 전환점

Java 5부터는 **“스레드를 직접 만들지 말라”**는 방향이 명확해진다.
`java.util.concurrent` 패키지가 도입되며  
스레드 관리 방식이 크게 개선되었다.

---

### Callable – 결과를 반환하는 작업

```java
Callable<Integer> task = () -> 1 + 2;
```

- 작업이 값을 반환할 수 있음
- 예외 처리 가능

> 이제 작업은  
> **“실행 단위”**로 표현된다.

---

### Future – 결과를 담는 상자

```java
Future<Integer> future = executor.submit(task);
```

- 작업 결과를 나중에 받기 위한 객체
- 아직 결과가 없을 수도 있음
- 비동기 작업 결과를 담는 객체
- 작업 완료 여부 확인 가능

**한계**
- `get()` 호출 시 스레드가 멈춤(Blocking)
- 여러 작업을 연결하기 어려움

---

### Executor / ExecutorService – 핵심 변화

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(task);
```

이 시점부터 Java의 철학은 이렇게 바뀐다.

> **“스레드는 만들지 말고, 맡겨라.”**

- 스레드 풀 내부에서 스레드 재사용
- 개발자는 *작업*만 제출

---

### Thread Pool 개념을 쉽게 말하면

- 스레드를 미리 만들어두고
- 작업이 오면 꺼내서 사용
- 작업이 끝나면 다시 반납

> ㅋㅋ스레드는 **자원**이고  
> 무한히 만들 수 있는 존재가 아니다.

---

## 4.3 Java 7 – CPU를 더 잘 쓰기 시작하다

### Fork/Join Framework

- 큰 작업을 작은 작업으로 나눔
- 여러 CPU 코어에 분산 처리

> “하나의 큰 일을  
> 여러 사람이 나눠서 동시에 처리”

---

### RecursiveTask / RecursiveAction

- `RecursiveTask` : 결과가 필요한 경우
- `RecursiveAction` : 결과 없이 처리만 하는 경우

```java
class SumTask extends RecursiveTask<Integer> {
    protected Integer compute() {
        // 작업을 쪼개거나 직접 처리
        return 0;
    }
}
```

**특징**
- CPU 집약적인 작업에 적합
- 구조가 복잡해 실무 사용 빈도는 제한적

---

## 4.4 Java 8 – 비동기를 사람이 이해할 수 있게

### CompletableFuture 등장 배경

Future의 가장 큰 문제는 이것이다.

> “작업이 끝나면 뭘 할지”를  
> 자연스럽게 표현할 수 없다.

---

### CompletableFuture – 흐름을 코드로 표현

```java
CompletableFuture
    .supplyAsync(() -> "data")
    .thenApply(data -> data + " processed")
    .thenAccept(System.out::println);
```

이 코드는 이렇게 읽으면 된다.

> 1. 데이터를 비동기로 가져오고  
> 2. 가공한 뒤  
> 3. 결과를 사용한다

---

### 기존 Future와의 차이

| 관점 | Future | CompletableFuture |
|---|---|---|
| 결과 처리 | 직접 `get()` 호출 | 콜백으로 연결 |
| 스레드 | 결과 대기 시 멈춤 | 멈추지 않음 |
| 코드 표현 | 절차적 | 흐름 중심 |

---

## 정리 – Java 스레드의 진화 방향

Java의 스레드 모델은 다음 흐름으로 발전했다.

1. 스레드를 직접 다루던 시대  
2. 스레드 관리를 프레임워크에 위임  
3. 비동기 흐름을 사람이 이해할 수 있게 표현

---

### 이 장의 핵심 문장

> **Java는 스레드를 잘 쓰는 법이 아니라  
> 스레드를 덜 신경 쓰게 만드는 방향으로 발전해왔다.**
