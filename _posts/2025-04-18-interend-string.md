---
title: JVM - Interned string      
date: 2025-04-18
categories:
 - Android
tags:
 - Android
 - Java
 - JVM
---

JVM 의 Interned string 에 대해서 알아봅니다.    

<!-- more -->

### Interned String

<aside>
💡 JVM이 동일한 내용의 문자열 리터럴을 한 번만 생성하고 재사용 하도록 관리하는 매커니즘
</aside>

- “Hello” 라는 문자열 리터럴이, 코드 어딘가에 여러 번 등장한다면 JVM 은 실제로 하나의 String 객체만 생성하고, 모든 참조가 이 하나의 객체를 공유하도록 만듬
- Interning 을 하는 이유 ?
  - 메모리 효율성
    - 같은 문자열 데이터를 여러 개 생성하는 대신, 하나의 객체를 공유 함으로써 힙 메모리의 중복 할당을 줄임
    - 자주 사용되는 문자열은 String 상수 풀 ( String pool ) 에 저장 되어, 필요 할 때마다 새로 객체를 생성하는 대신 이미 만들어진 객체를 재사용
  - 성능 개선
    - 두 문자열을 비교 할 때, equals() 대신 == 연산자 ( 객체 참조 비교 )를 사용해도, 동일 인스턴스라면 빠르게 동일성 판단 가능
    - 문자열은 Immutable 함으로, 중복 생성되는 비용을 줄여 GC 부담도 감소 됨
- 동작 방식
  - 문자열 리터럴
    - 예시 코드

        ```kotlin
        Stirng s1 = "Hello";
        Stirng s2 = "Hello";
        // s1, s2 는 동일한 객체를 참조하므로, s1 == s2 는 true 
        ```

      - 소스 코드에, “Hello” 와 같이 직접 쓴 문자열은 컴파일러에 의해 자동으로 상수 풀에 저장
      - JVM이 로드하면서 interned 진행
  - 명시적 intern
    - `String.intern()` 메서드 활용하면, 런타임에 특정 문자열 객체를 intern pool 에 등록 할 수는 있음

        ```kotlin
        String s1 = new String("Hello");
        String s2 = s1.intern();
        ```

    - 하지만 일반적으로 문자열 리터럴은 자동으로 intern 처리되므로, 특별한 성능이나 메모리 최적화가 필요한 경우에 한해 intern()을 신중하게 사용
- Java 8 이전 / 이후 동작 차이
  - Java 8 이전
    - Interned 문자열은 메소드 영역 내의 PermGen 영역에 저장 됨
    - 클래스 로딩 시점에, 상수 풀에 함께 포함되어 관리 됨
  - Java 8 이후
    - PermGen → MetaSpace 로 대체 됨에 따라, 힙 내부의 String Pool 영역에서 관리 됨
- 구현 방식
  - Intern Pool 은 HashTable 이나, 유사한 자료 구조를 사용하여 이미 존재하는 문자열의 내용을 기반으로 빠르게 찾을 수 있도록 관리 함
  - 만약 intern() 메서드가 호출될 때, 동일한 내용의 문자열이 이미 풀에 존재하면 그 참조를 반환하고, 없으면 현재 객체를 풀에 등록한 후 그 참조를 반환