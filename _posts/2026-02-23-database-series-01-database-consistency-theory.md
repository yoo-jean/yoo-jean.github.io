---
title: 데이터베이스 핵심 개념 정리 ① 데이터베이스 이론 
date: 2026-02-23 20:00:00 +0900
categories: [CS, Database]
tags: [Database, ACID, CAP, PACELC, DistributedSystem, Consistency]
---


# 1. ACID

ACID는 전통적인 **RDB 트랜잭션의 안정성 보장 원칙**이다.  
단일 노드 또는 강한 일관성을 유지하는 시스템에서 핵심이 되는 개념이다.

> Atomicity  
> Consistency  
> Isolation  
> Durability

---

## 1.1 Atomicity (원자성)

> "All or Nothing"

트랜잭션은 전부 성공하거나, 전부 실패해야 한다.

### 예시
은행 송금:
- A 계좌 -100
- B 계좌 +100

중간에 장애 발생 시?
→ 둘 다 롤백되어야 한다.

### 구현 방식
- Undo Log
- Rollback
- WAL(Write-Ahead Logging)

---

## 1.2 Consistency (일관성)

트랜잭션이 실행되기 전과 후에 데이터베이스는 항상 **정의된 규칙을 만족하는 상태**여야 한다.

즉,
- 제약 조건을 위반하면 안 되고
- 비즈니스 규칙을 깨면 안 된다

예를 들어:

- 잔액은 음수가 될 수 없다
- PK는 중복될 수 없다
- FK는 존재하지 않는 값을 참조할 수 없다

이 규칙을 깨는 트랜잭션은 **commit 될 수 없다.**

여기서 중요한 점은:

> ACID의 Consistency는 "데이터 무결성" 관점이다.  
> (분산 시스템에서 말하는 CAP의 Consistency와는 완전히 다른 개념이다.)

---

## 1.3 Isolation (격리성)

동시에 실행되는 트랜잭션이 서로 영향을 주지 않아야 한다.

### 발생 가능한 문제

- Dirty Read
- Non-repeatable Read
- Phantom Read

### Isolation Level

| Level | Dirty Read | Non-repeatable | Phantom |
|-------|------------|---------------|---------|
| Read Uncommitted | O | O | O |
| Read Committed | X | O | O |
| Repeatable Read | X | X | O |
| Serializable | X | X | X |

> Serializable은 가장 안전하지만 성능이 낮다.

---

## 1.4 Durability (지속성)

Commit이 완료되면 데이터는 영구 보존되어야 한다.

### 구현
- WAL
- Redo Log
- fsync
- Replication

---

# 2. CAP 이론

CAP는 **분산 시스템에서의 트레이드오프 이론**이다.

> Consistency  
> Availability  
> Partition Tolerance

---

## 2.1 Partition Tolerance

네트워크 분할이 발생해도 시스템은 동작해야 한다.

현실에서는:
> 네트워크 장애는 반드시 발생한다.

따라서 실제 분산 시스템은 P를 기본 전제로 한다.

---

## 2.2 Consistency (CAP에서의 의미)

CAP에서 말하는 Consistency는  
" 모든 노드가 같은 시점의 같은 데이터를 반환해야 한다 " 는 의미다.

예를 들어 서버가 2대 있다고 가정하자.

사용자가 서버 A에서 데이터를 수정했다.

그 직후 서버 B에 요청했을 때도  
반드시 수정된 최신 데이터가 보여야 한다.

이것이 CAP에서 말하는 Strong Consistency다.

즉,

- 어떤 서버에 요청해도
- 항상 최신 데이터가 보장되어야 한다

이 Consistency는
ACID에서 말하는 제약 조건 유지와는 전혀 다른 개념이다.

---

## 2.3 Availability

모든 요청에 대해 응답을 반환해야 한다.  
(에러가 아닌 정상 응답)

---

## 2.4 CAP Trade-off

문제는 네트워크 장애(Partition)가 발생했을 때다.

예를 들어 서버 A와 서버 B가 있는데,
두 서버 사이의 네트워크가 끊어졌다고 가정해보자.

이때:

- 서버 A는 최신 데이터
- 서버 B는 이전 데이터

를 가지고 있을 수 있다.

이 상황에서 우리는 선택해야 한다.

