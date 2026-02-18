---
title: JPA 핵심 개념 정리 ④ 실무 설계 팁
date: 2026-02-16 21:00:00 +0900
categories: [CS, JPA]
tags: [JPA, Hibernate, N+1, FetchJoin, BatchSize, 트랜잭션, 동시성]
---


## 1. 연관관계 최소화

-   불필요한 양방향 관계 제거
-   조회는 쿼리로 해결하는 것이 유지보수에 유리

``` java
@Entity
class Member {
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "team_id")
  private Team team;
}
```

------------------------------------------------------------------------

## 2. 지연 로딩 기본 설정

-   모든 연관관계는 LAZY 권장

``` java
@Entity
class Order {
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "member_id")
  private Member member;
}
```

------------------------------------------------------------------------

## 3. DTO 기반 조회

-   엔티티 직접 반환 지양
-   응답 스펙은 DTO로 고정

``` java
public record OrderSummaryDto(Long orderId, String memberName) {}
```

``` java
@Query("""
  select new com.example.dto.OrderSummaryDto(o.id, m.name)
  from Order o join o.member m
""")
List<OrderSummaryDto> findOrderSummaries();
```

------------------------------------------------------------------------

## 4. 조회 전용 쿼리 분리 (CQRS 활용)

``` java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class OrderQueryService {
  private final OrderQueryRepository orderQueryRepository;

  public List<OrderSummaryDto> list() {
    return orderQueryRepository.findOrderSummaries();
  }
}
```

------------------------------------------------------------------------

## 5. Cascade / OrphanRemoval 신중히

``` java
@Entity
class Post {
  @OneToMany(mappedBy = "post", cascade = CascadeType.PERSIST, orphanRemoval = true)
  private List<Comment> comments = new ArrayList<>();
}
```

------------------------------------------------------------------------

## 6. N+1 방지 전략

``` java
@Query("""
  select o
  from Order o
  join fetch o.member
  where o.id = :id
""")
Optional<Order> findByIdWithMember(Long id);
```

------------------------------------------------------------------------

## 7. 페이징은 컬렉션 fetch join과 분리

``` java
@Query("""
  select o.id
  from Order o
  order by o.id desc
""")
Page<Long> findOrderIds(Pageable pageable);
```

------------------------------------------------------------------------

## 8. Batch Size 적용

application.properties:

    spring.jpa.properties.hibernate.default_batch_fetch_size=100

------------------------------------------------------------------------

## 9. 트랜잭션은 Service에서 관리

``` java
@Transactional
public void placeOrder(Long memberId) {
  // 변경 로직
}

@Transactional(readOnly = true)
public OrderSummaryDto getOrder(Long id) {
  // 조회 로직
}
```

------------------------------------------------------------------------

## 10. 벌크 업데이트는 별도 경로로

``` java
@Modifying
@Query("update Seat s set s.status = 'AVAILABLE' where s.holdExpiresAt < :now")
int releaseExpiredHolds(LocalDateTime now);
```

------------------------------------------------------------------------

## 11. 도메인 규칙은 엔티티에

``` java
@Entity
class Seat {

  @Enumerated(EnumType.STRING)
  private SeatStatus status;

  public void hold(Long userId, LocalDateTime expiresAt) {
    if (status != SeatStatus.AVAILABLE) {
      throw new IllegalStateException("not available");
    }
    this.status = SeatStatus.HELD;
  }
}
```

------------------------------------------------------------------------

## 12. 동시성 전략 포함

``` java
@Version
private Long version;
```

``` java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select s from Seat s where s.id = :id")
Optional<Seat> findByIdForUpdate(Long id);
```

------------------------------------------------------------------------

