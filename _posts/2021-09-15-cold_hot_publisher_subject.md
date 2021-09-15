---
title: RxJava 강의 7 - Cold, Hot Observable / Subject ( Processor )
date: 2021-09-15
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

Reactive Stream의 Cold / Hot Publisher ( Cold Observable / Hot Observable )에 대해 알아보고, Subject ( Processor )가 무엇인지와 RxJava2에서 Subject의 구현체인 PublishSubject / BehaviorSubject / AsyncSubject / ReplaySubject에 대해 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📚 Cold Publisher / Hot Publisher

### 📖 Cold Publisher

- 구독자가 구독하는 시점에 데이터 방출을 시작하는 Publisher
- 구독자가 구독하는 시점에 데이터 발행을 시작하기 때문에, 모든 구독자는 동일한 데이터를 수신 받을 수 있음
- 앞서 살펴본 Single / Flowable / Observable 등이 이에 해당 됨

### 📖 Hot Publisher

- 구독자가 구독하지 않아도, 데이터를 방출하는 Publisher
- 구독자가 구독하지 않아도 데이터를 방출하기 때문에, 구독자가 구독하기 이전에 발행된 데이터는 구독자가 데이터를 받을 수 없음
- Processor / Subject 등이 이에 해당 됨
- 중복된 이벤트 발행을 막기 위해 사용 할 수 있음 ( MultiCasting / UniCasting, 추후 포스팅에서 Connectable Observable과 함께 다룰 예정 )

## 📚 Processor / Subject

### 📖 Processor / Subject

- Reactive Stream에서, Publihser / Subscriber를 동시에 구현하고 있는 interface
- 즉 관찰 대상으로도, 관찰자로써도 활용할 수 있도록 활용
- 앞서 살펴본 Observable / Flowable / Single 등과 다르게 Hot Publisher의 특성을 지님
- Flowable / Observable의 차이점과 마찬가지로, Processor는 배압을 활용할 수 있지만, Subject는 배압을 활용하지 못함
- RxJava2에는 Subject / Processor를 PublishSubject ( Processor ) / BehaviorSubject ( Processor ) / AsyncSubject ( Processor ) / ReplaySubject ( Processor ) 등으로 구현 해 둠

### 📖 PublishSubject

- 구독 시점 이전에 발행된 데이터는 무시하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject

### 📖 BehaviorSubject

- 구독 시점 이전에 발행된 데이터 중 가장 최근에 발행된 데이터 하나만 발행하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject
- `createDefault` 메소드를 활용하여, 이전에 발행된 데이터가 없을 시 기본적으로 방출 할 데이터를 정할 수 있음

### 📖 AsyncSubject

- 데이터 발행이 완료되면, 데이터 발행 완료 직전에 발행된 데이터만 발행하는 Subject

### 📖 ReplaySubject

- 구독 시점 이전에 발행된 데이터 모두를 발행하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject
- `createWithSize(int maxSize)` 메소드를 활용하여, 구독 시점 이전에 발행된 데이터 중 구독 시점에서 부터 가까운 순서대로 몇개를 발행 할 것인지를 정할 수 있음

# 📚 Cold Publisher / Hot Publisher

## 📖 Cold Publisher vs Hot Publisher

 Reactive Streams의 Publisher 즉, 구독 대상은 크게 두가지로 나눌 수 있습니다. 

 바로 Hot Publisher / Cold Publisher 인데요, 기준은 구독자 ( Subscriber )가 구독대상 ( Publisher )을 구독 ( Subscribe )하는 시점에 관계 없이 발행된 데이터를 처음부터 모두 발행 해 줄 것이냐 / 그렇지 않을 것이냐 입니다. 

 이를 살펴보기위해 Cold Publisher / Hot Publisher에 대해 알아보겠습니다. 

## 📖 Cold Publisher

