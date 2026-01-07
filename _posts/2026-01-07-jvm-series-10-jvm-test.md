---
title: JVM 정리 노트 ⑩ JVM 면접 질문 정리
date: 2026-01-07 17:00:00 +0900
categories: [CS, JVM]
tags: [jvm, java, runtime, gc, memory]
---

## 1) JVM 메모리 구조 설명해보세요

### 한 줄 요약
- JVM 메모리는 Heap, Stack, Method Area(Metaspace) 등으로 나뉘며 역할과 공유 범위가 다르다.

### 기본 답변
JVM Runtime Data Area는 Method Area(또는 Java 8 이후 Metaspace), Heap, Stack, PC Register, Native Method Stack으로 구성됩니다.  
Heap은 객체와 배열이 생성되는 영역으로 GC 대상이며, Stack은 스레드별로 생성되어 메서드 호출 프레임을 관리합니다.

### 꼬리질문 대비
- **Heap과 Stack 차이?**  
  Heap은 모든 스레드가 공유하는 객체 저장 영역이고 GC 대상입니다.  
  Stack은 스레드마다 독립적으로 생성되며 메서드 종료 시 자동으로 해제됩니다.

- **Metaspace와 PermGen 차이?**  
  PermGen은 고정 크기여서 OOM 문제가 잦았고, Java 8부터는 네이티브 메모리를 사용하는 Metaspace로 변경되어 동적 확장이 가능합니다.

- **OOM은 어떤 영역에서 발생하나요?**  
  Heap, Metaspace, 네이티브 스레드 영역 등 여러 곳에서 발생할 수 있으며 메시지에 따라 원인이 다릅니다.

---

## 2) Heap과 Stack 차이를 설명해주세요

### 한 줄 요약
- Heap은 객체 저장 공간, Stack은 호출 프레임 저장 공간이다.

### 기본 답변
Heap은 객체와 배열이 생성되는 공유 영역이며 GC 대상입니다.  
Stack은 스레드별로 관리되고 메서드 호출 시 Stack Frame이 쌓입니다.

### 꼬리질문 대비
- **Stack은 왜 GC 대상이 아닌가요?**  
  Stack은 호출과 함께 생성·소멸되어 생명주기가 명확하고 스레드 종료 시 OS가 정리합니다.

- **지역 변수의 객체 참조는 GC에 어떤 영향을 주나요?**  
  Stack의 지역 변수는 GC Root로 작용하며 참조가 살아 있는 동안 Heap 객체는 GC 대상이 아닙니다.

---

## 3) GC는 언제 발생하나요?

### 한 줄 요약
- Heap이 부족해질 때, 특히 Eden 또는 Old 영역 압박 시 발생한다.

### 기본 답변
객체는 Eden에 생성되고 Eden이 가득 차면 Minor GC가 발생합니다.  
오래 살아남은 객체는 Old로 이동하며, Old 영역이 부족하면 Major 또는 Full GC가 발생할 수 있습니다.

### 꼬리질문 대비
- **Minor / Major / Full GC 차이?**  
  Minor GC는 Young 영역, Major GC는 Old 영역, Full GC는 Heap 전체를 대상으로 합니다.

- **Promotion 실패 시?**  
  Old 영역 공간이 부족하면 Promotion Failure가 발생하고 Full GC로 이어질 수 있습니다.

- **GC 튜닝보다 우선할 것은?**  
  객체 생명주기 설계와 불필요한 객체 생성을 줄이는 것이 우선입니다.

---

## 4) Stop-The-World(STW)란?

### 한 줄 요약
- GC 수행 중 애플리케이션 스레드가 멈추는 시간이다.

### 기본 답변
GC가 안전하게 객체를 탐색·회수하기 위해 모든 애플리케이션 스레드를 일시 정지시키는 구간입니다.

### 꼬리질문 대비
- **STW를 줄이는 방법은?**  
  불필요한 객체 생성을 줄이고, 서비스 성격에 맞는 GC(G1 등)를 선택합니다.

- **G1이 latency에 유리한 이유는?**  
  Region 단위 수집과 Pause Time 목표 기반 동작으로 지연 시간을 예측 가능하게 관리합니다.

---

## 5) GC 종류 차이는?

### 한 줄 요약
- Serial/Parallel은 처리량 중심, G1은 응답성 중심이다.

### 기본 답변
Parallel GC는 처리량이 좋아 배치 작업에 적합하고,  
G1 GC는 STW 시간을 예측 가능하게 관리해 대규모 서비스에 적합합니다.

### 꼬리질문 대비
- **Batch는 왜 Parallel GC?**  
  응답 시간보다 전체 처리량이 중요하기 때문입니다.

- **CMS가 비권장인 이유는?**  
  단편화 문제와 유지관리 복잡성으로 deprecated 되었습니다.

---

## 6) Class Loader는 언제 클래스를 로드하나요?

### 한 줄 요약
- 클래스는 처음 참조되는 시점에 동적으로 로드된다.

### 기본 답변
객체 생성, static 접근, 리플렉션 시점에 Class Loader가 Loading → Linking → Initialization 단계를 수행합니다.

### 꼬리질문 대비
- **Verification / Preparation / Resolution?**  
  Verification은 검증, Preparation은 static 변수 기본값 할당, Resolution은 실제 참조 연결 단계입니다.

- **Delegation Model 장점?**  
  핵심 클래스 위조를 방지하고 보안성과 일관성을 확보합니다.

---

## 7) OutOfMemoryError 종류는?

### 한 줄 요약
- OOM은 부족한 자원 유형에 따라 원인이 다르다.

### 기본 답변
Heap space, Metaspace, native thread 등 메시지별로 원인과 대응이 다릅니다.

### 꼬리질문 대비
- **Heap dump에서 무엇을 보나요?**  
  Dominator Tree와 Retained Size를 확인합니다.

- **Thread dump에서 무엇을 보나요?**  
  BLOCKED, WAITING 상태와 락 경합 여부를 확인합니다.

---

## 8) JVM 성능 이슈 접근 방법은?

### 한 줄 요약
- GC, 스레드, IO 관점으로 분해한다.

### 기본 답변
GC 로그, 스레드 풀 상태, 외부 IO 병목을 순서대로 확인하고 근거 기반으로 원인을 좁힙니다.

### 꼬리질문 대비
- **GC 튜닝 vs 코드 개선?**  
  코드 개선이 우선이며 GC 튜닝은 마지막 단계입니다.

- **스레드 풀 고갈 확인법?**  
  Thread dump에서 WAITING/BLOCKED 비율과 큐 상태를 확인합니다.


