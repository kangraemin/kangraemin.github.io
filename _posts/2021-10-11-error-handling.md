---
title: RxJava 강의 12 - 에러 처리 연산자 ( retry / retrywhen / onErrorResumeNext / onErrorReturn )
date: 2021-10-11
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

Stream에서 데이터 처리 도중에 발생한 에러를 다루는 operator들의 특징과 예시 코드들을 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Retry

- 스트림에서 에러가 발생 하였을 때, 스트림을 재구독 해주는 operator
- 따로 인자를 받지 않는 `retry()` operator 는 횟수 제한 없이 재구독
- `times` 라는 인자를 주어 retry 횟수를 지정 할 수 있음
- Boolean을 리턴하는 함수를 의미하는 `predicate` 를 주어 재구독 할 조건을 지정 할 수 있음
- retry operator를 위치시키는 위치가 중요 할 수 있음

## 📖 RetryWhen

- retry 조건을 stream으로 지정할 때 사용되는 operator
- 에러를 감싸서 하나의 데이터로 방출하는 stream이 인자로 존재하고, 이 에러 stream에서 데이터가 방출 될 때 재시도가 이루어 짐
- 에러 stream을 `flatMap` / `switchMap` 등의 operator로 적절히 변경하여 사용 할 수 있음

## 📖 OnErrorResumeNext

- 에러가 발생 하였을 때, 스트림을 다른 스트림으로 변환 해 주는 operator
- 에러가 발생 하였을 때, 재시도가 아닌 스트림을 다른 스트림으로 변환하고자 할 때 주로 사용됨

## 📖 OnErrorReturn

- 에러가 발생 했을 때, 특정 데이터 형식의 아이템을 발행 해주는 operator
- 에러가 발생 하더라도 특정 형식의 데이터가 발행될 뿐 정상적으로 작동

# 📚 에러 처리 연산자

## 📖 개요

 Stream에서 데이터 발행 / 이벤트 처리 도중, 에러가 발생 했을 때 어떤 처리를 해줄지를 다루는 operator들을 알아보고자 합니다. 

 예시 코드를 통해 각 operator의 특징들을 살펴 볼 예정이며 아래에 쓰인 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/blob/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/operator/error/ErrorHandlingExampleActivity.kt)에서 확인 하실 수 있습니다. 

 또 아래의 예시 코드에서 공통적으로 쓰인 코드는 아래와 같으니 참고하여 코드 확인하시면 되겠습니다. 

```kotlin
// 예시 코드를 실행한 시간을 나타내기 위한 변수
// 예시 코드를 실행하면, 해당 변수가 실행한 시각으로 변경됩니다. 
private var startedTime = 0L

// 서버에서 이름 데이터를 가져오기 위한 함수입니다. 
private fun startTaskToGetNameFromServer(studentNumber: Int): String =
    "Rams num = $studentNumber".also {
        timeStampedLog("Start to Get name ( $it ) from server")
        Thread.sleep(1000)
        timeStampedLog("Success to Get name ( $it ) from server")
    }

// 서버에서 이름 데이터를 가져오기 위한 Observable 입니다. 
private fun getNameFromServerSingle(studentNumber: Int): Single<String> =
    Single
        .fromCallable { startTaskToGetNameFromServer(studentNumber) }
        .subscribeOn(Schedulers.io())
        .doOnDispose { timeStampedLog("Task is disposed !") }

// 예시 코드가 실행된 Thread name과 실행 한 시각과 로그가 찍힌 시각의 차이를 표기하기 위한 로그입니다.  
private fun timeStampedLog(message: Any) {
    Log.d(
        TAG_TRANSFORM_OPERATOR,
        "thread name = ${Thread.currentThread().name} 실행 후 흐른 시간 = ${System.currentTimeMillis() - startedTime} / message = $message"
    )
}

// 1, 2, 3, 4, 5 데이터를 차례대로 방출하고, 3이 방출 될 때 CustomException 에러를 방출하는 Observable 입니다. 
private val emittedIntegerList = listOf(1, 2, 3, 4, 5)
private val fromIterableSource =
    Observable
        .fromIterable(emittedIntegerList)
        .doOnNext { timeStampedLog("emitted value = $it") }
        .doOnNext {
            if (it == 3) {
                timeStampedLog("throw error in $it")
                throw CustomException
            }
        }

private object CustomException : Throwable()
```

