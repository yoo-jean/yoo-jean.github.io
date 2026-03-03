---
title: 데이터베이스 핵심 개념 정리 ⑤ Lock
date: 2026-03-01 20:00:00 +0900
categories: [CS, Database]
tags: [JOIN, InnerJoin, OuterJoin, CrossJoin, SQL, DB, Query]
---

# 1. JOIN이란?

JOIN은 두 개 이상의 테이블을 연결하여 하나의 결과 집합으로 만드는
연산이다.

> 관계형 데이터베이스에서 테이블 간 관계를 기반으로 데이터를 조회하는
> 핵심 기능이다.

------------------------------------------------------------------------

# 2. 왜 JOIN이 필요할까?

예를 들어:

users 테이블

| id | name |
 |----|------|
| 1  | kim  |
| 2  | lee  |


orders 테이블

| order_id | user_id | price |
 |----------|---------|-------|
| 101      | 1       | 10000 |
| 102      | 2       | 20000 |



→  "kim이 주문한 내역"을 보고 싶다면?\
→  두 테이블을 연결해야 한다.

이때 사용하는 것이 JOIN이다.

------------------------------------------------------------------------

# 3. INNER JOIN

##  개념

두 테이블에서 **조건이 일치하는 행만 조회**

> 교집합 개념

##  예시

``` sql
SELECT u.name, o.order_id
FROM users u
INNER JOIN orders o
ON u.id = o.user_id;
```

##  결과
| name | order_id |
 |----|----------|
| kim  | 101      |
| kim  | 102      |


→ 주문이 없는 사용자(lee)는 조회되지 않음

------------------------------------------------------------------------

# 4. LEFT OUTER JOIN

##  개념

왼쪽 테이블의 모든 데이터 + 일치하는 오른쪽 데이터

> 왼쪽 기준 전체 조회

##  예시

``` sql
SELECT u.name, o.order_id
FROM users u
LEFT JOIN orders o
ON u.id = o.user_id;
```

##  결과
| name | order_id |
 |----|----------|
| kim  | 101      |
| kim  | 102      |
| lee  | NULL      |


→ 주문이 없는 사용자도 포함됨

------------------------------------------------------------------------

# 5. RIGHT OUTER JOIN

## 개념

오른쪽 테이블 기준 전체 조회

``` sql
SELECT u.name, o.order_id
FROM users u
RIGHT JOIN orders o
ON u.id = o.user_id;
```

→  orders는 모두 나오고 users가 없으면 NULL

------------------------------------------------------------------------

# 6. FULL OUTER JOIN

##  개념

양쪽 테이블 모두 포함\
일치하지 않으면 NULL 처리

``` sql
SELECT u.name, o.order_id
FROM users u
FULL OUTER JOIN orders o
ON u.id = o.user_id;
```

→  MySQL은 직접 지원하지 않음 (UNION 사용 필요)

------------------------------------------------------------------------

# 7. CROSS JOIN

##  개념

모든 경우의 수 조합 (카테시안 곱)

``` sql
SELECT *
FROM users
CROSS JOIN orders;
```

→  users 2명 × orders 2건 = 4건 결과

 실수로 조건 없이 JOIN하면 발생 가능

------------------------------------------------------------------------

# 8. JOIN 정리 비교

| JOIN 종류      | 설명       | 설명      |
 |--------------|----------|---------|
| INNER JOIN   | 교집합      | 조건 일치   |
| LEFT JOIN    | 왼쪽 전체    | 왼쪽 기준   |
| RIGHT JOIN   | 오른쪽 전체   | 오른쪽 기준  |
| FULL JOIN    | 양쪽 전체    | 전체      |
| CROSS JOIN   | 모든 조합    | 조건 없음   |



------------------------------------------------------------------------

# 9. 실무에서 중요한 포인트

✔ ON 조건을 명확히 작성\
✔ 인덱스가 없으면 JOIN 성능 급격히 저하\
✔ JOIN 순서와 실행계획 확인 필요\
✔ 불필요한 JOIN 지양

------------------------------------------------------------------------

# 한 줄 정리

> INNER JOIN은 교집합, OUTER JOIN은 한쪽 또는 전체 데이터를 포함하며,
> CROSS JOIN은 모든 조합을 생성하는 방식입니다.
