---
title: 스프링 정리 노트 ② 스프링의 3대 요소 (IoC/DI, PSA, AOP)
date: 2026-01-29 13:00:00 +0900
categories: [CS, Spring]
tags: [spring, ioc, di, psi, aop, transactional, proxy]
---


스프링을 “왜 쓰는가”를 가장 간단히 설명하는 축은 아래 3가지다.

1. **IoC/DI**: 객체 생성/연결을 컨테이너에 위임해 결합도를 낮춘다.
2. **PSA**: 특정 기술(JDBC, JPA, 트랜잭션 등)을 추상화해 코드가 기술에 덜 묶이게 한다.
3. **AOP**: 트랜잭션, 로깅처럼 공통 관심사를 비즈니스 코드에서 분리한다.

---

## 1. IoC / DI

### IoC(Inversion of Control)란?
제어의 역전. “내 코드가 객체를 관리한다”에서 “컨테이너가 객체를 관리한다”로 바뀐다.

- 개발자는 “무엇을 쓸지(의존성)”만 선언
- 스프링 컨테이너가 “언제/어떻게 생성하고 연결할지”를 결정

### DI(Dependency Injection)란?
의존성을 **외부에서 주입**하는 방식. DI를 통해 객체 간 결합도를 낮춘다.

---

### 의존 관계 주입 3가지 방법

#### 1) 생성자 주입(권장)
- 불변(필드 `final`)로 만들기 쉬움
- 테스트에 가장 유리
- 순환 참조를 비교적 빠르게 발견

```java
@Service
public class OrderService {
  private final OrderRepository repo;
  public OrderService(OrderRepository repo) {
    this.repo = repo;
  }
}
```

#### 2) Setter 주입
- 런타임에 의존성 교체 가능(대부분의 서비스 코드에서는 오히려 위험)
- 선택적 의존성에 사용

```java
@Service
public class OrderService {
  private OrderRepository repo;

  @Autowired
  public void setRepo(OrderRepository repo) {
    this.repo = repo;
  }
}
```

#### 3) 필드 주입(비권장)
- 테스트 어렵고, DI 컨테이너에 강하게 묶임
- 순환참조/초기화 타이밍 문제를 늦게 발견

```java
@Autowired
private OrderRepository repo;
```

---

### @Qualifier / @Primary

#### 문제 상황: 같은 타입 Bean이 여러 개일 때
```java
interface PayClient {}
@Component class KakaoPayClient implements PayClient {}
@Component class NaverPayClient implements PayClient {}
```

이때 `PayClient`를 주입하면 어떤 구현체를 넣을지 모호해진다.

#### 해결 1) @Primary
“기본값”을 정한다.

```java
@Primary
@Component
class KakaoPayClient implements PayClient {}
```

#### 해결 2) @Qualifier
“이름으로 지정”한다.

```java
@Service
class PayService {
  private final PayClient payClient;

  public PayService(@Qualifier("naverPayClient") PayClient payClient) {
    this.payClient = payClient;
  }
}
```

> 팁: 규모가 커지면 “이름 문자열”이 깨지기 쉬워서, 커스텀 애노테이션(@KakaoPay 같은)로 Qualifier를 래핑하는 방식도 많이 쓴다.

---

## 2. PSA(Portable Service Abstraction)

### PSA란?
스프링이 제공하는 **일관된 추상화 계층** 덕분에, 구현 기술이 바뀌어도 애플리케이션 코드를 크게 바꾸지 않아도 되는 구조.

### 대표 예시
- 데이터 접근: JDBC → JPA로 바꿔도 스프링의 트랜잭션/예외 체계는 유지
- 트랜잭션: 다양한 트랜잭션 매니저(JdbcTransactionManager, JpaTransactionManager 등)를 동일한 방식으로 사용

### “예외 변환”도 PSA의 대표
스프링은 JDBC/ORM의 기술 종속 예외를 `DataAccessException` 계열로 변환해 애플리케이션이 DB 벤더 예외에 덜 묶이도록 만든다.

---

## 3. AOP(Aspect Oriented Programming)

### AOP란?
애플리케이션의 핵심 기능(비즈니스 로직)과, 여러 곳에 반복되는 기능(공통 관심사)을 분리하는 방법.

- 공통 관심사: 트랜잭션, 로깅, 권한, 메트릭, 캐싱 등

---

### AOP는 프록시 기반이다 (데코레이터 패턴 느낌)
스프링 AOP는 보통 **프록시 객체**를 만들어서, 실제 타깃 객체 호출 전/후에 부가 기능을 끼워 넣는다.

```
Client → Proxy(부가기능) → Target(비즈니스)
```

---

### @Transactional은 왜 AOP인가?
`@Transactional`이 붙은 메서드를 호출하면, 스프링은 프록시를 통해 아래를 자동으로 수행한다.

1. 트랜잭션 시작
2. 메서드 실행
3. 예외 없으면 commit
4. 예외 있으면 rollback

---

### JDK Dynamic Proxy vs CGLIB

| 구분 | JDK Dynamic Proxy | CGLIB |
|---|---|---|
| 대상 | 인터페이스 기반 | 클래스 기반 |
| 원리 | java.lang.reflect.Proxy | 바이트코드 생성(서브클래싱) |
| 제약 | 인터페이스 필요 | final 클래스/메서드 프록시 어려움 |
| 사용 | 인터페이스가 있으면 선택 가능 | 인터페이스 없어도 가능 |

> 실무 팁: `@Transactional`이 “작동 안 한다” 이슈의 상당수는  
> **프록시를 거치지 않고 자기 자신 메서드를 호출(self-invocation)**했기 때문인 경우가 많다.

---

### AOP 용어 정리
- Aspect: 공통 기능 모음(클래스 단위)
- Join point: 끼어들 수 있는 지점(스프링 AOP에서는 주로 메서드 실행)
- Pointcut: 어떤 join point에 적용할지 조건
- Advice: 실제로 실행될 부가 로직
- Weaving: 적용 과정(스프링 AOP는 런타임 프록시 방식이 일반적)

---

### AOP 사용 예시
```java
@Aspect
@Component
public class LogAspect {

  @Pointcut("execution(* io.hhplus..*(..))")
  public void appMethods() {}

  @Around("appMethods()")
  public Object around(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    try {
      return pjp.proceed();
    } finally {
      long end = System.currentTimeMillis();
      System.out.println(pjp.getSignature() + " took " + (end - start) + "ms");
    }
  }
}
```

---

## 핵심 요약
- IoC/DI: 객체 생성/연결을 컨테이너가 담당 → 결합도 감소/테스트 용이
- PSA: 기술이 바뀌어도 애플리케이션 코드를 최대한 유지
- AOP: 공통 관심사 분리(대표: @Transactional), 프록시 기반으로 동작