## 📖 Default Example

- 에러 처리를 따로 하지 않은 경우입니다.
- `fromIterableSource` 스트림에서 3이 방출 될 때 `CustomException` 에러가 방출됩니다.
- 예시 코드
    
    ```kotlin
    private fun runDefaultExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable.just(Unit)
                    .doOnNext { timeStampedLog("작업을 시작합니다.") }
                    .flatMap { fromIterableSource }
                    .flatMapSingle { getNameFromServerSingle(it) }
                    .observeOn(AndroidSchedulers.mainThread())
                    .doOnError { timeStampedLog("Error 발생 !") }
                    .subscribe({
                        timeStampedLog("data received ! $it")
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과 
    thread name = main 실행 후 흐른 시간 = 1 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 29 / message = emitted value = 2
    thread name = main 실행 후 흐른 시간 = 30 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 30 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 31 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 31 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 36 / message = Error 발생 !
    W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
    W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:125)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:19)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.onNext(ObservableRetryPredicate.java:69)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
            at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
            at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.Observable.subscribe(Observable.java:12270)
            at io.reactivex.Observable.subscribe(Observable.java:12198)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
            at android.view.View.performClick(View.java:7870)
            at android.widget.TextView.performClick(TextView.java:14970)
            at android.view.View.performClickInternal(View.java:7839)
            at android.view.View.access$3600(View.java:886)
            at android.view.View$PerformClick.run(View.java:29363)
            at android.os.Handler.handleCallback(Handler.java:883)
    W/System.err:     at android.os.Handler.dispatchMessage(Handler.java:100)
            at android.os.Looper.loop(Looper.java:237)
            at android.app.ActivityThread.main(ActivityThread.java:7860)
            at java.lang.reflect.Method.invoke(Native Method)
            at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
            at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
    ```
    
    - 코드가 실행되고, `작업이 시작됩니다` 가 한번 출력 된 것을 확인 할 수 있습니다.
    - `thread name = main 실행 후 흐른 시간 = 31 / message = Task is disposed !` 를 살펴보면, 3을 방출 하며 Error가 발생 했을 때, 이미 방출된 1, 2를 통해 진행중이던 작업은 dispose 되는 것을 확인 할 수 있습니다.
    - 에러가 발생되고, `printStackTrace()` 와 함께 `Error 발생 !` 문구가 발행 된 것을 확인 할 수 있습니다.

## 📖 Retry

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-10-11-error-handling/Untitled.png)
    
    - 빨간색 / 노란색 데이터가 방출 되고, 에러가 발생 했을 때 재구독이 이루어 지고 있습니다.
    - 재구독이 진행되었을 때, 데이터를 처음부터 방출하며 초록색 데이터가 방출되며 성공적으로 데이터가 방출 된 것을 확인 할 수 있습니다.
- 특징
    - 스트림에서 에러가 발생 하였을 때, 스트림을 재구독 해주는 operator 입니다.
    - 따로 인자를 받지 않는 `retry()` operator 는 횟수 제한 없이 재구독합니다.
    - `times` 라는 인자를 주어 retry 횟수를 지정 할 수 있습니다.
    - Boolean을 리턴하는 함수를 의미하는 `predicate` 를 주어 재구독 할 조건을 지정 할 수 있습니다.
    - retry operator를 위치시키는 위치가 중요 할 수 있습니다.
