---
title: JVM 정리 노트 ② Java 프로그램은 어떻게 실행되는가
date: 2026-01-04 19:00:00 +0900
categories: [CS, JVM]
tags: [jvm, java, runtime, gc, memory]
---

## `.java → .class → 실행` 흐름

1. **소스 코드 작성 (`.java`)**
  - 개발자가 Java 소스 파일을 작성한다.
  - 사람이 읽을 수 있는 고수준 언어이다.

2. **컴파일 (`.java → .class`)**
  - `javac` 컴파일러가 `.java` 파일을 **바이트코드(bytecode)** 로 변환한다.
  - 생성된 `.class` 파일은 JVM이 이해할 수 있는 중간 코드로, 플랫폼에 독립적이다.

3. **실행**
  - `java` 명령어를 통해 JVM이 실행된다.
  - JVM은 `.class` 파일을 메모리로 로드하고 바이트코드를 실행한다.

이러한 구조로 인해 Java는  
**Write Once, Run Anywhere** 특성을 가진다.

---

## Class Loader 개념

JVM은 실행 시 필요한 클래스들을 **Class Loader**를 통해 메모리로 로드한다.  
Class Loader는 클래스를 **필요한 시점에 동적으로 로드**하며, 다음과 같은 계층 구조를 가진다.

- **Bootstrap Class Loader**
  - JVM 내부에서 동작
  - `java.lang`, `java.util` 등 핵심 클래스 로드

- **Extension Class Loader**
  - 확장 라이브러리 로드

- **Application Class Loader**
  - 애플리케이션에서 사용하는 클래스 로드

Class Loader는 **Delegation Model(위임 모델)** 을 사용하여  
하위 로더가 클래스를 직접 로드하기 전에 상위 로더에게 먼저 위임함으로써  
보안성과 안정성을 확보한다.

---

##  Runtime Data Area 간단 소개

JVM이 프로그램을 실행하면서 사용하는 메모리 영역을 **Runtime Data Area**라고 한다.

- **Method Area**
  - 클래스 메타데이터, 메서드 정보, static 변수 저장
  - 모든 스레드가 공유

- **Heap**
  - 객체와 배열이 생성되는 영역
  - Garbage Collection 대상
  - 모든 스레드가 공유

- **Stack**
  - 메서드 호출 시 생성되는 스택 프레임 저장
  - 지역 변수, 매개변수, 반환 주소 포함
  - 스레드별로 관리

- **PC Register**
  - 현재 실행 중인 JVM 명령어 주소 저장
  - 스레드별로 존재

- **Native Method Stack**
  - 네이티브(C/C++) 메서드 실행 시 사용

---

## Interpreter와 JIT Compiler 간단 비교

JVM은 바이트코드를 실행하기 위해 **Interpreter**와 **JIT Compiler**를 함께 사용한다.

- **Interpreter**
  - 바이트코드를 한 줄씩 해석하여 실행
  - 초기 실행 속도는 빠르나 반복 실행 시 성능이 낮음

- **JIT Compiler (Just-In-Time)**
  - 반복적으로 실행되는 바이트코드를 **네이티브 코드**로 변환
  - 이후 실행 시 높은 성능 제공
  - 런타임 중 성능 최적화 수행

JVM은 두 방식을 병행하여  
**초기 응답성과 장기 실행 성능을 모두 확보**한다.

---

> JVM의 실행 과정은  
> 클래스 로딩, 메모리 관리, 실행 최적화가 유기적으로 결합된 구조이다.
