---
title: Hile - ComponentScope     
date: 2025-04-16
categories:
 - Android
tags:
 - Android
 - Hilt
 - ComponentScope
---

Hilt 에서 쓰이는 ComponentScope 에 대해서 알아봅니다. ( SingletonScope / ActivityRetainScope ... )  

<!-- more -->

### Component Scope

<aside>
💡내부적으로 여러개의 미리 정의된 Component 를 활용하여, Android 각 계층에 맞는 의존성 주입 구성
</aside>

- Application / Activity / Fragment / Service 등 다양한 라이프사이클에 맞춘 컴포넌트 존재
- 해당 컴포넌트들의 생명 주기에 맞게 Scope 가 각각 존재

### SingletonScope

<aside>
💡앱이 실행되는 동안 단 하나의 인스턴스가 생성되어 공유 ( @Singleton )
</aside>

- 앱이 시작 될 때 생성되고, 앱 종료 시 까지 유지됨
- 사용처
  - Retrofit / OkHttpClient 등 전역에서 하나만 존재해야 하는 글로벌 인스턴스를 만들 때 활용
  - 앱 전반에 동일한 데이터를 제공 해야 하는 캐싱매니저 등을 만들 때 활용
- 장점
  - 한번 생성된 객체를 여러 곳에서 재사용 하여 성능 효율성 높임
  - 동일 인스턴스 사용으로 상태 관리가 간편
- 단점
  - 애플리케이션 전체에 상주하므로, 메모리 낭비 우려가 있을 수 있음
  - Activity / Context 와 같은 UI 관련 객체를 Singleton 에 포함시키면 메모리 누수 위험

### ActivityRetainedScope

<aside>
💡Activity와 연관되지만, configurationChange 에도 유지되는 Scope ( @ActivityRetainedScoped )
</aside>

- 최초 Activity 생성 시 만들어지고, 구성 변경시 재생성 되지 않음
- ViewModel 같이 UI 상태를 보존 해야 할 때 사용
- 사용처
  - 네트워크 호출 / Repository / Data caching 등 구성 변경 후에도 동일한 인스턴스 유지 해야 하는 객체
  - 초기화 비용이 높은 객체를, 액티비티 생명주기 내에서 재사용 할 때
  - Repository / UseCase 등을 해당 Scope 으로 둘 수 있음
- 장점
  - 화면 회전 등 구성 변경 시에도 동일한 객체를 유지해 상태 손실 방지
- 단점
  - Activity 소멸시 까지 객체가 유지되므로, 불필요한 자원 공유 가능성이 존재
  - Activity 에 연결된 리소스 ( Context / View )를 실수로 참조하지 않도록 설계에 주의

### ActivityScope

<aside>
💡개별 Activity 객체에 연관된 Scope ( @ActivityScoped )
</aside>

- 개별 Activity 객체에 연관 됨
- 구성 변경 시엔, 새로운 객체가 생성 됨
- 사용처
  - Activity 내에서만 필요하고, 구성 변경 시 새로 생성 되어도 문제가 없는 객체 관리
  - 해당 Activity 에 국한되는 의존성 관리
- 장점
  - Activity 가 종료되면, 해당 객체들도 함께 소멸되어 메모리 관리가 쉬움
  - 상태 보존이 필요하지 않은 경우 간편하게 정리
- 단점
  - 화면 회전 등 구성 변경시마다 객체를 재생성

### FragmentScope

<aside>
💡개별 Fragment 객체에 연관된 Scope ( @FragmentScoped )
</aside>

- 개별 Fragment 객체에 연관 됨
- Fragment 생명 주기를 따름
- 사용처
  - Fragment 내에서 사용되는 모듈에 적용
  - Fragment 재생성시 초기화 과정을 진행 후 재사용
- 장점
  - Fragment 단위로 독립적인 의존성 관리가 가능해 모듈성 증가
  - Fragment 종료 시점에 맞춰 객체가 소멸되어 메모리 누수 위험 낮춤
- 단점
  - Fragment가 자주 생성되고 소멸하는 경우, 객체 초기화 비용 발생 가능

### ServiceScope

<aside>
💡Android Service 에 연동된 Scope ( @ServiceScoped )
</aside>

- Service 의 생명주기를 따름
- 사용처
  - 음악 재생, 위치 추적 등 백그라운드 작업에 적용
  - UI 와 별개로 지속적으로 실행되는 작업에 적합
  - long-running worker, alarmManager 등에 활용
- 장점
  - 서비스 라이프 사이클과 동일하여, 백그라운드 작업에 안정적
  - UI와 분리되어 관리됨으로써, 코드 구조가 단순해짐
- 단점
  - 서비스가 종료 될 때 의존성이 적절히 정리되는지 확인 필요