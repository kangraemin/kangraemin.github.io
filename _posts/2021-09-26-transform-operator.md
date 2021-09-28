---
title: RxJava 강의 9 - 흐름 제어 연산자 ( take / filter / map / flatMap / switchMap ... )
date: 2021-09-26
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava에서 방출된 데이터들을 변형하고, 발행된 데이터를 취사선택하는 Operator들( take / filter / map / flatMap / switchMap ... )의 특징과 예시 코드들을 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Take

- 발행된 데이터가 특정 조건을 만족 할 때 까지만 데이터를 발행 받을 수 있도록 하는 operator
- `take` / `takeLast` / `takeUntil` / `takeWhile` 등의 operator가 존재
- take 조건이 완료 된 이후, 데이터를 더이상 방출 하지 않을 시점에 `onComplete`를 호출하는 것을 확인 할 수 있음
- `Interval` operator와 함께 사용하여 timer를 제작 할 수 있음

## 📖 Skip

- 특정 조건 까지 발행된 데이터를 발행하지 않도록 하는 operator
- `skip` / `skipLast` / `skipWhile` 등의 operator가 존재

## 📖 Filter

- Boolean을 리턴하는 조건식을 인자로 받으며, 조건식이 true인 경우만 데이터를 방출하도록 하는 operator

## 📖 Map

- 발행된 데이터를 변형해주는 operator

## 📖 FlatMap

- 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator
- Stream을 다른 Stream으로 변형 할 때 사용
- map은 발행된 데이터를 다른 데이터로 변형 해주는 operator 라면, flatMap은 발행된 데이터를 다른 Stream에서 발행하는 데이터로 변형 해주는 operator

## 📖 SwitchMap

- flatMap 처럼 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator
- Stream을 다른 Stream으로 변형 할 때 사용 되며, 이전의 stream을 취소시키는 기능을 갖고 있음

## 📖 ConcatMap

- flatMap 처럼 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator
- Stream을 다른 Stream으로 변형 할 때 사용 되며, 작업의 순서를 보장해주는 기능을 갖고 있음

# 📚 흐름 제어 연산자

## 📖 개요

 방출된 데이터를 취사선택 / 데이터 변형 / 방출된 데이터를 스트림으로 변형하는 Operator들을 알아보고자 합니다. 

 예시 코드를 통해 각 operator의 특징들을 살펴 볼 예정이며 아래에 쓰인 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/blob/ad95d47ba338e5902bc081d092fe1ec72530de6c/AndroidProject/app/src/main/java/com/example/rxjavalecture/operator/transform/TransformOperatorExampleActivity.kt)에서 확인 하실 수 있습니다. 

## 📖 Take

### 개요

- 발행된 데이터가 특정 조건을 만족 할 때 까지만 데이터를 발행 받을 수 있도록 하는 operator 입니다.
- `take` / `takeLast` / `takeUntil` / `takeWhile` 등의 operator가 존재합니다.
- take 조건이 완료 된 이후, 데이터를 더이상 방출 하지 않을 시점에 `onComplete`를 호출하는 것을 확인 할 수 있습니다.

### Take

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled.png)
    
- 특징
    - 발행된 데이터 중 처음 `Count`개 만 발행합니다.
    - 발행된 데이터가 `Count`개를 넘어서면, 더이상 발행되지 않습니다.
    - 위 Marble Diagram에서, 2개를 방출하고 `onComplete`이벤트를 를 방출하는 것을 확인 할 수 있습니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runTakeExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .take(3)
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 0
    message = 1
    message = 2
    ```
    
    - 0 ~ 9까지 데이터를 모두 발행하였으나, 3개만 받도록 되어 있으므로 0, 1, 2만 발행 된것을 확인 할 수 있습니다.

### TakeLast

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%201.png)
    
- 특징
    - 발행된 데이터 중 마지막의 `Count`개 만 발행합니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runTakeExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .takeLast(3)
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과 
    message = 7
    message = 8
    message = 9
    ```
    
    - 0 ~ 9까지 데이터를 모두 발행하였으나, 마지막에 발행된 3개만 받도록 되어 있으므로 7, 8, 9만 발행 된것을 확인 할 수 있습니다.

