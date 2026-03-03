---
title: 데이터베이스 핵심 개념 정리 ⑦ 옵티마이저와 실행계획
date: 2026-03-03 23:00:00 +0900
categories: [CS, Database]
tags: [Database, Optimizer, Explain, Index, Hint, Interview]
---

# 1. 옵티마이저(Optimizer)란?

옵티마이저는 **SQL을 가장 효율적인 실행 계획으로 변환하는 DB 내부 엔진**이다.

우리는 SQL을 작성하지만,
실제로 DB는 그 SQL을 **실행 계획(Execution Plan)** 으로 바꿔 실행한다.

즉,

> SQL 작성 ≠ 실제 실행 방식

그 사이에서 최적 경로를 선택하는 것이 옵티마이저다.

---

# 2. 옵티마이저의 동작 과정

1. SQL 파싱 (문법 체크)
2. 통계 정보 기반 비용 계산
3. 여러 실행 경로 후보 생성
4. 가장 비용이 낮은 실행 계획 선택

대부분의 RDBMS는 **Cost-Based Optimizer(CBO)** 방식을 사용한다.

---

# 3. 옵티마이저 힌트(Optimizer Hint)

옵티마이저가 선택한 실행 계획이 비효율적일 경우,
개발자가 강제로 실행 방식을 지정할 수 있다.

예시 (Oracle / MySQL)

```sql
SELECT /*+ INDEX(users idx_user_name) */
*
FROM users
WHERE name = 'kim';
```

힌트는 최후의 수단이다.
통계가 잘못됐거나 특수한 상황에서만 사용하는 것이 좋다.

---
# 4. 인덱스 힌트(Index Hint)

MySQL 예시:

```sql
SELECT *
FROM users USE INDEX (idx_user_name)
WHERE name = 'kim';
```

종류:

- USE INDEX
- FORCE INDEX
- IGNORE INDEX

무분별하게 사용하면 오히려 성능이 나빠질 수 있다.

---
# 5. 쿼리 실행계획 보는 법 (EXPLAIN)

MySQL 예시:

```sql
EXPLAIN SELECT * FROM users WHERE name = 'kim';
```
주요 컬럼:

- type → 접근 방식 (ALL, index, range, ref, const 등)
- key → 사용된 인덱스
- rows → 예상 스캔 행 수
- Extra → 추가 정보  

좋은 실행계획 예시:
- type = const / ref / range
- rows 낮음
- Using index

나쁜 실행계획:  
- type = ALL (Full Scan)
- rows 매우 큼
---
# 5. 실행계획 해석 포인트
-  Full Scan → 인덱스 검토
- rows 많으면 통계 확인
- Using temporary, Using filesort → 정렬/그룹 성능 문제 가능성


# 한 줄 정리

> 옵티마이저는 SQL 성능의 핵심이다.  
> 인덱스를 이해하지 못하면 실행계획을 해석할 수 없고 실행계획을 못 보면 성능 튜닝은 불가능하다.
