---
title: 스프링 정리 노트 ③ 빈 생명주기 (Bean Lifecycle)
date: 2026-01-31 13:00:00 +0900
categories: [CS, Spring]
tags: [bean, lifecycle, postconstruct, predestroy, initialization]
---

스프링 빈(Bean)은 컨테이너가 생성/주입/초기화/소멸까지 관리한다.  
그래서 “언제 어떤 콜백이 실행되는지”를 알면, 리소스 초기화/정리(커넥션, 스레드풀, 파일 핸들 등)를 안정적으로 처리할 수 있다.

---

## 전체 흐름(핵심 시퀀스)
1. 빈 인스턴스 생성
2. 의존성 주입(DI)
3. 초기화 콜백 실행
4. 사용
5. 컨테이너 종료 시 소멸 콜백 실행

---

## 초기화/소멸 콜백 방법 3종

### 1) 애노테이션 기반: @PostConstruct / @PreDestroy
```java
@Component
class CacheClient {

  @PostConstruct
  public void init() {
    // 초기화 (서버 연결, 데이터 로딩 등)
  }

  @PreDestroy
  public void close() {
    // 종료 전 정리
  }
}
```

### 2) 인터페이스 기반: InitializingBean / DisposableBean
```java
@Component
class CacheClient implements InitializingBean, DisposableBean {

  @Override
  public void afterPropertiesSet() {
    // init
  }

  @Override
  public void destroy() {
    // destroy
  }
}
```

### 3) 설정 기반: initMethod / destroyMethod
```java
@Configuration
class AppConfig {

  @Bean(initMethod = "init", destroyMethod = "close")
  public CacheClient cacheClient() {
    return new CacheClient();
  }
}
```

---

## 싱글톤 vs 프로토타입 소멸 차이
- 싱글톤: 컨테이너가 생성/소멸 모두 관리 → `@PreDestroy` 호출됨
- 프로토타입: 컨테이너는 생성까지만, 소멸은 관리하지 않음 → `@PreDestroy`가 호출되지 않을 수 있음

---

## 요약
- 생성 → 주입 → 초기화 → 사용 → 소멸
- 초기화/소멸은 `@PostConstruct/@PreDestroy`가 가장 일반적
- 프로토타입은 소멸 관리 X → 리소스 정리는 직접 처리 필요