### TakeUntil

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%202.png)
    
- 특징
    - Boolean을 리턴하는 조건식을 인자로 받습니다.
    - 조건식이 최초의 true를 리턴 할 때 까지 데이터를 방출합니다.
    - 최초의 true를 리턴한 케이스도 포함되어 방출되며, true를 방출한 이후 발행된 데이터는 방출되지 않습니다.
    - 최초의 true를 리턴한 데이터를 방출한 직후, `onComplete` 이벤트를 방출합니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runTakeExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .takeUntil { it > 3 }
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 0
    message = 1
    message = 2
    message = 3
    message = 4
    ```
    
    - 0, 1, 2, 3 모두 false를 리턴하여 데이터가 방출 되었습니다.
    - 4는 최초로 true를 방출하여 데이터가 방출되었습니다.
    - 5부터는 최초 true가 방출 된 이후기 때문에 방출 되지 않은것을 확인 할 수 있습니다.

### TakeWhile

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%203.png)
    
- 특징
    - Boolean을 리턴하는 조건식을 인자로 받습니다.
    - 조건식이 최초의 false를 리턴 할 때 까지 데이터를 방출합니다.
    - 최초의 false를 리턴한 케이스는 방출되지 않으며, false가 발생한 순간 `onComplete` 이벤트를 방출합니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runTakeExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .takeWhile { it < 3 }
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과 
    message = 0
    message = 1
    message = 2
    ```
    
    - 0, 1, 2 모두 조건식의 결과값이 true 이기 때문에 데이터가 방출되었습니다.
    - 3부터는 조건식이 false 이기 때문에 방출되지 않은 것을 확인 할 수 있습니다.

### 💡 Take를 활용한 Timer 만들기

- Interval을 활용하여 특정 주기마다 이벤트를 받을 수 있고, takeWhile 을 활용하여 interval의 데이터 발행 완료 시점을 제어 할 수 있습니다.
- 이 특성 두개를 조합하여 5초동안 1초마다 데이터를 방출하고, 5초가 지난 이후엔 `onComplete` 방출하는 Timer를 만들 수 있습니다.
    
    ![ezgif-7-ee4becb94152.gif](/assets/images/posts/2021-09-26-transform-operator/ezgif-7-ee4becb94152.gif)
    
- 예시 코드
    
    ```kotlin
    private val totalTime = 5
    private var currentRemainTime = totalTime
    
    private fun runTimerUsingTakeExample() {
        compositeDisposable
            .add(
                Observable
                    .interval(1000, TimeUnit.MILLISECONDS)
                    .doOnSubscribe { currentRemainTime = totalTime } // 구독시 ( timer 실행 시 ) 남은 시간 초기화 
                    .takeWhile { currentRemainTime > 0 } // 남은 시간이 0초 초과인 경우만 데이터 발행 
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe({
                        // Interval에서 발행된 데이터가 이곳으로 발행 됨 
                        simpleLog("emitted value = $it")
                        binding.btnTakeTimer.text = currentRemainTime.toString()
                        currentRemainTime--
                    }, { it.printStackTrace() }, {
    										// onComplete 이벤트가 발행 되었을 때 할 행동 정의
                        simpleLog("Timer is Done !")
                        binding.btnTakeTimer.text = "Timer using take Example"
                    })
            )
    }
    
    // 실행 결과
    message = emitted value = 0
    message = emitted value = 1
    message = emitted value = 2
    message = emitted value = 3
    message = emitted value = 4
    message = Timer is Done !
    ```
    
    - interval에서 데이터를 발행한 것을 확인 할 수 있으며 ( 0, 1, 2, 3, 4 ), totalTime인 5 이후부터는 발행이 되지 않았음을 확인 할 수 있습니다.
    - interval에서 5가 발행 되었을 때 onComplete 이벤트가 발행되어 Timer is Done ! 로그가 발생 한 것을 확인 할 수 있습니다.

## 📖 Skip

### 개요

- 특정 조건 까지 발행된 데이터를 발행하지 않도록 하는 operator 입니다.
- `skip` / `skipLast` / `skipWhile` 등의 operator가 존재합니다.

