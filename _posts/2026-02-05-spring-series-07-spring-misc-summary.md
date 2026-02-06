---
title: 스프링 정리 노트 ⑦ 기타 정리 (Spring vs Boot, 요청 바인딩 애노테이션)
date: 2026-02-05 18:00:00 +0900
categories: [CS, Spring]
tags: [springboot, autoconfiguration, requestbody, requestparam, modelattribute, pathvariable]
---


이번 글에서는 실무와 면접에서 자주 등장하는 주제인\
Spring vs Spring Boot 차이점과 요청 바인딩 애노테이션을 정리한다.

이 두 주제는 스프링 기반 개발자의 기본 소양에 해당한다.

------------------------------------------------------------------------

## 1. Spring Framework vs Spring Boot

### 1.1 Spring Framework란?

Spring Framework는 자바 기반 엔터프라이즈 애플리케이션 개발을 위한 핵심
프레임워크이다.

주요 기능:

-   DI (Dependency Injection)
-   AOP (Aspect Oriented Programming)
-   MVC
-   Transaction 관리
-   Validation
-   Security 연동

즉, 스프링의 뼈대 역할을 담당한다.

------------------------------------------------------------------------

### 1.2 Spring Framework의 한계

기존 Spring 기반 프로젝트는 다음과 같은 단점이 있었다.

-   복잡한 XML 설정
-   많은 환경 설정
-   라이브러리 직접 관리
-   서버 별도 설치 필요

프로젝트 초기 설정에 많은 시간이 소요되었다.

------------------------------------------------------------------------

### 1.3 Spring Boot란?

Spring Boot는 Spring Framework를 더 쉽게 사용하기 위한 프로젝트이다.

목표는 다음과 같다.

> "설정은 최소화하고, 바로 실행 가능한 환경 제공"

------------------------------------------------------------------------

### 1.4 Spring Boot의 핵심 기능

#### 1) Auto Configuration

의존성을 기반으로 자동 설정을 수행한다.

예:

-   spring-web → MVC 자동 설정
-   spring-data-jpa → JPA 자동 설정

개발자가 설정하지 않아도 동작한다.

------------------------------------------------------------------------

#### 2) Starter 의존성

관련 라이브러리를 묶어서 제공한다.

``` xml
spring-boot-starter-web
spring-boot-starter-data-jpa
spring-boot-starter-security
```

버전 충돌을 방지할 수 있다.

------------------------------------------------------------------------

#### 3) Embedded Server

내장 톰캣, 제티, 언더토우 제공

``` bash
java -jar app.jar
```

단독 실행 가능하다.

------------------------------------------------------------------------

#### 4) Actuator

운영 모니터링 기능 제공

-   health
-   metrics
-   env
-   beans

운영 환경 관리에 필수적이다.

------------------------------------------------------------------------

### 1.5 Spring vs Spring Boot 비교

항목     Spring      Spring Boot
  -------- ----------- -------------
설정     수동        자동
서버     외부 필요   내장
배포     복잡        단순
생산성   보통        높음

------------------------------------------------------------------------

## 2. 요청 바인딩이란?

요청 바인딩(Request Binding)이란,

HTTP 요청 데이터를 자바 객체로 변환하는 과정을 의미한다.

Spring MVC는 이를 자동으로 처리해준다.

------------------------------------------------------------------------

## 3. @RequestBody

### 3.1 개념

HTTP Body(JSON/XML)를 객체로 변환한다.

HttpMessageConverter를 사용한다.

------------------------------------------------------------------------

### 3.2 사용 예시

``` java
@PostMapping("/users")
public void save(@RequestBody UserRequest request) {
}
```

요청 예시:

``` json
{
  "name": "kim",
  "age": 30
}
```

------------------------------------------------------------------------

### 3.3 특징

-   주로 REST API에서 사용
-   POST / PUT / PATCH와 함께 사용
-   JSON 기반 통신

------------------------------------------------------------------------

### 3.4 주의사항

-   GET 요청에서는 권장하지 않음
-   Content-Type 필수 설정

```{=html}
<!-- -->
```
    application/json

------------------------------------------------------------------------

## 4. @RequestParam

### 4.1 개념

쿼리 스트링 또는 form 데이터를 바인딩한다.

------------------------------------------------------------------------

### 4.2 사용 예시

``` java
@GetMapping("/users")
public List<User> list(
    @RequestParam String name,
    @RequestParam int age
) {
}
```

요청 예시:

    /users?name=kim&age=30

------------------------------------------------------------------------

### 4.3 옵션 설정

``` java
@RequestParam(required = false, defaultValue = "0") int page
```

-   required
-   defaultValue

설정 가능

------------------------------------------------------------------------

### 4.4 활용 사례

-   검색 조건
-   필터
-   페이징
-   정렬

------------------------------------------------------------------------

## 5. @ModelAttribute

### 5.1 개념

여러 파라미터를 객체로 묶어서 바인딩한다.

------------------------------------------------------------------------

### 5.2 사용 예시

``` java
@GetMapping("/users")
public List<User> list(@ModelAttribute UserSearch search) {
}
```

``` java
public class UserSearch {
    private String name;
    private int age;
}
```

------------------------------------------------------------------------

### 5.3 특징

-   내부적으로 RequestParam 사용
-   DTO 기반 바인딩
-   가독성 향상

------------------------------------------------------------------------

### 5.4 자동 생략

GET 요청에서는 생략 가능하다.

``` java
public List<User> list(UserSearch search)
```

------------------------------------------------------------------------

## 6. @RequestParam vs @PathVariable

### 6.1 @PathVariable

URL 경로의 일부를 변수로 추출한다.

``` java
@GetMapping("/users/{id}")
public User find(@PathVariable Long id) {
}
```

특징:

-   리소스 식별 목적
-   RESTful 설계 핵심

------------------------------------------------------------------------

### 6.2 @RequestParam

``` java
/users?status=ACTIVE
```

특징:

-   검색 / 조건 / 옵션 전달
-   선택적 값

------------------------------------------------------------------------

### 6.3 사용 기준

구분   PathVariable   RequestParam
  ------ -------------- --------------
목적   식별           조건
필수   대부분 필수    선택적
의미   리소스         옵션

------------------------------------------------------------------------

## 7. 바인딩 애노테이션 선택 기준

### 권장 패턴

상황             애노테이션
  ---------------- -----------------
REST 저장/수정   @RequestBody
검색/필터        @RequestParam
검색 DTO         @ModelAttribute
단건 조회        @PathVariable

------------------------------------------------------------------------

## 8. 실무 활용 예제

``` java
@PostMapping("/orders")
public void create(@RequestBody OrderRequest req) {
}

@GetMapping("/orders/{id}")
public Order get(@PathVariable Long id) {
}

@GetMapping("/orders")
public List<Order> list(@ModelAttribute OrderSearch search) {
}
```

------------------------------------------------------------------------

## 9. 면접 핵심 포인트

✔ Spring과 Boot 차이 설명 가능\
✔ Auto Configuration 원리 이해\
✔ 바인딩 방식 차이 구분\
✔ REST URL 설계 기준 이해

------------------------------------------------------------------------

## 10. 정리

Spring Boot는 Spring을 더 쉽게 사용하기 위한 도구이며,\
요청 바인딩 애노테이션은 REST API 설계의 핵심 요소이다.

### 핵심 요약

✔ Spring = 기반 기술\
✔ Boot = 생산성 도구\
✔ Body = 객체 변환\
✔ Param = 옵션 전달\
✔ Path = 리소스 식별

이 개념들을 정확히 이해하면 실무와 면접에서 큰 강점이 된다.

