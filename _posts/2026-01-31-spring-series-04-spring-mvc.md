---
title: 스프링 정리 노트 ④ Spring MVC 핵심 (Servlet, DispatcherServlet, Filter/Interceptor)
date: 2026-01-31 18:00:00 +0900
categories: [CS, Spring]
tags: [mvc, servlet, dispatcherservlet, filter, interceptor, frontcontroller]
---

## 1. 서블릿(Servlet)이란?
서블릿은 Java에서 HTTP 요청/응답을 처리하는 표준 컴포넌트다.  
서블릿 컨테이너(Tomcat 등)가 서블릿의 생명주기를 관리한다.

### 동작 방식(요약)
1. 요청 수신
2. 스레드 할당
3. `service()` 실행
4. 응답 반환

---

## 2. DispatcherServlet이란?
Spring MVC의 Front Controller로, 모든 요청의 진입점이다.

### 처리 흐름
1. 요청 수신(DispatcherServlet)
2. HandlerMapping으로 핸들러 조회
3. HandlerAdapter로 컨트롤러 호출
4. 컨트롤러 실행
5. ViewResolver 또는 ResponseBody로 응답 생성

---

## 3. Filter vs Interceptor

| 구분 | Filter | Interceptor |
|---|---|---|
| 레벨 | Servlet Container | Spring MVC |
| 위치 | DispatcherServlet 전 | DispatcherServlet 후 |
| 범위 | 전체 요청(정적 리소스 포함) | 컨트롤러 중심 |
| 훅 | doFilter | preHandle/postHandle/afterCompletion |

### 한 장 요약
```
Client
  ↓
[Filter Chain]
  ↓
DispatcherServlet
  ↓
(Interceptor)
  ↓
Controller
  ↓
Response
```