### Skip

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%204.png)
    
- 특징
    - skip할 데이터 갯수를 뜻하는 `Count` 를 인자로 받으며, Count 개수가 발행되기 전까지 데이터를 발행하지 않도록 하는 operator 입니다.
- 예시 코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runSkipExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .skip(3)
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 3
    message = 4
    message = 5
    message = 6
    message = 7
    message = 8
    message = 9
    ```
    
    - 처음에 발행된 3개의 데이터 ( 0, 1, 2 )가 발행되지 않았음을 확인 할 수 있습니다.

### SkipLast

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%205.png)
    
- 특징
    - skip할 데이터 갯수를 뜻하는 `Count` 를 인자로 받으며, 마지막에 발행된 데이터로부터 Count 개수 만큼 데이터를 발행하지 않도록 하는 operator 입니다.
- 예시 코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runSkipExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .skipLast(3)
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 0
    message = 1
    message = 2
    message = 3
    message = 4
    message = 5
    message = 6
    ```
    
    - 마지막 3개의 데이터 ( 7, 8, 9 )가 발행되지 않았음을 확인 할 수 있습니다.

### SkipWhile

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%206.png)
    
- 특징
    - Boolean을 리턴하는 조건식을 인자로 받으며, 조건식이 최초로 false가 날 때 까지 데이터를 skip 하도록 하는 operator 입니다.
- 예시 코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runSkipExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .skipWhile { it < 3 }
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 3
    message = 4
    message = 5
    message = 6
    message = 7
    message = 8
    message = 9
    ```
    
    - 조건식을 true로 만드는 데이터인 0, 1, 2가 발행되지 않았음을 확인 할 수 있습니다.

## 📖 Filter

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%207.png)
    
- 특징
    - Boolean을 리턴하는 조건식을 인자로 받으며, 조건식이 true인 경우만 데이터를 방출하도록 하는 operator 입니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runFilterExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .filter { it > 3 }
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = 4
    message = 5
    message = 6
    message = 7
    message = 8
    message = 9
    ```
    
    - 3보다 작은 수인 0, 1, 2는 데이터 방출이 이루어 지지 않았음을 확인 할 수 있습니다.

## 📖 Map

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%208.png)
    