- Cold Publisher는 구독자 ( 소비자 )가 구독대상 ( 생산자 )을 구독 할 때마다 데이터를 처음부터 새로 통지 합니다.

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled.png)

    ( 출처 : [https://jade314.tistory.com/entry/리엑티브-생산자Publisher-Cold-Publisher-Hot-Publisher](https://jade314.tistory.com/entry/%EB%A6%AC%EC%97%91%ED%8B%B0%EB%B8%8C-%EC%83%9D%EC%82%B0%EC%9E%90Publisher-Cold-Publisher-Hot-Publisher) )

- 구독자가 구독 할 때 마다 데이터를 통지하는 새로운 타임라인이 생성 됩니다.
- 구독자는 구독 시점과 상관 없이 구독 대상에서 통지된 데이터를 처음부터 전달 받을 수 있습니다.
- 구독 시점과 관계 없이 결과를 받아야 하는 Network 통신 / DB 쿼리 등에 알맞습니다.
- Observable / Flowable / Single 등 [앞선 포스팅](https://kangraemin.github.io/android/2021/08/31/stream-implementation/)에서 알아본 Reactive Stream의 구현체들이 모두 Cold Publisher의 성질을 띄고 있습니다.

## 📖 Hot Publisher

- Hot Publisher는 구독자 ( 소비자 )수와 상관 없이 데이터를 한번만 통지합니다.

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled%201.png)

    ( 출처 : [https://jade314.tistory.com/entry/리엑티브-생산자Publisher-Cold-Publisher-Hot-Publisher](https://jade314.tistory.com/entry/%EB%A6%AC%EC%97%91%ED%8B%B0%EB%B8%8C-%EC%83%9D%EC%82%B0%EC%9E%90Publisher-Cold-Publisher-Hot-Publisher) )

- 구독자 ( 소비자 )의 수에 관계 없이 데이터를 통지하는 타임라인은 하나입니다.
- 구독자 ( 소비자 )는 발행된 데이터를 처음부터 전달 받는 것이 아니라, 구독한 시점 이후에 통지된 데이터들만 전달 받을 수 있습니다.
- 구독 시점 이전의 데이터는 중요하지 않은 GUI 이벤트에 알맞게 활용 할 수 있습니다.
- 아래에서 알아볼 Processor / Subject가 Hot Publisher의 성질을 띄고 있습니다.

# 📚 Processor / Subject

## 📖 Processor

 Reactive Stream을 살펴보면 이벤트를 발행하는 역할을 하는 Publisher, 발행된 데이터를 구독하는 구독자 역할을 하는 Subscriber가 존재하고 Publisher / Subscriber의 역할을 모두 수행하는 Processor를 확인 할 수 있습니다. ( [link](https://github.com/reactive-streams/reactive-streams-jvm#4processor-code) ) 

```kotlin
public interface Processor<T, R> extends Subscriber<T>, Publisher<R>
```

 따라서 Processor를 구현하는 클래스는, 데이터를 발행 할 수도, 데이터를 구독 할 수도 있어야 하며, 배압에 관련된 기능도 제공 해아 합니다. 

 또한 Processor는 Hot Publisher의 특징을 갖고 있어, 구독자가 구독을 하는 시점에 따라 구독자에게 발행되는 데이터가 다를 수 있습니다. 

## 📖 Subject

 RxJava2에서 나오는 Subject란, Processor와 기능적으로 동일하게 동작하지만, 배압에 관련된 기능이 제거된 인터페이스 입니다. 

```kotlin
public abstract class Subject<T> extends Observable<T> implements Observer<T>
```

 따라서 Subject도 Hot Publisher의 특징을 갖고 있으며, Observable과 동일하게 사용 할 수도 있습니다. 

 RxJava2에는, Processor와 Subject의 구현체로써 여러가지를 두었는데, 이번 포스팅에서는 Subject 위주로 알아 볼 것이며, `PublishSubject` / `BehaviorSubject` / `AsyncSubject` / `ReplaySubject` 에 대해서 알아 보겠습니다. 

# 📚 Subject 구현체들

- 아래에 적힌 코드는 데이터 발행에 공통적으로 쓰인 코드입니다.

    ```kotlin
    private fun startTaskToGetFirstString(): String = "1".also { threadLog("Start task to emit $it") }
    private fun startTaskToGetSecondString(): String = "2".also { threadLog("Start task to emit $it") }
    private fun startTaskToGetThirdString(): String = "3".also { threadLog("Start task to emit $it") }
    ```

## 📖 PublishSubject

- 구독 시점 이전에 발행된 데이터는 무시하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject 입니다.

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled%202.png)

    ( 출처 : [https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/PublishSubject.png) )

- 구독자가 구독 대상을 구독 하기 전의 이벤트가 중요하지 않을 때 주로 사용합니다.
- 예시코드

    ```kotlin
    val publishSubject = PublishSubject.create<String>()

    publishSubject.onNext(startTaskToGetFirstString())
    publishSubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        publishSubject
            .subscribe({
                threadLog("$it in subscribe")
            }, { it.printStackTrace() })
    )
    publishSubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        publishSubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = Start task to emit 3
    thread name = main / message = 3 in first subscribe
    thread name = main / message = --------구분선--------
    ```

    - 첫번째 구독자가 구독하기 이전에 발행된 1, 2는 구독자에게 도달하지 못했고, 구독한 이후에 발행된 3만 첫번쨰 구독자에게 도달하였음을 확인 할 수 있습니다.
    - 두번째 구독자는 구독 이후 아무런 데이터가 발행되지 않았기 때문에, 어떤한 데이터도 두번쨰 구독자에게 도달하지 않았음을 확인 할 수 있습니다.

## 📖 BehaviorSubject

- 구독 시점 이전에 발행된 데이터 중 가장 최근에 발행된 데이터 하나만 발행하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject 입니다.

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled%203.png)

    ( 출처 : [https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.png](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/S.BehaviorSubject.png) )

- `createDefault` 메소드를 활용하여, 이전에 발행된 데이터가 없을 시 기본적으로 방출 할 데이터를 정할 수 있습니다.
- 예시 코드

    ```kotlin
    val behaviorSubject = BehaviorSubject.create<String>()

    behaviorSubject.onNext(startTaskToGetFirstString())
    behaviorSubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        behaviorSubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    behaviorSubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        behaviorSubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = 2 in first subscribe
    thread name = main / message = Start task to emit 3
    thread name = main / message = 3 in first subscribe
    thread name = main / message = --------구분선--------
    thread name = main / message = 3 in second subscribe
    ```

    - 첫번째 구독자가 구독하기 이전에, 첫번째에 발행한 1은 무시되고 2부터 구독자에게 도달함을 확인 할 수 있습니다.
    - 두번째 구독자가 구독하기 이전에, 구독 직전에 발행한 3만 구독자에게 도달하고 나머지 데이터는 무시되었음을 확인할 수 있습니다.

    ```kotlin
    val behaviorSubject = BehaviorSubject.createDefault("default")

    compositeDisposable.add(
        behaviorSubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    behaviorSubject.onNext(startTaskToGetFirstString())
    behaviorSubject.onNext(startTaskToGetSecondString())
    behaviorSubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        behaviorSubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = default in first subscribe
    thread name = main / message = Start task to emit 1
    thread name = main / message = 1 in first subscribe
    thread name = main / message = Start task to emit 2
    thread name = main / message = 2 in first subscribe
    thread name = main / message = Start task to emit 3
    thread name = main / message = 3 in first subscribe
    thread name = main / message = --------구분선--------
    thread name = main / message = 3 in second subscribe
    ```

    - `createDefault` 메소드를 활용하여, 첫번째 구독자가 구독하기 이전 아무런 데이터가 발행이 되지 않았을 때 기본 값을 설정 할 수 있음을 확인 할 수 있습니다.

## 📖 AsyncSubject

- 데이터 발행이 완료되면, 데이터 발행 완료 직전에 발행된 데이터만 발행하는 Subject

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled%204.png)

    ( 출처 : [https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/AsyncSubject.png) )

- 이전의 데이터는 중요하지 않고, 발행 완료 시점 직전의 데이터만 중요한 경우에 주로 사용합니다.
- 예시 코드

    ```kotlin
    val asyncSubject = AsyncSubject.create<String>()

    asyncSubject.onNext(startTaskToGetFirstString())
    asyncSubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        asyncSubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    asyncSubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        asyncSubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    asyncSubject.onComplete()
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = Start task to emit 3
    thread name = main / message = --------구분선--------
    thread name = main / message = 3 in first subscribe
    thread name = main / message = 3 in second subscribe
    ```

    - 첫번째 구독자와 / 두번쨰 구독자 모두 데이터 발행이 완료 된 시점에 ( `onComplete()` ) 제일 마지막에 발행된 데이터만 구독자에게 도달하였음을 확인 할 수 있습니다.

    ```kotlin
    val asyncSubject = AsyncSubject.create<String>()

    asyncSubject.onNext(startTaskToGetFirstString())
    asyncSubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        asyncSubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    asyncSubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        asyncSubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = Start task to emit 3
    thread name = main / message = --------구분선--------
    ```

    - 데이터 발행이 완료되지 않았을 때엔, 어떠한 데이터도 구독자에게 도달하지 못했음을 확인 할 수 있습니다.

## 📖 ReplaySubject

- 구독 시점 이전에 발행된 데이터 모두를 발행하고, 구독 시점 이후에 발행된 데이터를 발행하는 Subject 입니다.

    ![Untitled](/assets/images/posts/2021-09-15-cold_hot_publisher_subject/Untitled%205.png)

    ( 출처 : [https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/ReplaySubject.u.png) )

- `createWithSize(int maxSize)` 메소드를 활용하여, 구독 시점 이전에 발행된 데이터 중 구독 시점에서 부터 가까운 순서대로 몇개를 발행 할 것인지를 정할 수 있습니다.
- 예시코드

    ```kotlin
    val replaySubject = ReplaySubject.create<String>()

    replaySubject.onNext(startTaskToGetFirstString())
    replaySubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        replaySubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    replaySubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        replaySubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = 1 in first subscribe
    thread name = main / message = 2 in first subscribe
    thread name = main / message = Start task to emit 3
    thread name = main / message = 3 in first subscribe
    thread name = main / message = --------구분선--------
    thread name = main / message = 1 in second subscribe
    thread name = main / message = 2 in second subscribe
    thread name = main / message = 3 in second subscribe
    ```

    - 첫번째, 두번째 구독자가 구독하기 이전의 데이터도 모두 각 구독자에게 도달하였음을 확인 할 수 있습니다.

    ```kotlin
    val replaySubject = ReplaySubject.createWithSize<String>(2)

    replaySubject.onNext(startTaskToGetFirstString())
    replaySubject.onNext(startTaskToGetSecondString())
    compositeDisposable.add(
        replaySubject
            .subscribe({
                threadLog("$it in first subscribe")
            }, { it.printStackTrace() })
    )
    replaySubject.onNext(startTaskToGetThirdString())

    threadLog("--------구분선--------")

    compositeDisposable.add(
        replaySubject
            .subscribe({
                threadLog("$it in second subscribe")
            }, { it.printStackTrace() })
    )
    ```

    ```kotlin
    thread name = main / message = Start task to emit 1
    thread name = main / message = Start task to emit 2
    thread name = main / message = 1 in first subscribe
    thread name = main / message = 2 in first subscribe
    thread name = main / message = Start task to emit 3
    thread name = main / message = 3 in first subscribe
    thread name = main / message = --------구분선--------
    thread name = main / message = 2 in second subscribe
    thread name = main / message = 3 in second subscribe
    ```

    - `createWithSize` 를 활용하여, 담아놓는 이벤트의 최대 갯수를 정해 주었습니다.
    - 따라서 두번째 구독자에게는 최근의 2개의 이벤트 ( 2, 3 )만 도달하였음을 확인 할 수 있습니다.