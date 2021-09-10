---
title: RxJava 강의 6 - Schedulers
date: 2021-09-10
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava2에서 비동기 프로그래밍을 위해 사용하고 있는 Scheduler, subscribeOn, observeOn 등에 대해서 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Scheduler

- RxJava에서 비동기 프로그래밍을 위한 쓰레드 관리자
- Scheduler설정을 통해, 이벤트를 어떤 쓰레드에서 발행 할 지 / 발행 한 이벤트를 어떤 쓰레드에서 발급 받을 지 설정할 수 있음
- 이벤트를 처리하는 스케줄러 지정을 위해 `observeOn` / `subscribeOn` 오퍼레이터를 활용

## 📖 HandlerScheduler ( AndroidSchedulers.mainThread() )

- Android에서 UI 작업을 할 때 사용하는 스케쥴러
- RxAndroid 라이브러리에 존재
- UI Thread에서 작업을 진행 할 수 있도록 해줌

## 📖 IO Scheduler ( Schedulers.io() )

- I / O 처리 작업을 할 때 사용하는 스케쥴러
- 네트워크 요청 처리, 각종 입 / 출력 작업, DB 쿼리 등에 사용 됨

## 📖 Computation Scheduler ( Schedulers.computation() )

- 논리적인 연산 처리 시 사용하는 스케쥴러
- 대기 시간 없이 빠르게 계산 작업을 수행하기 위해 사용 함

## 📖 NewThread Scheduler ( Schedulers.newThread() )

- 요청 할 때마다 매번 새로운 쓰레드를 생성
- 쓰레드풀에서 쓰레드를 가져오는 것이 아니라 매번 생성 하고, 이를 위한 추가적인 자원이 소모됨

## 📖 Trampoline Scheduler ( Schedulers.trampoline() )

- 현재 실행되고 있는 쓰레드에 Ready Queue를 생성하여, 처리할 작업들을 큐에 넣고 순서대로 처리
- 새로운 쓰레드를 생성하지 않고, 현재 실행되고 있는 쓰레드에 큐를 생성 한 이후에, 큐에 들어간 이벤트 순서대로 처리

## 📖 Single Scheduler ( Schedulers.single() )

- 단일 쓰레드 ( 현재 진행중인 쓰레드에서 새로 생성 )를 생성하여 처리 작업을 진행
- 다른 곳에서 구독 하더라도, single 스케쥴러를 사용 한다면 별도로 생성한 단일 쓰레드에서 작업을 진행

## 📖 Executor Scheduler ( Schedulers.from(executor) )

- Executor를 사용해서 생성한 쓰레드를 사용

## 📖 SubscribeOn

- 이벤트 발행을 할 때의 Scheduler를 지정 해 줌

## 📖 ObserveOn

- 발행 된 이벤트를 수신받고, 이벤트가 도착했을 때 Scheduler를 지정 해 줌

# 📚 Scheduler

## 📖 Scheduler

 개발을 하다보면, 병렬적인 작업 처리등을 위해 현재 작업을 진행하고 있는 Thread 이외에 다른 Thread에서 작업을 해야 하는 경우가 있습니다. 

 RxJava에서 Scheduler란 어떤 작업에 대한 쓰레드 관리자 입니다. 

 즉, RxJava에서는 스케쥴러를 통해 어떤 작업을 어떤 쓰레드에서 할지 정할 수 있습니다. 

 예를들어 Scheduler A에는 쓰레드에 대한 규칙으로 `작업이 들어 올 때 마다 새로운 쓰레드를 생성 하여 사용` 이라는 규칙이 있다고 가정 해 봅시다. 

 그리고 어떤 작업을 스케줄러 A에 할당 해 놓았다면, 그 작업은 스케쥴러 A의 규칙에 의해 작업이 진행되고, 따라서 그 작업은 새롭게 생성된 `NewThread1` 에서 작업이 진행 될 것입니다. 

 여기서 이해하고 넘어 갈 것은, 각 스케쥴러마다 작업을 진행하는 쓰레드 관리에 대한 규칙이 있다는 점 입니다. 

 이러한 스케쥴러를 RxJava에서는 `io` / `computation` 등으로 구현 해 놓았고, 이를 가져다 쓸 수 있습니다. 

 따라서 이번 포스팅에서는 구현된 스케쥴러의 종류는 어떤 것이 있는지, 또 스케쥴러를 작업에 어떻게 할당 할 지에 대해 알아봅니다. 