- 특징
    - 발행된 데이터를 변형해주는 operator 입니다.
    - 위 마블 다이어그램에서 동그라미를 네모로 바꾸어 주듯, 발행된 데이터를 변형하는데 자주 사용되는 operator 입니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun runMapExample() {
        compositeDisposable
            .add(
                fromIterableSource
                    .map { "emitted integer -> $it" }
                    .subscribe(::simpleLog)
            )
    }
    
    // 실행 결과
    message = emitted integer -> 0
    message = emitted integer -> 1
    message = emitted integer -> 2
    message = emitted integer -> 3
    message = emitted integer -> 4
    message = emitted integer -> 5
    message = emitted integer -> 6
    message = emitted integer -> 7
    message = emitted integer -> 8
    message = emitted integer -> 9
    message = emitted integer -> 1
    message = emitted integer -> 2
    ```
    
    - list에서 방출된 `0, 1, 2 ...` 가 `emitted ineger → number` 형태로 변형 되어 방출 된 것을 확인 할 수 있습니다.

## 📖 FlatMap

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%209.png)
    
    - flatMap 안에 적힌 것을 보면 동그라미 데이터가 발행 되면, 마름모 데이터를 두개 발행하는 Stream으로 변형해 주는 것을 확인 할 수 있습니다.
    - 빨간색 동그라미가 발행 되고, 빨간색 마름모 두개가 발행 된 것을 확인 할 수 있습니다.
    - 초록색, 파란색도 마찬가지로 초록색, 파란색 마름모 두개가 각각 발행 된 것을 확인 할 수 있습니다.
- 특징
    - 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator 입니다.
    - Stream을 다른 Stream으로 변형 할 때 사용됩니다.
    - map은 발행된 데이터를 다른 데이터로 변형 해주는 operator 라면, flatMap은 발행된 데이터를 다른 Stream에서 발행하는 데이터로 변형 해주는 operator 입니다.
    - observable에 있는 flatMap operator는 observableSource을 인자로 받고, Single에 있는 flatMap operator는 SingleSource를 인자로 받습니다.
    - observable stream에서 flatMap을 통해 single stream에서 발행되는 데이터를 받고자 하는 경우 `flatMapSingle` operator를, maybe stream에서 발행되는 데이터를 받고자 하는 경우에는 `flatMapMaybe` operator를 활용하면 됩니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerObservable(studentNumber: Int): Observable<String> =
        Observable
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnFlatmap
                .clicks() // 유저가 btnFlatmap 버튼을 클릭 하면, Unit 객체를 아래로 발행 해 주는 operator 입니다.
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .flatMap { getNameFromServerObservable(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    
    // 실행 결과
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 1021 / message = Get name ( Rams num = 0 ) from server
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 1021 / message = Get name ( Rams num = 1 ) from server
    thread name = RxCachedThreadScheduler-4 실행 후 흐른 시간 = 1022 / message = Get name ( Rams num = 3 ) from server
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 1023 / message = Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 1023 / message = data received ! Rams num = 0
    thread name = RxCachedThreadScheduler-5 실행 후 흐른 시간 = 1023 / message = Get name ( Rams num = 4 ) from server
    thread name = RxCachedThreadScheduler-6 실행 후 흐른 시간 = 1024 / message = Get name ( Rams num = 5 ) from server
    thread name = main 실행 후 흐른 시간 = 1024 / message = data received ! Rams num = 1
    thread name = main 실행 후 흐른 시간 = 1024 / message = data received ! Rams num = 3
    thread name = main 실행 후 흐른 시간 = 1025 / message = data received ! Rams num = 2
    thread name = main 실행 후 흐른 시간 = 1026 / message = data received ! Rams num = 4
    thread name = main 실행 후 흐른 시간 = 1026 / message = data received ! Rams num = 5
    thread name = RxCachedThreadScheduler-7 실행 후 흐른 시간 = 1026 / message = Get name ( Rams num = 6 ) from server
    thread name = main 실행 후 흐른 시간 = 1027 / message = data received ! Rams num = 6
    thread name = RxCachedThreadScheduler-10 실행 후 흐른 시간 = 1031 / message = Get name ( Rams num = 9 ) from server
    thread name = RxCachedThreadScheduler-11 실행 후 흐른 시간 = 1031 / message = Get name ( Rams num = 1 ) from server
    thread name = RxCachedThreadScheduler-12 실행 후 흐른 시간 = 1032 / message = Get name ( Rams num = 2 ) from server
    thread name = RxCachedThreadScheduler-8 실행 후 흐른 시간 = 1032 / message = Get name ( Rams num = 7 ) from server
    thread name = main 실행 후 흐른 시간 = 1033 / message = data received ! Rams num = 9
    thread name = RxCachedThreadScheduler-9 실행 후 흐른 시간 = 1034 / message = Get name ( Rams num = 8 ) from server
    thread name = main 실행 후 흐른 시간 = 1034 / message = data received ! Rams num = 1
    thread name = main 실행 후 흐른 시간 = 1034 / message = data received ! Rams num = 2
    thread name = main 실행 후 흐른 시간 = 1035 / message = data received ! Rams num = 7
    thread name = main 실행 후 흐른 시간 = 1035 / message = data received ! Rams num = 8
    ```
    
    - `clicks()` operator에 대해
        - `clicks()` operator는 유저가 btnFlatmap 버튼을 클릭 하면, Unit 객체를 아래로 발행 해 주는 operator 입니다.
        - 즉, View에서 발행한 이벤트를 RxStream으로 변형해주는 역할을 하고 있습니다.
        - `clicks()` operator에 대한 상세한 설명은 추후 포스팅 할 RxBinding 포스팅에서 다룰 예정입니다.
    - Multi Threading에 대해
        - 실제 작업이 진행되는 thread는 `RxCachedThreadScheduler` 인 것을 확인 할 수 있으며, 모든 작업이 병렬적으로 실행 되었음을 확인 할 수 있습니다.
        - `getNameFromServerObservable` 메소드 안의 `subscribeOn(Schedulers.io())` operator를 통해 Multi Thread를 활용하고 있음을 확인 할 수 있습니다.
    - flatMap 분석
        - 첫번째 flatMap을 통해 유저의 버튼 클릭 이벤트를 list에 담긴 데이터 ( `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` )를 하나씩 방출하는 fromIterable stream으로 변형 해 줍니다.
        - 즉 click event 하나를 방출하는 stream이 `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` 를 방출하는 stream으로 변형 된 것입니다.
        - 두번째 flatMap을 통해, fromIterable stream에서 발행된 데이터 하나하나 마다 `getNameFromServerObservable` 메소드를 활용하여 서버에서 이름 데이터를 가져오는 observable stream으로 변형 해 주고 있습니다.
        - 즉, 숫자 하나하나를 방출하는 stream이 숫자 마다 `getNameFromServerObservable` 를 통해 서버에서 이름을 가져와 가져온 데이터를 방출하는 stream으로 변형 된 것입니다.
        - 그 뒤, 서버에서 이름을 가져오는걸 성공한 순서대로 데이터가 방출된 것을 확인 할 수 있습니다.
    
    ```kotlin
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerSingle(studentNumber: Int): Single<String> =
        Single
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnFlatmap
                .clicks()
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .flatMapSingle { getNameFromServerSingle(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    ```
    
    - 위의 예시에선 observableSource를 인자로 받는 `flatMap` operator를 활용 하여 stream을 변형 해 주었고, 아래의 예시에선 `flatMapSingle` operator를 통해 singleSource 형태의 stream으로 변형 해 준것을 확인 할 수 있습니다.

