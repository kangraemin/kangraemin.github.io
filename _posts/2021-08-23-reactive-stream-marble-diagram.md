---
title: RxJava 강의 3 - Stream이란 / Operator란 / 마블 다이어그램 보는법
date: 2021-08-23
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava에서 Stream이 무엇인지, Stream의 상태엔 어떤 것이 있는지, Operator가 무엇이고 어떤 역하을 하는지, 마블 다이어그램은 무엇이고 Marble Diagram을 어떻게 보는 것인지 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Stream

- 데이터 변화 도중, 생긴 이벤트들을 관찰자들에게 알려주는 객체
- Observer Pattern에서 데이터의 상태변화에 따라 데이터를 방출해주는 Subject와 유사한 개념
- 데이터의 변화를 방출하는 객체인 스트림에는, 데이터 흐름에서 발생할 수 있는 다양한 경우들도 하나의 이벤트로 방출한다.
- RxJava2에는 여러가지 종류의 Stream이 존재한다. ( 추후에 추가로 다룰 예정 )
- 스트림에서 방출되는 이벤트로는 `onNext` / `onError` / `onComplete` 등이 있으며, Stream의 종류 마다 이벤트가 다를 수 있다.
- 정수기 필터를 달아 수도관의 물을 우리가 변형 할 수 있듯 Stream을 원하는 데로 변형 할 수 있다
- ReactiveX 라이브러리에는 Stream을 변형하는데 사용되는 함수가 이미 정의되어 있으며, 이를 Operator라 한다.

## 📖 마블 다이어그램

- Stream에서 방출되는 데이터가 RxOperator를 통해 어떻게 다루어 지는지를 묘사한 다이어그램
- 마블 다이어그램을 통해서 데이터의 변환 과정을 쉽게 이해 할 수 있음

# 📚 Stream