### 1) Consistency를 선택한다면 (CP)

- 최신 데이터가 아닐 경우 요청을 거부한다
- 일부 요청은 실패할 수 있다
- 대신 데이터는 항상 정확하다

→ 금융 시스템, 결제 시스템

### 2) Availability를 선택한다면 (AP)

- 최신 데이터가 아닐 수도 있지만 일단 응답은 한다
- 서비스는 멈추지 않는다
- 대신 일시적으로 데이터 불일치가 발생할 수 있다

→ SNS, 로그 시스템

즉,

> 네트워크 장애가 발생했을 때  
> "정확성"을 지킬 것인가  
> "서비스 지속성"을 지킬 것인가  
> 를 선택해야 한다는 의미다.

---

### CP 시스템 예
- HBase
- MongoDB (설정에 따라)

### AP 시스템 예
- Cassandra
- DynamoDB

---

# 3. PACELC 이론

CAP 이론은 네트워크 장애가 발생했을 때만 설명한다.

하지만 실제 시스템은
"장애 상황" 뿐 아니라
"정상 상황"에서도 선택이 필요하다.

이를 설명하는 것이 PACELC 이론이다.

PACELC는 이렇게 말한다:

If Partition (P)
→ Availability(A)와 Consistency(C) 중 선택

Else (정상 상황, E)
→ Latency(L)와 Consistency(C) 중 선택

즉,

네트워크 장애가 없더라도
항상 "응답 속도"와 "데이터 정확성" 사이에서 트레이드오프가 존재한다는 뜻이다.

예를 들어:

데이터를 여러 서버에 복제한 시스템에서

- 모든 서버에 동기 복제를 하면 → Consistency는 높지만 느리다
- 비동기 복제를 하면 → 빠르지만 일시적 불일치가 발생할 수 있다

이 선택을 설명하는 이론이 PACELC다.

---

## 3.1 의미

분산 시스템은:

- 장애 상황(P)
- 정상 상황(E)

모두에서 트레이드오프가 존재한다.

---

## 3.2 예시

### Cassandra
→ PA / EL
- Partition 발생 시 Availability 선택
- 정상 시에도 Latency 우선

### RDB Replication
→ PC / EC
- Partition 시 Consistency 선택
- 정상 시에도 Consistency 우선

---

# 4. ACID vs CAP 차이

| 구분 | ACID | CAP |
|------|------|------|
| 적용 대상 | 단일 DB 트랜잭션 | 분산 시스템 |
| Consistency 의미 | 제약 조건 유지 | 모든 노드 동일 데이터 |
| 초점 | 무결성 | 분산 트레이드오프 |

---

# 5. 실무 관점 정리 

### Q. ACID와 CAP의 차이는?

ACID는 단일 데이터베이스 트랜잭션의 무결성을 보장하는 원칙이고,  
CAP은 분산 시스템에서 네트워크 분할 상황에서 무엇을 포기할 것인지에 대한 이론입니다.

---

### Q. CAP에서 왜 P는 기본 전제인가?

현실의 분산 시스템에서는 네트워크 장애가 반드시 발생하기 때문에  
Partition Tolerance는 필수이며, 결국 Consistency(항상 최신데이터 보장)와 Availability(항상 응답보장 ) 중 하나를 선택해야 합니다.

---

### Q. PACELC는 왜 필요한가?

CAP은 Partition 발생 시만 설명하지만,  
PACELC는 정상 상황에서도 Latency와 Consistency 사이의 선택이 존재함을 설명합니다.

---

# 6. 내 관점 정리

결국 정답은 없다.

어떤 비즈니스인지에 따라 선택이 달라진다.

- 금융 시스템 → 데이터 정확성이 절대적 → ACID + CP
- 좌석 예약 시스템 → 동시성 중요 → CP 설계 고려
- SNS / 로그 시스템 → 서비스 중단이 더 치명적 → AP
- 대규모 캐시 시스템 → Eventually Consistent 허용

중요한 것은

> 이론을 아는 것이 아니라  
> 상황에 맞게 선택할 수 있는 설계 능력이다.

---

# 정리

ACID → 트랜잭션 안정성  
CAP → 분산 시스템 트레이드오프  
PACELC → 장애 + 정상 상황 모두 고려한 확장 이론

---