- 예시 코드
    - 아무런 인자가 없는 `retry` operator
        
        ```kotlin
        private fun runRetryExample() {
            startedTime = System.currentTimeMillis()
            compositeDisposable
                .add(
                    Observable.just(Unit)
                        .doOnNext { timeStampedLog("작업을 시작합니다.") }
                        .flatMap { fromIterableSource }
                        .flatMapSingle { getNameFromServerSingle(it) }
                        .retry()
                        .observeOn(AndroidSchedulers.mainThread())
                        .doOnError { timeStampedLog("Error 발생 !") }
                        .subscribe({
                            timeStampedLog("data received ! $it")
                        }, { it.printStackTrace() })
                )
        }
        
        // 실행 결과 
        thread name = main 실행 후 흐른 시간 = 1 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 1 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 2 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 2 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 2 / message = Task is disposed !
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 2 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 2 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 3 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 3 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 4 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 4 / message = Task is disposed !
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 4 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 4 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 4 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 4 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 5 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 5 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 5 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 5 / message = throw error in 3
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 5 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 5 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 6 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 6 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 6 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 6 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 6 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 7 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 7 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 7 / message = Task is disposed !
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 7 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 7 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 7 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 7 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 8 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 8 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 8 / message = emitted value = 3
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 8 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 8 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 8 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 8 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 9 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 9 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 9 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 9 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 9 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 9 / message = throw error in 3
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 9 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 9 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 10 / message = Task is disposed !
        ...
        ```
        
        - 에러가 발생 했을 때, `retry`  재구독이 시작되어 Stream의 처음부터 다시 데이터가 흐르고 있음을 확인 할 수 있습니다.
        - `retry` 의 인자로써 아무런 값이 없기 때문에, 성공할 때 까지 ( 에러가 발생하지 않을 때 까지 )지속적으로 재구독이 이루어 집니다.
        - 흐른 시간을 살펴보면, 따로 기다리는 대기시간 없이 바로바로 재구독이 이루어 지고 있음을 확인 할 수 있습니다.
    - `times` 인자가 주어진 `retry` operator
        
        ```kotlin
        private fun runRetryExample() {
            startedTime = System.currentTimeMillis()
            compositeDisposable
                .add(
                    Observable.just(Unit)
                        .doOnNext { timeStampedLog("작업을 시작합니다.") }
                        .flatMap { fromIterableSource }
                        .flatMapSingle { getNameFromServerSingle(it) }
                        .retry(3)
                        .observeOn(AndroidSchedulers.mainThread())
                        .doOnError { timeStampedLog("Error 발생 !") }
                        .subscribe({
                            timeStampedLog("data received ! $it")
                        }, { it.printStackTrace() })
                )
        }
        
        // 실행 결과 
        thread name = main 실행 후 흐른 시간 = 1 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 1 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 2 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 2 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 2 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 2 / message = Task is disposed !
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 2 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 3 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 3 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 3 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 3 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 4 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 4 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 4 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 4 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 5 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 5 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 5 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 5 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 5 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 5 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 5 / message = emitted value = 1
        thread name = main 실행 후 흐른 시간 = 6 / message = emitted value = 2
        thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 6 / message = Start to Get name ( Rams num = 1 ) from server
        thread name = main 실행 후 흐른 시간 = 6 / message = emitted value = 3
        thread name = main 실행 후 흐른 시간 = 6 / message = throw error in 3
        thread name = main 실행 후 흐른 시간 = 6 / message = Task is disposed !
        thread name = main 실행 후 흐른 시간 = 7 / message = Task is disposed !
        thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 6 / message = Start to Get name ( Rams num = 2 ) from server
        thread name = main 실행 후 흐른 시간 = 9 / message = Error 발생 !
        W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
        W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$fromIterableSource$2.accept(ErrorHandlingExampleActivity.kt:54)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$fromIterableSource$2.accept(ErrorHandlingExampleActivity.kt:19)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableFromIterable$FromIterableDisposable.run(ObservableFromIterable.java:98)
        W/System.err:     at io.reactivex.internal.operators.observable.ObservableFromIterable.subscribeActual(ObservableFromIterable.java:58)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMap$MergeObserver.subscribeInner(ObservableFlatMap.java:165)
                at io.reactivex.internal.operators.observable.ObservableFlatMap$MergeObserver.onNext(ObservableFlatMap.java:139)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
                at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
        W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.Observable.subscribe(Observable.java:12270)
                at io.reactivex.Observable.subscribe(Observable.java:12198)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
                at android.view.View.performClick(View.java:7870)
                at android.widget.TextView.performClick(TextView.java:14970)
                at android.view.View.performClickInternal(View.java:7839)
                at android.view.View.access$3600(View.java:886)
                at android.view.View$PerformClick.run(View.java:29363)
                at android.os.Handler.handleCallback(Handler.java:883)
                at android.os.Handler.dispatchMessage(Handler.java:100)
                at android.os.Looper.loop(Looper.java:237)
                at android.app.ActivityThread.main(ActivityThread.java:7860)
                at java.lang.reflect.Method.invoke(Native Method)
                at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
                at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
        ```
        
        - `times` 의 값으로 3이 주어졌습니다.
        - `작업을 시작합니다.` 라는 로그가 네번 찍힌 것을 확인 할 수 있는데 이는 한번의 최초 시도, 세번의 재시도가 이루어 졌음을 의미합니다.
        - `Error 발생 !` 이라는 로그와 함께 `printStackTrace`가 찍힌 것을 확인 할 수 있습니다.
    - `retry` operator의 위치에 따른 재시도 여부 확인
        
        ```kotlin
        private fun runRetryExample() {
            startedTime = System.currentTimeMillis()
            compositeDisposable
                .add(
                    Observable.just(Unit)
                        .doOnNext { timeStampedLog("작업을 시작합니다.") }
                        .retry(3)
                        .doOnNext { timeStampedLog("작업을 시작합니다22.") }
                        .doOnNext { throw CustomException }
                        .flatMap { fromIterableSource }
                        .flatMapSingle { getNameFromServerSingle(it) }
                        .observeOn(AndroidSchedulers.mainThread())
                        .doOnError { timeStampedLog("Error 발생 !") }
                        .subscribe({
                            timeStampedLog("data received ! $it")
                        }, { it.printStackTrace() })
                )
        }
        
        // 실행 결과
        thread name = main 실행 후 흐른 시간 = 51 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 51 / message = 작업을 시작합니다22.
        thread name = main 실행 후 흐른 시간 = 58 / message = Error 발생 !
        W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:125)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:19)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.onNext(ObservableRetryPredicate.java:69)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
                at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
        W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
        W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.Observable.subscribe(Observable.java:12270)
                at io.reactivex.Observable.subscribe(Observable.java:12198)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
                at android.view.View.performClick(View.java:7870)
                at android.widget.TextView.performClick(TextView.java:14970)
                at android.view.View.performClickInternal(View.java:7839)
                at android.view.View.access$3600(View.java:886)
                at android.view.View$PerformClick.run(View.java:29363)
                at android.os.Handler.handleCallback(Handler.java:883)
                at android.os.Handler.dispatchMessage(Handler.java:100)
                at android.os.Looper.loop(Looper.java:237)
                at android.app.ActivityThread.main(ActivityThread.java:7860)
                at java.lang.reflect.Method.invoke(Native Method)
                at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
        W/System.err:     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
        ```
        
        - `retry` operator가 Exception을 방출하는곳 이전에 선언되었는데, 재구독이 이루어 지지 않았음을 확인 할 수 있습니다.
        
        ```kotlin
        private fun runRetryExample() {
            startedTime = System.currentTimeMillis()
            compositeDisposable
                .add(
                    Observable.just(Unit)
                        .doOnNext { timeStampedLog("작업을 시작합니다.") }
                        .doOnNext { timeStampedLog("작업을 시작합니다22.") }
                        .doOnNext { throw CustomException }
                        .retry(3)
                        .flatMap { fromIterableSource }
                        .flatMapSingle { getNameFromServerSingle(it) }
                        .observeOn(AndroidSchedulers.mainThread())
                        .doOnError { timeStampedLog("Error 발생 !") }
                        .subscribe({
                            timeStampedLog("data received ! $it")
                        }, { it.printStackTrace() })
                )
        }
        
        // 실행 결과
        thread name = main 실행 후 흐른 시간 = 62 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 62 / message = 작업을 시작합니다22.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다22.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다22.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다.
        thread name = main 실행 후 흐른 시간 = 63 / message = 작업을 시작합니다22.
        thread name = main 실행 후 흐른 시간 = 69 / message = Error 발생 !
        W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
        W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:125)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:19)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
                at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
                at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
        W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
                at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
                at io.reactivex.Observable.subscribe(Observable.java:12284)
                at io.reactivex.Observable.subscribe(Observable.java:12270)
                at io.reactivex.Observable.subscribe(Observable.java:12198)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
                at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
        W/System.err:     at android.view.View.performClick(View.java:7870)
                at android.widget.TextView.performClick(TextView.java:14970)
                at android.view.View.performClickInternal(View.java:7839)
                at android.view.View.access$3600(View.java:886)
                at android.view.View$PerformClick.run(View.java:29363)
                at android.os.Handler.handleCallback(Handler.java:883)
                at android.os.Handler.dispatchMessage(Handler.java:100)
                at android.os.Looper.loop(Looper.java:237)
                at android.app.ActivityThread.main(ActivityThread.java:7860)
                at java.lang.reflect.Method.invoke(Native Method)
                at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
                at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
        ```
        
        - `retry` operator가 Exception을 방출하는곳 이후에 선언되었는데, 재구독이 3번 이루어 졌음을 확인 할 수 있습니다.