## 📖 SwitchMap

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%2010.png)
    
    - switchMap 안에 적힌 것을 보면 동그라미 데이터가 발행 되면, 마름모 데이터하나와 네모 데이터 하나를 발행하는 Stream으로 변형해 주는 것을 확인 할 수 있습니다.
    - 빨간색 동그라미가 발행 되고, 빨간색 마름모와 네모 데이터가 발행 된 것을 확인 할 수 있습니다.
    - 초록색 동그라미가 발행 되고, 초록색 마름모가 발행 된 뒤, 초록색 네모가 발행되기 전 파란색 동그라미가 발행 된것을 확인 할 수 있습니다.
    - 이때 초록색 네모가 발행되지 않고, 파란색 마름모와 파란색 네모가 발행 된 것을 확인 할 수 있습니다.
    - 즉, 초록색 stream의 데이터 발행이 파란색 stream의 데이터 발행에 의해 취소 된 것입니다.
- 특징
    - flatMap 처럼 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator 입니다.
    - Stream을 다른 Stream으로 변형 할 때 사용 되며, 이전의 stream을 취소시키는 기능을 갖고 있습니다.
    - observable에 있는 switchMap operator는 observableSource을 인자로 받고, Single에 있는 switchMap operator는 SingleSource를 인자로 받습니다.
    - flatMap과 마찬가지로, observable stream에서 switchMap을 통해 single stream에서 발행되는 데이터를 받고자 하는 경우 `switchMapSingle` operator를, maybe stream에서 발행되는 데이터를 받고자 하는 경우에는 `switchMapMaybe` operator를 활용하면 됩니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerObservable(studentNumber: Int): Observable<String> =
        Observable
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnSwitchmap
                .clicks()
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .switchMap { getNameFromServerObservable(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 18 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 19 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 19 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 20 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 21 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 20 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 21 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 21 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 21 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 21 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 22 / message = Task is disposed !
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 1024 / message = Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 1026 / message = data received ! Rams num = 2
    ```
    
    - `clicks()` operator에 대해
        - flatMap에서 설명한것과 같습니다. 이에 대한 자세한 내용은 추후 포스팅에서 다룰 예정입니다.
    - Multi Threading에 대해
        - 실제 작업이 진행되는 thread는 `RxCachedThreadScheduler` 인 것을 확인 할 수 있습니다.
        - `getNameFromServerObservable` 메소드 안의 `subscribeOn(Schedulers.io())` operator를 통해 Multi Thread를 활용하고 있음을 확인 할 수 있습니다.
    - switchMap 분석
        - 첫번째 flatMap을 통해 유저의 버튼 클릭 이벤트를 list에 담긴 데이터 ( `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` )를 하나씩 방출하는 fromIterable stream으로 변형 해 줍니다.
        - 즉 click event 하나를 방출하는 stream이 `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` 를 방출하는 stream으로 변형 된 것입니다.
        - 그 뒤 switchMap을 통해, fromIterable stream에서 발행된 데이터 하나하나 마다 `getNameFromServerObservable` 메소드를 활용하여 서버에서 이름 데이터를 가져오는 observable stream으로 변형 해 주고 있습니다.
        - 즉, 숫자 하나하나를 방출하는 stream이 숫자 마다 `getNameFromServerObservable` 를 통해 서버에서 이름을 가져와 가져온 데이터를 방출하는 stream으로 변형 된 것입니다.
        - 하지만, switchMap은 `getNameFromServerObservable` stream의 데이터 발행이 끝나기 전, 새롭게 `getNameFromServerObservable` 데이터 발행이 이루어 질 때 이전의 작업은 취소시킵니다.
        - 따라서, 맨 마지막에 발행된 `2` 에 대한 stream 변환만 이루어 진 것을 확인 할 수 있으며, `2` 에 대한 데이터 발행만 진행 된 것을 확인 할 수 있습니다.
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerSingle(studentNumber: Int): Single<String> =
        Single
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnSwitchmap
                .clicks()
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .switchMapSingle { getNameFromServerSingle(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    ```
    
    - 위의 예시에선 observableSource를 인자로 받는 `switchMap` operator를 활용 하여 stream을 변형 해 주었고, 아래의 예시에선 `switchMapSingle` operator를 통해 singleSource 형태의 stream으로 변형 해 준것을 확인 할 수 있습니다.

## 📖 ConcatMap

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-26-transform-operator/Untitled%2011.png)
    
    - concatMap 안에 적힌 것을 보면 동그라미 데이터가 발행 되면, 마름모 데이터를 두개 발행하는 Stream으로 변형해 주는 것을 확인 할 수 있습니다.
    - 빨간색 동그라미가 발행 되고, 빨간색 마름모 두개가 발행 된 것을 확인 할 수 있습니다.
    - 초록색 동그라미가 발행 되고, 초록색 마름모가 발행 된 뒤, 파란색 동그라미가 발행 된 시점에 파란색 마름모를 발행시키지 않고 초록색 마름모를 발행 시키는 것을 확인 할 수 있습니다.
    - 초록색 마름모가 발행 된 이후, 파란색 마름모 두개가 발행 된 것을 확인 할 수 있습니다.
    - 즉, 초록색 stream의 데이터 발행이 끝나기 전에는 파란색 stream의 데이터 발행이 시작되지 않았음을 확인 할 수 있습니다.
- 특징
    - flatMap 처럼 데이터를 Stream으로 변경하고, 새로운 Stream을 구독 후 발행되는 데이터를 발행 해주는 operator 입니다.
    - Stream을 다른 Stream으로 변형 할 때 사용 되며, 작업의 순서를 보장해주는 기능을 갖고있습니다.
    - observable에 있는 concatMap operator는 observableSource을 인자로 받고, Single에 있는 concatMap operator는 SingleSource를 인자로 받습니다.
    - flatMap과 마찬가지로, observable stream에서 concatMap을 통해 single stream에서 발행되는 데이터를 받고자 하는 경우 `concatMapSingle` operator를, maybe stream에서 발행되는 데이터를 받고자 하는 경우에는 `concatMapMaybe` operator를 활용하면 됩니다.
- 예시코드
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerObservable(studentNumber: Int): Observable<String> =
        Observable
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnConcatmap
                .clicks()
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .concatMap { getNameFromServerObservable(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    
    // 실행 결과
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 1005 / message = Get name ( Rams num = 0 ) from server
    thread name = main 실행 후 흐른 시간 = 1007 / message = data received ! Rams num = 0
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 2011 / message = Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 2013 / message = data received ! Rams num = 1
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 3014 / message = Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 3015 / message = data received ! Rams num = 2
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 4017 / message = Get name ( Rams num = 3 ) from server
    thread name = main 실행 후 흐른 시간 = 4018 / message = data received ! Rams num = 3
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 5020 / message = Get name ( Rams num = 4 ) from server
    thread name = main 실행 후 흐른 시간 = 5022 / message = data received ! Rams num = 4
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 6023 / message = Get name ( Rams num = 5 ) from server
    thread name = main 실행 후 흐른 시간 = 6025 / message = data received ! Rams num = 5
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 7026 / message = Get name ( Rams num = 6 ) from server
    thread name = main 실행 후 흐른 시간 = 7028 / message = data received ! Rams num = 6
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 8029 / message = Get name ( Rams num = 7 ) from server
    thread name = main 실행 후 흐른 시간 = 8030 / message = data received ! Rams num = 7
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 9032 / message = Get name ( Rams num = 8 ) from server
    thread name = main 실행 후 흐른 시간 = 9033 / message = data received ! Rams num = 8
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 10034 / message = Get name ( Rams num = 9 ) from server
    thread name = main 실행 후 흐른 시간 = 10036 / message = data received ! Rams num = 9
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 11037 / message = Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 11039 / message = data received ! Rams num = 1
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 12041 / message = Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 12042 / message = data received ! Rams num = 2
    ```
    
    - `clicks()` operator에 대해
        - flatMap에서 설명한것과 같습니다. 이에 대한 자세한 내용은 추후 포스팅에서 다룰 예정입니다.
    - Multi Threading에 대해
        - 실제 작업이 진행되는 thread는 `RxCachedThreadScheduler` 인 것을 확인 할 수 있습니다.
        - `getNameFromServerObservable` 메소드 안의 `subscribeOn(Schedulers.io())` operator를 통해 Multi Thread를 활용하고 있음을 확인 할 수 있습니다.
    - concatMap 분석
        - 첫번째 flatMap을 통해 유저의 버튼 클릭 이벤트를 list에 담긴 데이터 ( `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` )를 하나씩 방출하는 fromIterable stream으로 변형 해 줍니다.
        - 즉 click event 하나를 방출하는 stream이 `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` 를 방출하는 stream으로 변형 된 것입니다.
        - 그 뒤 concatMap을 통해, fromIterable stream에서 발행된 데이터 하나하나 마다 `getNameFromServerObservable` 메소드를 활용하여 서버에서 이름 데이터를 가져오는 observable stream으로 변형 해 주고 있습니다.
        - 즉, 숫자 하나하나를 방출하는 stream이 숫자 마다 `getNameFromServerObservable` 를 통해 서버에서 이름을 가져와 가져온 데이터를 방출하는 stream으로 변형 된 것입니다.
        - concatMap은 데이터 발행을 순차적으로 진행하는 특성이 있기 때문에 `0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2` 의 순서대로 서버 통신이 이루어진것을 확인 할 수 있습니다.
    
    ```kotlin
    private val emittedIntegerList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2)
    private val fromIterableSource = Observable.fromIterable(emittedIntegerList)
    
    private fun startTaskToGetNameFromServer(studentNumber: Int): String =
        "Rams num = $studentNumber".also { Thread.sleep(1000); timeStampedLog("Get name ( $it ) from server") }
    
    private fun getNameFromServerSingle(studentNumber: Int): Single<String> =
        Single
            .fromCallable { startTaskToGetNameFromServer(studentNumber) }
            .subscribeOn(Schedulers.io())
            .doOnDispose { timeStampedLog("Task is disposed !") }
    
    compositeDisposable
        .add(
            binding
                .btnConcatmap
                .clicks()
                .doOnNext { startedTime = System.currentTimeMillis() }
                .flatMap { fromIterableSource }
                .concatMapSingle { getNameFromServerSingle(it) }
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                    timeStampedLog("data received ! $it")
                }, { it.printStackTrace() })
        )
    ```
    
    - 위의 예시에선 observableSource를 인자로 받는 `concatMap` operator를 활용 하여 stream을 변형 해 주었고, 아래의 예시에선 `concatMapSingle` operator를 통해 singleSource 형태의 stream으로 변형 해 준것을 확인 할 수 있습니다.