## 📖 Stream 이란 ?

 [반응형 프로그래밍 포스팅](https://kangraemin.github.io/android/2021/08/16/reactive-programming/)을 통해, 반응형 프로그래밍이 `데이터가 변경 되었을 때 어떤 작업을 서술하는 방식` 이라는 것을 공부했습니다. 

 또, [옵저버 패턴 포스팅](https://kangraemin.github.io/android/2021/08/16/observer-pattern/)을 통해, 옵저버 패턴에는 데이터의 변화를 관찰자들에게 알려주는 Subject와 Subject를 관찰하는 Observer가 있다는 것을 확인 했습니다. 

 그렇다면, 반응형 프로그래밍에서 데이터의 변화를 관찰자들에게 알려주는 역할을 하는것이 있어야 하는데, 그 역할을 해주는 것을 Stream 이라고 칭합니다. 

 즉, ReactiveX에서 Stream은, 관찰자들에게 데이터의 변화를 알려주는 역할을 합니다. 

## 📖 Stream의 역할

 위에서, 스트림은 데이터의 변화를 알려주는 역할을 진행한다고 말했습니다. 이는 맞는 말이지만, Stream이 데이터의 변화를 알려주는 역할만 진행한다는 것은 틀린 말입니다. 

 ReactiveX에서는 Stream은 데이터가 변화되었을 때, 변화된 데이터를 방출하는 것 뿐 아니라 좀 더 광범위한 역할을 수행하고 있습니다. 

 Stream에서 다루고 있는 이벤트로는, 데이터의 변화 ( 발행 ) / 데이터 흐름 도중 에러 발생 / 데이터 흐름 종료 등이 있습니다. 

 즉, 앞선 Observer Pattern 포스팅에서 Subject는 데이터가 변화 했을 때만 Observer들에게 데이터의 변화를 알리고, 데이터를 방출 했으나, ReactiveX에서 Stream은 데이터를 다루는 도중 발생한 에러도 하나의 이벤트로 Observer들에게 알리고, 데이터의 흐름이 종료되었다는 것도 Observer들에게 알리며, 데이터가 변화되었다는 이벤트도 Observer들에게 알립니다. 

 이를 대략적인 코드로 비교 해 보겠습니다

- Observer pattern 에서의 Subject / Observer interface

    ```kotlin
    interface Subject<T> {
    		fun subscribe(observer: Observer<T>)
    }
    ```

    ```kotlin
    interface Observer<T> {
        fun notifyDataIsChanged(value: T)
    }
    ```

- RxJava2 에서의 Stream ( ObservableSource ) / Observer interface

    ```kotlin
    interface Stream<T> {
        fun subscribe(observer: Observer<T>)
    }
    ```

    ```kotlin
    interface Observer<T> {
        fun onSubscribe(s: Disposable) // 추후에 설명 할 예정이며, 지금은 무엇인지 몰라도 됩니다.
        fun onNext(t: T)
        fun onError(t: Throwable)
        fun onComplete()
    }
    ```

    - onNext : 데이터가 성공적으로 발행 되었을 때 호출
    - onError : 데이터 발행 도중 Error condition이 발생 했을 때 호출
    - onComplete : 데이터 발행이 끝났을 때 호출

 위 코드에서 살펴볼 수 있듯, Stream을 구독하는 Observer 에서는, Stream에서 발행하는 이벤트를 모두 추적 하기 위해 onError / onComplete 등의 함수가 추가 된 것을 확인 할 수 있습니다. 

 정리하면, Stream은 데이터가 변화되었을 때 관찰자들에게 이를 알려 줄 뿐 아니라, 데이터 변화 도중 생긴 이벤트들을 관찰자들에게 알려주는 역할도 맡는 객체입니다. 

 Stream을 물이 흐르는 수도관에 비유 해 본다면, Stream을 구독하는 행위는 수도꼭지를 수도관에 연결하는 것으로 이야기 할 수 있습니다. 

 이 때, Observer Pattern이라면 수도꼭지가 일반 수도꼭지라, 물이 나올 때 물을 받을 수 있기만 합니다. 

 ReactiveX에서의 Stream 에서는 수도꼭지가 일반 수도꼭지가 아니라 물이 흐르는 관의 상태를 알 수도 있는 스마트 수도꼭지입니다. ( 물이 흐르는 관이 수도꼭지에 상태를 알려줍니다. ) 

 즉, ReactiveX의 수도꼭지는 물이 정상적으로 흐를 때 물을 받을 수 있을 뿐 아니라, 물이 흐르는데 문제가 생겼을 시 수도꼭지에 `Error !` 라는 문구를 띄울 수 있고, 물 흐름이 완료되었을 때 `Complete !` 라는 문구를 띄울 수 있는 것입니다. 

# 📚 Operator를 활용한 Stream의 변형

## 📖 Stream의 특성

 위에서, Stream을 물이 흐르는 수도관에 비유했습니다. 

 우리는 현실에서 수도관에 흐르는 물을 정수기 필터를 통해서 마실 수 있는 물로 바꾸거나, 저녁 8시 ~ 다음날 오전 8시까지는 수도관에 물이 하나도 흐르지 않게 하는 등 수도관에 흐르는 물의 흐름 / 상태를 변경 할 수 있습니다. 

## 📖 Operator란

 Stream의 특성에서 이야기 한 것 처럼, 우리는 수도관을 필요에 맞게 변형 했듯 Stream에서 발생하는 데이터 / 이벤트도 필요에 맞게 변형 할 수 있습니다. 

 ReactiveX 라이브러리에는 Stream에서 발생하는 이벤트들을 우리의 입맛에 맞게 변형 할 수 있는 함수를 구현 해 놓았고, 이를 Operator라 부릅니다. 

 Operator의 예시로는 조건식이 True일때만 관찰자들에게 데이터를 방출하는 `filter`, 방출된 데이터를 변형시켜주는 `map`, 방출된 데이터를 처리 할 Thread를 지정 해주는 `observeOn` 등이 있습니다. 

 이러한 operator의 종류들은 [공식문서](http://reactivex.io/documentation/operators.html) 에서 확인 할 수 있으며, 정말 다양한 operator가 이미 reactiveX 라이브러리에 존재합니다. 

 아직은 operator 라는 것이 있고, Operator를 통해 Stream에서 발행된 이벤트 / 데이터를 변형 할 수 있다는 것만 알고, 구체적으로 어떤 operator가 있는지는 앞으로 차근차근 살펴보도록 합시다. 

# 📚 Marble Diagram

## 📖 Marble Diagram 이란

 위에서 Stream은 데이터 변화 도중 생긴 이벤트들을 관찰자들에게 알려주는 역할을 하는 객체라고 했습니다. 

 그렇다면, 이 Stream이 어떨 때 데이터를 발행하고, 어떨 때 데이터 발행을 완료하는지를 한눈에 보기 쉽게 정리 해 놓는다면, 구독자가 특정 Stream의 행동 양태를 알아보기 쉬울 수 있습니다. 

 ReactiveX 공식 문서에 보면, Stream에서 데이터가 발행되는지, 발행된 데이터가 어떻게 처리되는지 등을 한눈에 보기 쉽게 Marble Diagram이라는 형식으로 정리 해 놓았습니다. 

 또한 Operator가 Stream의 이벤트를 어떤 식으로 변형 하는지도 Marble diagram을 통해서 확인 할 수 있어 Marble diagram을 통해 operator가 어떤 역할을 하는지 한눈에 알 수 있습니다. 

 따라서 Marble Diagram을 보면, Stream에 대한 정보를 쉽게 얻을 수 있습니다. 

## 📖 Marble Diagram 보는 법

 Marble Diagram에서는, 약속된 기호들이 표기됩니다. 

### 💡 화살표가 있는 실선

![Timeline](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled.png)

( 이미지 출처 : [https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5#59a5](https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5#59a5) )

- 먼저, Marble Digram에서 화살표가 있는 실선은 Timeline을 의미합니다.
- 왼쪽에서 오른쪽으로 시간이 흐름을 이야기 하며, 여기서 특정 이벤트가 발생하면 이 Timeline에 이벤트를 표기하는 방식으로 Marble Diagram을 그립니다.
- 즉, 이 Timeline은 시간이 흐름을 의미하는데 이를 활용하여 Stream을 표기 할 수 있습니다.
- Timeline에 Stream에서 발생한 이벤트를 표기한다면, 이 Timeline 자체를 하나의 Stream으로 생각해도 무방합니다.
- 즉, 화살표가 있는 실선은 Stream을 의미합니다.

### 💡 아이템 발행 / 발행 완료 / 에러 발생 이벤트

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%201.png)

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%202.png)

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%203.png)

( 이미지 출처 : [https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5#59a5](https://medium.com/@jshvarts/read-marble-diagrams-like-a-pro-3d72934d3ef5#59a5) )

- Stream에서 발생한 이벤트에는 onNext / onComplete / onError가 있었습니다.
- Marble Diagram에서는 수직 선분으로 Stream에서 발행이 완료 된 시점을 표기합니다.
- Marble Diagram에서는 X 모양으로 Stream에서 에러가 발생 한 시점을 표기합니다.
- 위에서 보이는 동그라미 / 오각형 / 삼각형 도형은 Stream에서 발행한 Item을 의미합니다.
- Stream 발행이 완료되었을 때 표기하는 선분 / 밑에서 나오는 Error가 났을 때를 의미하는 X표를 제외한 다른 이벤트는 Item의 형태일 가능성이 매우 높습니다.
- 세번째 그림은 완료를 의미하는 선분, 에러가 났음을 의미하는 X가 없는데, 이는 Stream이 종료되지 않았음을 의미합니다.

## 📖 Operator Marble Diagram Example

 위에서 이야기 했듯, Marble Diagram을 활용하면 Operator가 Stream을 어떻게 변형하는지를 한눈에 볼 수 있습니다. 

 일반적으로, operator에 대한 marble diagram은 크게 세부분으로 나눌 수 있습니다. 

 중간부분에 operator가 어떤 역할을 하는지 대략적인 그림 / 설명으로 나와 있고, 윗부분엔 변형되기 전 Stream, 아랫 부분엔 변형 된 이후의 Stream의 형태를 보여줍니다. 

 아래의 예시들을 보며 확인을 해 볼 예정인데, 아래에 적힌 Operator가 어떤 역할을 하는지 marble diagram을 통해 읽어 볼 뿐이지, 어떤 operator 인지는 지금 설명도 하지 않을 예정이고, 어떤 역할을 하는 operator 인지는  몰라도 전혀 문제되지 않습니다. 

### 💡 map

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%204.png)

- 먼저 윗부분을 살펴보면, Item 1 / 2 / 3 이 발행 되고, 3이 발행된 이후 데이터 발행이 끝났음을 알 수 있습니다.
- 중간 부분을 보면, x값이 들어오면 x에 10을 곱한 값으로 화살표가 되어있습니다.
- 아랫 부분을 보면, 노란색 원 안에 1이 들어있던 것이 10으로 변했고, 빨간색 원 안에 들어있던 값이 2에서 20으로 변했으며 파란색 원 안에 들어있던 3이 30으로 변한것을 확인 할 수 있습니다.
- 또 데이터 발행이 완료된 시점에 대한 변형은 없는 것을 확인 할 수 있습니다.
- 따라서 이 map 이라는 operator는 발행된 데이터의 값에 10을 곱하여 주는 역할을 진행 하고 있음을 알 수 있습니다.

### 💡 flatMap

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%205.png)

- 먼처 윗부분을 살펴보면, 빨간색 공 / 초록색 공 / 파란색 공모양의 아이템이 발행 되고, 데이터 발행이 종료되었음을 알 수 있습니다.
- 중간 부분을 살펴보면, 동그라미 공 모양이 화살표가 있는 실선으로 바뀌었고, 동그라미 하나가 네모 두개로 바뀐 것을 알 수 있습니다.
- 아랫 부분을 살펴보면, 빨간색 공 아이템이 빨간색 네모 두개로 변경되었고, 초록색 공 아이템이 초록색 네모 두개로 변경되었으며, 파란색 공 아이템이 파란색 네모 두개로 변경 되었음을 알 수 있습니다.
- 따라서 flatmap 이라는 operator는, 발행된 공모양 아이템을 네모 두개로 바꾸어 주는 역할을 진행하고 있음을 알 수 있습니다.

### 💡 debounce

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%206.png)

- 먼저 윗부분을 살펴보면, 1 ~ 6까지의 아이템이 발행되고, 데이터 발행이 종료되었음을 알 수 있습니다.
- 중간 부분을 살펴보면 아무런 힌트가 없습니다.
- 아랫 부분을 살펴보면, 첫번째 아이템인 1이 적힌 원이 어느 정도 시간이 흐른 이후 발행 된 것을 확인 할 수 있습니다.
- 또 두번째, 세번째, 네번째, 다섯번쨰 아이템이 꽤 가까운 시간 간격으로 아이템이 발행되었는데, 결국에는 다섯번째 아이템만 발행 된 것을 확인 할 수 있습니다.
- 또 여섯번째 아이템은 어느 정도의 시간이 지난 이후 아이템이 발행 된 것을 확인 할 수 있습니다.
- 따라서, debounce라는 operator는, 아이템 발행이 일정 시간 이후에 진행되도록 아이템 발행을 늦추는 역할을 하고있고, 또 가까운 시간간격으로 발행된 아이템이라면 마지막 아이템만 일정 시간 이후 아이템을 발행시켜주는 operator 라는 것을 알 수 있습니다.

### 💡 onErrorReturn

![Untitled](/assets/images/posts/2021-08-23-reactive-stream-marble-diagram/Untitled%207.png)

- 먼저 윗부분을 살펴보면, 첫번째 그림에선 동그라미 하나가 발행되고 timeline이 끊겨 있고, 두번째 그림에선 X모양이 나오고 ( 에러 이벤트가 발생하고 ) timeline이 끊겨 있음을 알 수 있습니다.
- 그림의 중간 부분에는 X 모양 ( 위에서 이야기했듯, X는 에러 이벤트를 의미합니다. )이 나오면, 빨간색 네모의 아이템이 나오는 것을 확인 할 수 있습니다.
- 아랫 부분을 살펴보면, 첫번째 그림에선 동그라미가 동그라미로 화살표가 이어 져 있고, 두번째 그림에선 에러 이벤트가 네모 아이템으로 화살표가 이어져 있습니다.
- 따라서 onErrorReturn operator는, 에러 이벤트를 특정 아이템으로 변환해주는 역할을 하고 있음을 알 수 있습니다.