## 📖 Retry With Predicate

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-10-11-error-handling/Untitled%201.png)
    
    - 에러가 발생 했을 때, 조건문의 함수를 실행시켜 true가 나오면 재구독을 진행합니다.
    - 에러가 발생 했을 때, 조건문의 함수를 실행시켜 false가 나오면 재구독을 진행하지 않고 에러를 방출하고 있습니다.
    - `retry` operator와 마찬가지로 times 인자를 통해 재구독 횟수를 정할 수 있습니다.
- 특징
    - `retry` operator의 오버로딩 형태 중 하나이며, `predicate` 인자를 준  형태입니다.
    - 재구독을 진행 할 조건을 정할 수 있습니다.
- 예시 코드
    
    ```kotlin
    private fun runRetryPredicateExample() {
        startedTime = System.currentTimeMillis()
        var count = 0
        compositeDisposable
            .add(
                Observable.just(Unit)
                    .doOnNext { timeStampedLog("작업을 시작합니다.") }
                    .flatMap { fromIterableSource }
                    .flatMapSingle { getNameFromServerSingle(it) }
                    .retry { _: Throwable ->
                        Thread.sleep(1000)
                        return@retry count++ < maxRetries
                    }
                    .observeOn(AndroidSchedulers.mainThread())
                    .doOnError { timeStampedLog("Error 발생 !") }
                    .subscribe({
                        timeStampedLog("data received ! $it")
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 44 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 45 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 63 / message = emitted value = 2
    thread name = main 실행 후 흐른 시간 = 64 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 64 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 65 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 66 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 1066 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 1067 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 1067 / message = emitted value = 2
    thread name = main 실행 후 흐른 시간 = 1068 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 1068 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 1068 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 1068 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 2069 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 2069 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 2070 / message = emitted value = 2
    thread name = main 실행 후 흐른 시간 = 2071 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 2071 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 2072 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 2072 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 3073 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 3073 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 3074 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 3075 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 3075 / message = emitted value = 3
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 3075 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 3076 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 3076 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 3076 / message = Task is disposed !
    I/Choreographer: Skipped 244 frames!  The application may be doing too much work on its main thread.
    thread name = main 실행 후 흐른 시간 = 4095 / message = Error 발생 !
    W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
    W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:125)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:19)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.onNext(ObservableRetryPredicate.java:69)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
            at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12270)
            at io.reactivex.Observable.subscribe(Observable.java:12198)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
    W/System.err:     at android.view.View.performClick(View.java:7870)
            at android.widget.TextView.performClick(TextView.java:14970)
            at android.view.View.performClickInternal(View.java:7839)
            at android.view.View.access$3600(View.java:886)
            at android.view.View$PerformClick.run(View.java:29363)
    W/System.err:     at android.os.Handler.handleCallback(Handler.java:883)
            at android.os.Handler.dispatchMessage(Handler.java:100)
            at android.os.Looper.loop(Looper.java:237)
    W/System.err:     at android.app.ActivityThread.main(ActivityThread.java:7860)
            at java.lang.reflect.Method.invoke(Native Method)
            at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
    W/System.err:     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
    ```
    
    - 조건문 안에서는 `count++ < maxRetries` 함수를 통해 최대 시도 횟수와 현재 시도 횟수를 리턴 해주고 있습니다.
    - 단지 조건문의 결과값을 방출하기 전, 1초동안 Thread를 멈추고 있어 1초 뒤에 재시도를 진행하고 있습니다.
    - 위코드는 predicate 함수를 활용할 수 있다는 예시일 뿐, 1초 뒤에 재시도를 위해 위 코드와 같이 1초뒤에 조건문의 결과값을 리턴한다면 Main Thread 가 1초간 작업을 진행하지 않기 때문에 위 방법을 권장하지 않습니다.
    - 총 3번 재구독이 이루어 지고, 그 뒤에는 재구독이 이루어 지지 않았음을 확인 할 수 있습니다.