# 📚 스케쥴러의 종류 및 특성

## 📖 HandlerScheduler ( AndroidSchedulers.mainThread() )

- 작업을 UI Thread에서 진행 하려고 할 때 사용하는 스케쥴러 입니다.
- RxAndroid 라이브러리 안에 있는 스케쥴러입니다.
- Android에서는 main scheduler로 로 이 스케쥴러를 사용하고 있습니다.
- 백그라운드 쓰레드에서 작업 이후, UI 변경을 할 때 주로 사용합니다.

## 📖 IO Scheduler ( Schedulers.io() )

- 네트워크 통신 / 로컬 DB 쿼리 처리 등과 같은 Input / Output 처리 작업을 할 때 사용하는 스케쥴러 입니다.
- 쓰레드 풀 ( CachedThreadPool )에서 쓰레드를 가져오거나 가져올 쓰레드가 없으면 새로운 쓰레드를 생성 합니다.
- 쓰레드를 생성하는데 갯수 제한이 없으며, 만약 너무 많은 쓰레드가 생성되면 Out of Memory Error가 발생 할 수 있습니다.

## 📖 Computation Scheduler ( Schedulers.computation() )

- 논리적인 연산 처리 시 사용하는 스케쥴러 입니다.
- CPU 코어의 물리적 쓰레드 수 ( [availableProcessor](https://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html#availableProcessors()) )와 동일한 갯수의 쓰레드가 생성되어 있고, 생성된 쓰레드에서 연산 작업을 진행합니다.
- CPU 코어의 물리적 쓰레드 수를 넘는 연산 작업을 병렬적으로 실행 한 경우, 성능 이슈가 있을 수 있습니다.
- 대기 시간 없이 빠르게 계산 작업의 결과값을 받기 위한 경우에 사용 합니다.

## 📖 NewThread Scheduler ( Schedulers.newThread() )

- 요청 할 때마다 매번 새로운 쓰레드를 생성하는 스케쥴러 입니다.
- 즉, 쓰레드풀에서 이미 만들어 져 있는 쓰레드를 쓰레드풀에서 가져오는 것이 아니라 매번 생성 합니다.
- 매번 생성되면, 생성하는데 비용도 많이 들고 생성된 쓰레드가 재사용도 되지 않기 때문에 많이 사용되지 않습니다.

## 📖 Trampoline Scheduler ( Schedulers.trampoline() )

- 현재 실행되고 있는 쓰레드에 Ready Queue를 생성하여, 처리할 작업들을 큐에 넣고 순서대로 처리하는 스케쥴러 입니다.
- 새로운 쓰레드를 생성하지 않고, 현재 실행되고 있는 쓰레드에 큐를 생성 한 이후에, 큐에 들어간 이벤트 순서대로 처리합니다.

## 📖 Single Scheduler ( Schedulers.single() )

- 단일 쓰레드 ( 현재 진행중인 쓰레드에서 새로 생성 )를 생성하여 처리 작업을 진행하는 스케쥴러 입니다.
- 다른 위치에서 해당 스케쥴러를 활용 할 경우에도 모든 곳에서 이미 생성된 단일 쓰레드를 사용합니다.

## 📖 Scheduler ( Schedulers.from(executor) )

- Executor를 사용해서 생성한 쓰레드를 사용하는 스케쥴러 입니다.
- 작업에 이미 만들어진 Executor를 사용하여야 하는 경우에 사용 할 수 있습니다.

# 📚 SubscribeOn / ObserveOn

 RxJava에서는 작업을 어떤 스케쥴러에서 진행 할 것인지를 정하기 위해 `subscribeOn` / `observeOn` 오퍼레이터를 사용하고 있습니다. 

 두 오퍼레이터의 차이점과 특성에 대해서 간단하게 알아보겠습니다. 

## 📖 SubscribeOn

- Marble Diagram

    ![Untitled](/assets/images/posts/2021-09-10-schedulers/Untitled.png)

    ( 이미지 출처 : [http://reactivex.io/documentation/operators/subscribeon.html](http://reactivex.io/documentation/operators/subscribeon.html) )

- 이벤트를 발행하는 곳, 발행된 이벤트를 구독하는 곳에서 어떤 Scheduler를 사용 할 지 정할 때 사용하는 operator 입니다.
- observeOn 오퍼레이터를 사용하지 않을 시, 이벤트를 구독하는 곳에서 진행되는 작업도 같은 Scheduler에서 진행 됨을 유의 해 주세요.
- subscribeOn 오퍼레이터는, 여러번 사용하더라도 스트림에서 한번만 적용됩니다.

    ```kotlin
    private fun threadLog(message: String) {
        Log.d("scheduler : ", "thread name = ${Thread.currentThread().name} / message = $message")
    }
    ```

    ```kotlin
    Observable
        .just("1", "2", "3")
        .subscribeOn(Schedulers.io())
        .subscribeOn(Schedulers.computation())
        .subscribe({
            threadLog(it)
        }, { it.printStackTrace() })

    // 결과값 
    thread name = RxCachedThreadScheduler-1 / message = 1
    thread name = RxCachedThreadScheduler-1 / message = 2
    thread name = RxCachedThreadScheduler-1 / message = 3
    ```

    - 위를 살펴보면, scheduler를 io / computation으로 정해 주었지만 실제 데이터 발행이 진행 된 thread는 io scheduler의 thread 인 것을 확인 할 수 있습니다.

## 📖 ObserveOn

- Marble Diagram

    ![Untitled](/assets/images/posts/2021-09-10-schedulers/Untitled%201.png)

    ( 이미지 출처 : [http://reactivex.io/documentation/operators/subscribeon.html](http://reactivex.io/documentation/operators/subscribeon.html) )

- 발행된 이벤트를 구독하는 곳에서 어떤 Scheduler를 사용 할 지 정할 때 사용되는 operator 입니다.
- observeOn 오퍼레이터로 Scheduler를 바꾸어 준 이후 부터 이벤트를 처리하는 Scheduler가 변경 됩니다.

    ```kotlin
    private fun threadLog(message: String) {
        Log.d("scheduler : ", "thread name = ${Thread.currentThread().name} / message = $message")
    }
    ```

    ```kotlin
    Observable
        .just("1", "2", "3")
        .doOnNext { threadLog(it) }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe({
            threadLog("it in subscribe $it")
        }, { it.printStackTrace() })

    thread name = RxCachedThreadScheduler-1 / message = 1
    thread name = RxCachedThreadScheduler-1 / message = 2
    thread name = RxCachedThreadScheduler-1 / message = 3
    thread name = main / message = it in subscribe 1
    thread name = main / message = it in subscribe 2
    thread name = main / message = it in subscribe 3
    ```

- subscribeOn은 여러번 사용하더라도 한번만 적용 되지만, observeOn은 여러번 사용했을 때 사용 한 만큼 Scheduler가 변경 되고 있음에 유의 해 주세요.

    ```kotlin
    private fun threadLog(message: String) {
        Log.d("scheduler : ", "thread name = ${Thread.currentThread().name} / message = $message")
    }
    ```

    ```kotlin
    val observable = Observable.create<String> { emitter ->
        emitter.onNext("1")
        emitter.onNext("2")
        emitter.onNext("3")
        emitter.onNext("4")
    }

    compositeDisposable.add(
        observable
            .subscribeOn(Schedulers.io())
            .doOnNext { threadLog(it) }
            .observeOn(Schedulers.computation())
            .doOnNext { threadLog(it) }
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe({
                threadLog("it in subscribe $it")
            }, { it.printStackTrace() })
    )

    thread name = RxCachedThreadScheduler-1 / message = 1
    thread name = RxCachedThreadScheduler-1 / message = 2
    thread name = RxCachedThreadScheduler-1 / message = 3
    thread name = RxCachedThreadScheduler-1 / message = 4
    thread name = RxComputationThreadPool-1 / message = 1
    thread name = RxComputationThreadPool-1 / message = 2
    thread name = RxComputationThreadPool-1 / message = 3
    thread name = RxComputationThreadPool-1 / message = 4
    thread name = main / message = it in subscribe 1
    thread name = main / message = it in subscribe 2
    thread name = main / message = it in subscribe 3
    thread name = main / message = it in subscribe 4
    ```