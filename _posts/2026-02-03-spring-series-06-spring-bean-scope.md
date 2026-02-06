---
title: 스프링 정리 노트 ⑥ 빈 스코프 (Singleton, Prototype, Web Scope)
date: 2026-02-03 18:00:00 +0900
categories: [CS, Spring]
tags: [scope, singleton, prototype, request, session, provider]
---

스프링 컨테이너에서 관리되는 Bean은 생성 범위(Scope)에 따라 생명주기와
공유 방식이 달라진다.\
빈 스코프를 올바르게 이해하지 못하면 메모리 누수, 동시성 문제, 성능 저하
등의 문제가 발생할 수 있다.

이번 글에서는 Singleton, Prototype, Web Scope를 중심으로 빈 스코프의
개념과 활용 방법을 정리한다.

------------------------------------------------------------------------

## 1. 빈 스코프란?

빈 스코프란 **스프링 컨테이너가 빈을 생성하고 관리하는 범위**를
의미한다.

즉, 다음을 결정한다.

-   빈이 몇 개 생성되는가
-   언제 생성되는가
-   언제 소멸되는가
-   어디까지 공유되는가

스프링은 기본적으로 Singleton 스코프를 사용한다.

------------------------------------------------------------------------

## 2. 싱글톤 스코프 (Singleton Scope)

### 2.1 개념

-   스프링의 기본 스코프
-   컨테이너당 1개의 인스턴스만 생성
-   모든 요청이 동일 객체를 공유

``` java
@Component
public class OrderService {
}
```

별도 설정이 없으면 자동으로 Singleton이다.

------------------------------------------------------------------------

### 2.2 특징

-   메모리 효율이 높음
-   객체 생성 비용 감소
-   빠른 성능

하지만 동시성에 주의해야 한다.

------------------------------------------------------------------------

### 2.3 주의: 상태를 필드로 들고 있지 말 것

싱글톤 빈은 여러 스레드가 동시에 접근한다.

``` java
@Component
public class OrderService {

    private int price; // 위험

    public void order(int price) {
        this.price = price;
    }
}
```

이렇게 상태를 필드에 저장하면 동시성 문제가 발생한다.

#### 해결 방법

-   지역 변수 사용
-   파라미터 전달 방식 사용
-   무상태(Stateless) 설계

``` java
public int order(int price) {
    return price;
}
```

------------------------------------------------------------------------

## 3. 싱글톤 내부에서 프로토타입 사용 문제

싱글톤 빈에서 프로토타입 빈을 주입하면 문제가 발생한다.

``` java
@Component
public class ClientService {

    @Autowired
    private PrototypeBean prototypeBean;
}
```

초기 주입 시점에만 생성되고 이후에는 재사용된다.

즉, 프로토타입이 의미가 없어진다.

------------------------------------------------------------------------

## 4. 해결 방법

### 4.1 Proxy Mode

프록시 객체를 통해 매번 새 객체를 생성한다.

``` java
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
@Component
public class PrototypeBean {
}
```

장점: - 코드 수정 최소화 - 자동 관리

단점: - 프록시 비용 발생

------------------------------------------------------------------------

### 4.2 Dependency Lookup + ObjectProvider

필요한 시점에 직접 조회한다.

``` java
@Component
public class ClientService {

    @Autowired
    private ObjectProvider<PrototypeBean> provider;

    public PrototypeBean getBean() {
        return provider.getObject();
    }
}
```

장점: - 명확한 제어 - 가볍고 안전

실무에서 가장 많이 사용된다.

------------------------------------------------------------------------

### 4.3 JSR-330 Provider

표준 방식 Provider 사용

``` java
import javax.inject.Provider;

@Component
public class ClientService {

    @Autowired
    private Provider<PrototypeBean> provider;

    public PrototypeBean getBean() {
        return provider.get();
    }
}
```

장점: - 스프링 의존도 감소 - 표준 기반

------------------------------------------------------------------------

## 5. 프로토타입 스코프 (Prototype Scope)

### 5.1 개념

-   요청마다 새 객체 생성
-   컨테이너가 소멸 관리하지 않음

``` java
@Scope("prototype")
@Component
public class PrototypeBean {
}
```

------------------------------------------------------------------------

### 5.2 특징

-   상태 관리 가능
-   독립 객체 사용
-   메모리 사용 증가

개발자가 직접 소멸 관리해야 한다.

------------------------------------------------------------------------

### 5.3 소멸 관리

``` java
@PreDestroy
public void destroy() {
}
```

Prototype에서는 자동 호출되지 않는다.

필요 시 직접 호출해야 한다.

------------------------------------------------------------------------

## 6. 웹 스코프 (Web Scope)

웹 환경에서만 사용 가능한 스코프이다.

Spring MVC 환경에서만 동작한다.

------------------------------------------------------------------------

### 6.1 request

-   HTTP 요청당 1개 생성
-   요청 종료 시 소멸

``` java
@Scope(value = "request")
@Component
public class RequestBean {
}
```

활용 예: 요청 로그, 인증 정보

------------------------------------------------------------------------

### 6.2 session

-   세션당 1개 생성
-   로그아웃 시 소멸

``` java
@Scope(value = "session")
@Component
public class SessionBean {
}
```

활용 예: 로그인 정보, 사용자 설정

------------------------------------------------------------------------

### 6.3 application

-   ServletContext 단위 공유
-   애플리케이션 종료 시 소멸

``` java
@Scope("application")
@Component
public class AppBean {
}
```

------------------------------------------------------------------------

### 6.4 websocket

-   웹소켓 연결 단위 관리
-   실시간 통신 환경에서 사용

------------------------------------------------------------------------

## 7. 스코프별 비교

Scope         생성 기준   공유 여부   관리 주체
  ------------- ----------- ----------- -----------
Singleton     컨테이너    공유        Spring
Prototype     요청 시     미공유      개발자
Request       HTTP 요청   미공유      Spring
Session       세션        공유        Spring
Application   앱 전체     공유        Spring

------------------------------------------------------------------------

## 8. 실무 활용 전략

### 권장 패턴

-   기본: Singleton
-   상태 필요 시: Prototype
-   웹 정보: Request / Session

### 주의 사항

-   Singleton에 상태 저장 금지
-   Prototype 남용 금지
-   Session 메모리 관리 필수

------------------------------------------------------------------------

## 9. 정리

빈 스코프는 스프링 애플리케이션의 안정성과 직결되는 핵심 개념이다.

### 핵심 요약

✔ 기본은 Singleton\
✔ 상태는 지역 변수로 관리\
✔ Prototype은 수동 관리 필요\
✔ 웹 스코프는 목적에 맞게 사용

올바른 스코프 설계는 성능과 유지보수성을 크게 향상시킨다.