## 📖 RetryWhen

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-10-11-error-handling/Untitled%202.png)
    
    - `retryWhen` operator 안에는 에러가 발생 하였을 때, 에러를 감싸서 하나의 데이터로 방출하고 있는 스트림이 있습니다.
    - 이 에러 Stream을 다른 Stream으로 적절히 변형 ( 데이터를 방출 할 때 delay를 주고, 방출을 한번만 하는 Stream ) 합니다.
    - 변형된 스트림에서 파란색 네모가 발행될 때 재구독이 이루어집니다.
    - 또 에러가 발생 하면, 변형된 스트림에서는 한번만 에러 데이터를 받으므로 더이상 데이터를 발행하지 않으므로, error를 방출합니다.
- 특징
    - retry 조건을 stream으로 지정할 때 사용되는 operator 입니다.
    - 에러를 감싸서 하나의 데이터로 방출하는 stream이 인자로 존재하고, 이 에러 stream에서 데이터가 방출 될 때 재시도가 이루어 집니다.
    - 에러 stream을 `flatMap` / `switchMap` 등의 operator로 적절히 변경하여 사용 할 수 있습니다.
- 예시 코드
    
    ```kotlin
    private val maxRetries = 3
    
    private fun runRetryWhenExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable.just(Unit)
                    .doOnNext { timeStampedLog("작업을 시작합니다.") }
                    .flatMap { fromIterableSource }
                    .flatMapSingle { getNameFromServerSingle(it) }
                    .retryWhen { errors ->
                        val counter = AtomicInteger()
                        return@retryWhen errors
                            .flatMap {
                                if (it is CustomException && counter.getAndIncrement() < maxRetries) {
                                    return@flatMap Observable.timer(1000, TimeUnit.MILLISECONDS)
                                }
                                throw it
                            }
                    }
                    .observeOn(AndroidSchedulers.mainThread())
                    .doOnError { timeStampedLog("Error 발생 !") }
                    .subscribe({
                        timeStampedLog("data received ! $it")
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 6 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 6 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 7 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 7 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 7 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 7 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 7 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 7 / message = Task is disposed !
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 7 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1010 / message = 작업을 시작합니다.
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1011 / message = emitted value = 1
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1013 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 1013 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1014 / message = emitted value = 3
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 1014 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1014 / message = throw error in 3
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1015 / message = Task is disposed !
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1016 / message = Task is disposed !
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2019 / message = 작업을 시작합니다.
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2020 / message = emitted value = 1
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2022 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 2023 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2024 / message = emitted value = 3
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 2024 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2025 / message = throw error in 3
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2026 / message = Task is disposed !
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 2027 / message = Task is disposed !
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3030 / message = 작업을 시작합니다.
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3030 / message = emitted value = 1
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3032 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-2 실행 후 흐른 시간 = 3032 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3034 / message = emitted value = 3
    thread name = RxCachedThreadScheduler-1 실행 후 흐른 시간 = 3034 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3034 / message = throw error in 3
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3035 / message = Task is disposed !
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3036 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 3039 / message = Error 발생 !
    W/System.err: com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$CustomException
    W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:125)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$runRetryExample$3.accept(ErrorHandlingExampleActivity.kt:19)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:93)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.onNext(ObservableRetryPredicate.java:69)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach$DoOnEachObserver.onNext(ObservableDoOnEach.java:101)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableScalarXMap$ScalarDisposable.run(ObservableScalarXMap.java:248)
            at io.reactivex.internal.operators.observable.ObservableJust.subscribeActual(ObservableJust.java:35)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableRetryPredicate$RepeatObserver.subscribeNext(ObservableRetryPredicate.java:112)
            at io.reactivex.internal.operators.observable.ObservableRetryPredicate.subscribeActual(ObservableRetryPredicate.java:41)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableFlatMap.subscribeActual(ObservableFlatMap.java:55)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.internal.operators.observable.ObservableFlatMapSingle.subscribeActual(ObservableFlatMapSingle.java:48)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableObserveOn.subscribeActual(ObservableObserveOn.java:45)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12284)
            at io.reactivex.internal.operators.observable.ObservableDoOnEach.subscribeActual(ObservableDoOnEach.java:42)
            at io.reactivex.Observable.subscribe(Observable.java:12284)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12270)
    W/System.err:     at io.reactivex.Observable.subscribe(Observable.java:12198)
    W/System.err:     at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.runRetryExample(ErrorHandlingExampleActivity.kt:132)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity.access$runRetryExample(ErrorHandlingExampleActivity.kt:19)
            at com.example.rxjavalecture.operator.error.ErrorHandlingExampleActivity$onCreate$2.onClick(ErrorHandlingExampleActivity.kt:73)
    W/System.err:     at android.view.View.performClick(View.java:7870)
            at android.widget.TextView.performClick(TextView.java:14970)
            at android.view.View.performClickInternal(View.java:7839)
    W/System.err:     at android.view.View.access$3600(View.java:886)
            at android.view.View$PerformClick.run(View.java:29363)
            at android.os.Handler.handleCallback(Handler.java:883)
            at android.os.Handler.dispatchMessage(Handler.java:100)
    W/System.err:     at android.os.Looper.loop(Looper.java:237)
            at android.app.ActivityThread.main(ActivityThread.java:7860)
    W/System.err:     at java.lang.reflect.Method.invoke(Native Method)
            at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
    W/System.err:     at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1075)
    ```
    
    - `작업을 시작합니다.` 로그가 4번 찍혀있는데, 최초의 구독 1번 / 재구독 3번이 이루어 진것을 확인 할 수 있습니다.
    - errors는 `throwable`이 담긴 observable 입니다.
    - errors stream을 `flatMap`, `timer` operator를 통해 변형 해 주고 있습니다.
    - errors stream에 데이터가 방출, 즉 에러가 발생하면 발생된 에러가 특정 조건이 맞춰지면 ( if 문 안의 조건 ) `flatMap` 을 통해 `timer` stream으로 변환되고 있습니다.
    - `timer` 에서 데이터가 발행되면, 재구독이 이루어 지고 있습니다.
    - 위 조건문과 함꼐 활용한 retry 예시에는 Thread sleep을 활용하여 main thread를 pending 시켰지만, `timer` operator를 활용하는 경우엔 Computation Scheduler를 활용하여 main thread가 멈추지 않는 것을 확인 할 수 있습니다.
    - 이 예시에선 `timer` operator를 통해 error 스트림을 변형 하였지만, 에러가 났을 때, 다른 네트워크 통신 / 다른 비동기 작업을 실행하고 그 스트림에서 데이터가 방출 되었을 때 재시도를 하는 로직을 시도 할 수 있습니다.
    - 예를들어, accessToken이 만료된 경우, 서버에서 refreshToken을 활용하여 accessToken을 갱신하고 재시도 하는 경우에 retryWhen operator를 활용하면 쉽게 구현할 수 있습니다.
    - 조건문을 빠져 나온 경우엔 throw를 통해 에러를 밣생시키고 있습니다.

