---
title: JVM - Runtime Data Area - Method      
date: 2025-04-19
categories:
 - Android
tags:
 - Android
 - Java
 - JVM
---

JVM 의 Runtime Data Area 중 Method 영역에 대해서 알아봅니다.     

<!-- more -->

### Runtime Data Area

<aside>
💡JVM 이 어플리케이션 실행 중에 사용하는 메모리 영역들을 의미
</aside>

- JVM 은 프로그램을 실행 할 때, 여러가지 데이터와 코드를 저장 / 관리 하기 위해 여러 구역으로 메모리를 나누어 사용
- Method / Heap / Java Stack / PC Register / Native Method Stack 등이 존재

### Method 영역

<aside>
💡모든 클래스와 인터페이스의 설명 정보 ( MetaData )를 저장 함
</aside>

- JVM이 클래스를 로드 할 때, 해당 클래스의 바이트코드, 런타임 상수 풀, 필드, 메서드 정보 등이 이 영역에 저장 됨
- Java 8 이전 / 이후
  - Java 8 이전

    ![1.png](/assets/images/posts/2025-04-19-method-area/1.png)

    - 메소드 영역이 Permanent Generation ( PermGen ) 이라는 이름으로 힙과 별도로 관리 됨
    - Pergem 특징
      - 고정된 크기
        - 고정된 메모리 크기로 할당 됨
        - -XX:PermSize / -XX:MaxPermSize 로 시작, 최대 크기를 지정 할 수는 있었으나, 할당 가능한 메모리의 한계로 너무 많은 클래스를 로드하거나 동적 생성이 많으면 OOM 발생 가능
      - PermGen 영역은, 고정된 크기와 한정된 공간 때문에 메모리 단편화 문제에 취약
      - PermGen 에 저장되는 클래스 메타데이터는, 주로 클래스 로더에 의해 관리되었는데 클래스 로더의 언로드가 잘 이루어지지 않으면, PermGen 메모리가 회수되지 않고 누적 될 수 있었음
      - 문자열 상수 풀도 PermGen 에 저장 → 많은 문자열 상수가 생성되면 PermGen 메모리 부족 문제가 발생 할 수 있었음
  - Java 8 이후

    ![2.png](/assets/images/posts/2025-04-19-method-area/2.png)

    - PermGen 이 제거되고, 메타 데이터는 MetaSpace 라는 별도의 영역 ( 네이티브 메모리 영역 ) 에서 관리
    - MetaSpace
      - PermGen 을 대체하기 위해, Metaspace 라는 새로운 메모리 영역이 도입
      - Metaspace 는 메타데이터와 관련 정보를 저장하는 역할은 동일하지만, 할당되는 메모리가 NativeMemory 영역에 존재
      - 동적 크기
        - 기본적으로 네이티브 메모리에서 관리되고, 기본 크기가 제한되지 않고 필요에 따라 자동으로 확장 됨
        - -XX:MaxMetaspaceSize 옵션으로 제한을 둘 수는 있으나, 기본 설정은 PermGen 처럼 고정되어 있지는 않음
      - 메모리 관리 효율
        - Metaspace 는 네이티브 메모리 상에서 관리되므로, Java 힙과 별도로 관리되고 / GC의 대상도 아님
        - 클래스 언로드 정책도 개선되어, 사용되지 않는 클래스의 메타 데이터가 더 효율적으로 해제 됨
      - 문자열 상수 풀 변경
        - interned 문자열이 MetaSpace 가 아닌, Java Heap 내의 String Pool 로 이동 됨
        - 문자열 상수로 인한 PermGen 메모리 부족 문제 해결
      - JVM 튜닝 방식 변경
        - Java 8 이전에는, PermGen 크기를 직접 조정해야 했음
        - MetaSpace 는 운영체제의 네이티브 메모리 제약을 따르므로, 사용자가 직접 관리해야 하는 부담이 다소 줄어듬