## 📖 OnErrorResumeNext

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-10-11-error-handling/Untitled%203.png)
    
    - 에러가 발생 하였을 때, 다른 stream으로 변환해주고 있습니다.
- 특징
    - 에러가 발생 하였을 때, 스트림을 다른 스트림으로 변환 해 주는 operator 입니다.
    - 에러가 발생 하였을 때, 재시도가 아닌 스트림을 다른 스트림으로 변환하고자 할 때 주로 사용됩니다.
- 예시 코드
    
    ```kotlin
    private fun runOnErrorResumeNextExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable.just(Unit)
                    .doOnNext { timeStampedLog("작업을 시작합니다.") }
                    .flatMap { fromIterableSource }
                    .flatMapSingle { getNameFromServerSingle(it) }
                    .onErrorResumeNext(getNameFromServerSingle(1).toObservable())
                    .observeOn(AndroidSchedulers.mainThread())
                    .doOnError { timeStampedLog("Error 발생 !") }
                    .subscribe({
                        timeStampedLog("data received ! $it")
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 2 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 2
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 4 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 4 / message = emitted value = 3
    thread name = main 실행 후 흐른 시간 = 4 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 5 / message = Task is disposed !
    thread name = RxCachedThreadScheduler-4 실행 후 흐른 시간 = 5 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 5 / message = Task is disposed !
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 6 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 1007 / message = Success to Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 1008 / message = data received ! Rams num = 1
    ```
    
    - 에러가 발생 하였을 때, `studentNumber`가 1인 경우에 이름 가져오는 stream으로 변환 후 데이터를 발행 하고 있습니다.
    - `onErrorResumeNext` operator는 `observableSource`를 인자로 받기 때문에 stream을 observable 형태로 변환 해 주어야 합니다.
    - `getNameFromServerSingle(1)` 작업 도중엔 에러가 발생되지 않기 때문에, `Error 발생 !` 로그가 발생되지 않은것을 확인 할 수 있습니다.

## 📖 OnErrorReturn

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-10-11-error-handling/Untitled%204.png)
    
    - 에러가 발생하면, 네모 형태로 데이터를 발행하고 있습니다.
- 특징
    - 에러가 발생 했을 때, 특정 데이터 형식의 아이템을 발행 해주는 operator 입니다.
    - 에러가 발생 하더라도 특정 형식의 데이터가 발행될 뿐 정상적으로 작동합니다.
- 예시 코드
    
    ```kotlin
    private fun runOnErrorReturnExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable.just(Unit)
                    .doOnNext { timeStampedLog("작업을 시작합니다.") }
                    .flatMap { fromIterableSource }
                    .flatMapSingle { getNameFromServerSingle(it) }
                    .onErrorReturn { "에러가 발생했었어요" }
                    .observeOn(AndroidSchedulers.mainThread())
                    .doOnError { timeStampedLog("Error 발생 !") }
                    .subscribe({
                        timeStampedLog("data received ! $it")
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 2 / message = 작업을 시작합니다.
    thread name = main 실행 후 흐른 시간 = 2 / message = emitted value = 1
    thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 2
    thread name = main 실행 후 흐른 시간 = 3 / message = emitted value = 3
    thread name = RxCachedThreadScheduler-4 실행 후 흐른 시간 = 3 / message = Start to Get name ( Rams num = 1 ) from server
    thread name = main 실행 후 흐른 시간 = 3 / message = throw error in 3
    thread name = main 실행 후 흐른 시간 = 3 / message = Task is disposed !
    thread name = RxCachedThreadScheduler-3 실행 후 흐른 시간 = 3 / message = Start to Get name ( Rams num = 2 ) from server
    thread name = main 실행 후 흐른 시간 = 4 / message = Task is disposed !
    thread name = main 실행 후 흐른 시간 = 6 / message = data received ! 에러가 발생했었어요
    ```
    
    - `thread name = main 실행 후 흐른 시간 = 6 / message = data received ! 에러가 발생했었어요` 로그를 통해 에러가 발생 했을 때, `에러가 발생했었어요` 라는 string값이 리턴 되는 것을 확인 할 수 있습니다.
    - `Error 발생 !` 로그가 찍히지 않은 것을 확인 할 수 있습니다.