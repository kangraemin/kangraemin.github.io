---
title: RxJava 강의 8 - 데이터 발행 연산자 ( Create / Just / Defer / FromCallable / Interval ... )
date: 2021-09-25
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava에서 데이터 방출을위한 Operator들( Create / Just / Defer / FromCallable ... )의 특징과 예시 코드들을 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Create

- 각 Stream의 구현체 ( Observable / Single ... )에 맞는 Emitter를 활용하여 데이터를 방출하는 operator
- 구독자가 stream을 구독 할 때마다 데이터를 방출

## 📖 Just

- 주어진 데이터를 바로 방출 하는 operator
- 객체 선언 즉시 데이터 방출을 시작
- 데이터가 방출되는 동안 Thread를 펜딩시키기 때문에, 사용에 유의 할 필요가 있음
- 한번 방출 된 데이터는, 내부적으로 저장되어 있기 때문에 observable을 재구독시 데이터 재발행을 하지 않고 저장된 데이터를 방출

## 📖 Defer

- Stream 객체를 구독하는 시점에 데이터 방출이 시작되도록 하는 operator
- Stream을 defer로 감싼 형태가 되기 때문에, Stream을 두개( defer Stream / defer 내부 Stream )를 만들어 주어야 함

## 📖 FromCallable

- Callable 인터페이스를 구현한 객체 내부의 함수를 동작시키고, call 함수의 결과값을 방출해주는 operator
- 구독자가 Stream을 구독 할 때 데이터 발행을 시작
- 네트워크 통신, DB 쿼리 작업 등 비동기 처리에 활용

## 📖 FromIterable

- Iterable 인터페이스를 구현하고 있는 객체를 인자로 받고, 객체 내부의 아이템들을 하나하나 꺼내어 데이터를 발행 해주는 operator

## 📖 Interval

- 일정한 시간 간격을 두고 0부터 시작하여 1씩 증가하는 숫자를 방출하는 operator
- 별다른 Scheduler 지정 하지 않을 시, Computation Thread에서 데이터 방출

## 📖 Timer

- 일정 시간이 지난 뒤, 아이템을 방출하고 데이터 방출 완료 이벤트를 방출하는 operator
- 별다른 Scheduler 지정 하지 않을 시, Computation Thread에서 데이터 방출

## 📖 Range

- n부터 시작하고, 1씩 증가하는 숫자를 m개를 방출하는 operator

# 📚 생성 연산자

## 📖 개요

 Stream을 통해 데이터를 방출 할 때 사용하는 Operator들을 알아보고자 합니다. 

 예시 코드를 통해 각 operator의 특징들을 살펴 볼 예정이며 아래에 쓰인 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/blob/7ebaaab6d65f413a4ad2f1048ae842bb43e6a1ad/AndroidProject/app/src/main/java/com/example/rxjavalecture/operator/create/CreateOperatorExampleActivity.kt)에서 확인 하실 수 있습니다. 

 또 아래의 예시 코드에서 공통적으로 쓰인 코드는 아래와 같으니 참고하여 코드 확인하시면 되겠습니다. 

```kotlin
// 예시 코드를 실행한 시간을 나타내기 위한 변수
// 예시 코드를 실행하면, 해당 변수가 실행한 시각으로 변경됩니다. 
private var startedTime = 0L

// 예시 코드를 실행 한 시각과 로그가 찍힌 시각의 차이를 표기하기 위한 로그입니다.  
private fun timeStampedLog(message: String) {
    Log.d(
        TAG_CREATE_OPERATOR,
        "실행 후 흐른 시간 = ${System.currentTimeMillis() - startedTime} / message = $message"
    )
}

// Thread name이 표시된 log 
private fun timeStampedLog(message: String) {
    Log.d(
        TAG_CREATE_OPERATOR,
        "thread name = ${Thread.currentThread().name} 실행 후 흐른 시간 = ${System.currentTimeMillis() - startedTime} / message = $message"
    )
}

// 데이터 방출에 쓰이는 작업들이며, 각 작업에는 1초가 소모되도록 구현하였습니다. 
private fun startTaskToGetFirstString(): String =
    "1".also { Thread.sleep(1000); timeStampedLog("Start task to emit $it") }

private fun startTaskToGetSecondString(): String =
    "2".also { Thread.sleep(1000); timeStampedLog("Start task to emit $it") }

private fun startTaskToGetThirdString(): String =
    "3".also { Thread.sleep(1000); timeStampedLog("Start task to emit $it") }
```

## 📖 Create

- Marble Diagram
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled.png)
    
- 특징
    - 각 Stream의 구현체 ( Observable / Single ... )에 맞는 Emitter를 활용하여 데이터를 방출합니다.
    - 구독자가 stream을 구독 할 때마다 데이터를 방출합니다.
    - onNext / onError / onComplete 등의 메소드를 잘 활용하여 데이터를 방출 하여야 합니다.
        - 예를들어, onComplete 메소드를 호출 한 이후, onNext를 통해 데이터를 방출하더라도 데이터는 방출되지 않습니다.
- 예시코드
    
    ```kotlin
    private fun runCreateExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        compositeDisposable
            .add(
                Observable.create<String> { emitter ->
                    emitter.onNext(startTaskToGetFirstString())
                    emitter.onNext(startTaskToGetSecondString())
                    emitter.onNext(startTaskToGetThirdString())
                }
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
        timeStampedLog("end task")
    }
    
    // 실행 결과 
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 43 / message = end task
    실행 후 흐른 시간 = 1048 / message = Start task to emit 1
    실행 후 흐른 시간 = 1048 / message = 1
    실행 후 흐른 시간 = 2049 / message = Start task to emit 2
    실행 후 흐른 시간 = 2050 / message = 2
    실행 후 흐른 시간 = 3051 / message = Start task to emit 3
    실행 후 흐른 시간 = 3051 / message = 3
    ```
    
    - onNext 메소드를 활용하여 데이터를 방출할 수 있음을 확인 할 수 있습니다.

## 📖 Just

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%201.png)
    
- 특징
    - 데이터를 바로 방출 하고자 할 때 사용합니다.
    - Stream 객체 선언 즉시 데이터 방출을 시작합니다.
        - 즉, stream을 생성 한 후 따로 구독하지 않더라도 데이터 발행이 진행됩니다.
    - 데이터가 방출되는 동안 Thread를 펜딩시키기 때문에, 사용에 유의 할 필요가 있습니다.
    - Just를 통해 한번 방출 된 데이터는, 내부적으로 저장되어 있기 때문에 observable을 재구독시 데이터 재발행을 하지 않고 저장된 데이터를 방출합니다. ( ScalarDisposable, ObservableFromArray 클래스 참조 )
- 예시코드
    
    ```kotlin
    private fun runJustExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        compositeDisposable
            .add(
                Observable.just(
                    startTaskToGetFirstString(),
                    startTaskToGetSecondString(),
                    startTaskToGetThirdString()
                )
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 1001 / message = Start task to emit 1
    실행 후 흐른 시간 = 2002 / message = Start task to emit 2
    실행 후 흐른 시간 = 3003 / message = Start task to emit 3
    실행 후 흐른 시간 = 3054 / message = end task
    실행 후 흐른 시간 = 3056 / message = 1
    실행 후 흐른 시간 = 3056 / message = 2
    실행 후 흐른 시간 = 3056 / message = 3
    ```
    
    - create 오퍼레이터에서는, start task / end task 로그가 별다른 대기 시간 없이 바로 실행 되었지만, start task, end task 로그를 기록 할 때, 약 3초간의 대기 시간이 존재 했음을 확인 할 수 있습니다.
    - 이 3초는 just operator 안에서, 데이터를 방출할 때 thread를 펜딩시켜 나온 결과입니다.
    - 따라서 just operator 안에서는, 시간이 많이 걸리는 비동기 작업 등을 하지 않는것이 권장됩니다.
    
    ```kotlin
    val justObservable = Observable.just(
        startTaskToGetFirstString(),
        startTaskToGetSecondString(),
        startTaskToGetThirdString()
    )
    
    private fun runJustExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        compositeDisposable
            .add(
                justObservable
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = Start task to emit 1
    실행 후 흐른 시간 = 1000 / message = Start task to emit 2
    실행 후 흐른 시간 = 2000 / message = Start task to emit 3
    
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 1 / message = end task
    실행 후 흐른 시간 = 2 / message = 1
    실행 후 흐른 시간 = 2 / message = 2
    실행 후 흐른 시간 = 2 / message = 3
    
    // 재실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 1 / message = end task
    실행 후 흐른 시간 = 2 / message = 1
    실행 후 흐른 시간 = 2 / message = 2
    실행 후 흐른 시간 = 2 / message = 3
    ```
    
    - 처음 `justObservable` 이 선언 될 때, 데이터 방출이 진행 되었음을 확인 할 수 있습니다.
    - `runJustExample()` 함수를 다시 실행 했을 때, 데이터 방출을 다시 진행하지 않고 내부적으로 저장 된 데이터를 활용하는 것을 확인 할 수 있습니다.

## 📖 Defer

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%202.png)
    
- 특징
    - Just 오퍼레이터는, observable 객체를 선언 할 때 데이터 방출이 시작 되는것을 확인 할 수 있었습니다.
    - defer 오퍼레이터는, 객체를 선언하는 시점에 데이터를 방출하는것이 아니라, stream 객체를 구독하는 시점에 데이터 방출이 시작되도록 합니다.
    - Stream을 defer로 감싼 형태가 되기 때문에, Stream을 두개( defer Stream / defer 내부 Stream )를 만들어 주어야 합니다.
- 예시코드
    
    ```kotlin
    private fun runDeferExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        compositeDisposable
            .add(
                Observable.defer {
                    Observable.just(
                        startTaskToGetFirstString(),
                        startTaskToGetSecondString(),
                        startTaskToGetThirdString()
                    )
                }
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 23 / message = end task
    실행 후 흐른 시간 = 1027 / message = Start task to emit 1
    실행 후 흐른 시간 = 2027 / message = Start task to emit 2
    실행 후 흐른 시간 = 3028 / message = Start task to emit 3
    실행 후 흐른 시간 = 3031 / message = 1
    실행 후 흐른 시간 = 3032 / message = 2
    실행 후 흐른 시간 = 3033 / message = 3
    ```
    
    - Just 오퍼레이터를 defer로 감싼 형태입니다.
    - Just 오퍼레이터와 다르게 데이터 방출을 구독하는 시점으로 미루기 때문에, start task 와 end task가 나란히 기록된것을 확인 할 수 있습니다.
    
    ```kotlin
    val deferObservable = Observable.defer {
        Observable.just(
            startTaskToGetFirstString(),
            startTaskToGetSecondString(),
            startTaskToGetThirdString()
        )
    }
    
    private fun runDeferExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        compositeDisposable
            .add(
                deferObservable
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 0 / message = end task
    실행 후 흐른 시간 = 1001 / message = Start task to emit 1
    실행 후 흐른 시간 = 2002 / message = Start task to emit 2
    실행 후 흐른 시간 = 3003 / message = Start task to emit 3
    실행 후 흐른 시간 = 3004 / message = 1
    실행 후 흐른 시간 = 3004 / message = 2
    실행 후 흐른 시간 = 3004 / message = 3
    
    // 재실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 0 / message = end task
    실행 후 흐른 시간 = 1001 / message = Start task to emit 1
    실행 후 흐른 시간 = 2002 / message = Start task to emit 2
    실행 후 흐른 시간 = 3003 / message = Start task to emit 3
    실행 후 흐른 시간 = 3004 / message = 1
    실행 후 흐른 시간 = 3004 / message = 2
    실행 후 흐른 시간 = 3004 / message = 3
    ```
    
    - Just 오퍼레이터에서는 재 구독시 이전에 발행했던 데이터를 저장하고 있기 때문에, 데이터 발행을 다시 진행하는것이 아니라 저장 된 데이터를 발행 해 줍니다.
    - 하지만 defer로 감싼 observable 에서, Just 오퍼레이터 처럼 observable을 변수에 따로 담아 놓더라도 재구독시 데이터 발행이 새롭게 이루어 지는 것을 확인 할 수 있습니다.
    
    ```kotlin
    private fun runDeferExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        Observable.defer {
            Observable.just(
                startTaskToGetFirstString(),
                startTaskToGetSecondString(),
                startTaskToGetThirdString()
            )
        }
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 1 / message = start task
    실행 후 흐른 시간 = 1 / message = end task
    ```
    
    - 구독 할 때 데이터 방출을 시작하기 때문에, 구독을 하지 않으면 데이터 방출도 하지 않음을 확인 할 수 있습니다.

## 📖 FromCallable

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%203.png)
    
- 특징
    - callable 인터페이스를 구현한 객체를 인자로 받습니다.
        
        ```kotlin
        @FunctionalInterface
        public interface Callable<V> {
            /**
             * Computes a result, or throws an exception if unable to do so.
             *
             * @return computed result
             * @throws Exception if unable to compute a result
             */
            V call() throws Exception;
        }
        ```
        
        - Callable 인터페이스는 runnable과 다르게 값을 리턴 할 수 있음을 확인 할 수 있습니다.
    - Callable 인터페이스를 구현한 객체 내부의 함수를 동작시키고, call 함수의 결과값을 방출해주는 operator 입니다.
    - 구독자가 Stream을 구독 할 때 데이터 발행을 시작합니다.
    - 네트워크 통신, DB 쿼리 작업 등 비동기 처리에 종종 활용 됩니다.
- 예시코드
    
    ```kotlin
    private fun runFromCallableExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        Observable.fromCallable {
            startTaskToGetFirstString()
            startTaskToGetSecondString()
            return@fromCallable startTaskToGetThirdString()
        }
    
         compositeDisposable
             .add(
                 Observable.fromCallable {
                     startTaskToGetFirstString()
                     startTaskToGetSecondString()
                     return@fromCallable startTaskToGetThirdString()
                 }
                     .subscribeOn(Schedulers.io())
                     .subscribe(::timeStampedLog)
             )
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 24 / message = end task
    실행 후 흐른 시간 = 1029 / message = Start task to emit 1
    실행 후 흐른 시간 = 2030 / message = Start task to emit 2
    실행 후 흐른 시간 = 3031 / message = Start task to emit 3
    실행 후 흐른 시간 = 3032 / message = 3
    ```
    
    - Defer와 같이 구독하지 않은 Stream은 데이터 발행도 되지 않음을 확인 할 수 있습니다.
    - fromCallable 내부에서 `startTaskToGetThirdString` 만 리턴 해 주고 있기 때문에, 3만 발행 된 것을 확인 할 수 있습니다.
    - defer와 같이 데이터 발행 시점을 객체를 선언하는 시점이 아닌 구독하는 시점으로 미루어 줍니다.

## 📖 FromIterable

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%204.png)
    
- 특징
    - Iterable 인터페이스를 구현하고 있는 객체를 인자로 받습니다.
    - List / Set 등을 인자로 활용 할 수 있으며, Iterable 내부의 아이템들을 하나하나 꺼내어 데이터를 발행 해 줍니다.
- 예시코드
    
    ```kotlin
    private fun runIterableExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
        val stringList = listOf("1", "2", "3")
    
        compositeDisposable
            .add(
                Observable
                    .fromIterable(stringList)
                    .subscribeOn(Schedulers.io())
                    .subscribe(::timeStampedLog)
            )
    
        timeStampedLog("end task")
    }
    
    // 실행 결과
    실행 후 흐른 시간 = 0 / message = start task
    실행 후 흐른 시간 = 1 / message = end task
    실행 후 흐른 시간 = 2 / message = 1
    실행 후 흐른 시간 = 2 / message = 2
    실행 후 흐른 시간 = 2 / message = 3
    ```
    
    - 데이터를 하나 하나 꺼내어 발행 한 것을 확인 할 수 있습니다.

## 📖 Interval

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%205.png)
    
- 특징
    - 일정한 시간 간격을 두고 0부터 시작하여 1씩 증가하는 숫자를 방출하는 operator 입니다.
    - 별다른 Scheduler 지정 하지 않을 시, Computation에서 데이터 방출 합니다.
- 예시코드
    
    ```kotlin
    private fun runIntervalExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        compositeDisposable
            .add(
                Observable
                    .interval(1000, TimeUnit.MILLISECONDS)
                    .map { it.toString() }
                    .subscribe(::timeStampedLog)
            )
    
        timeStampedLog("end task")
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 0 / message = start task
    thread name = main 실행 후 흐른 시간 = 23 / message = end task
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1024 / message = 0
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 2023 / message = 1
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 3023 / message = 2
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 4023 / message = 3
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 5023 / message = 4
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 6023 / message = 5
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 7023 / message = 6
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 8023 / message = 7
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 9023 / message = 8
    ... 
    ```
    
    - Computation Thread에서 발행 되고 있음을 확인 할 수 있습니다.
    - 0부터 데이터가 하나씩 발행 되고 있음을 확인 할 수 있습니다.
    - map operator는 추후 포스팅에서 다룰 예정이지만, 발행 된 데이터를 변환해주는 operator 입니다. ( [공식문서](http://reactivex.io/documentation/operators/map.html) )
        
        ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%206.png)
        
        - map operator를 활용하여 interval에서 발행된 long 값을 string 형태로 변환 해 주었습니다.
    
    ```kotlin
    private fun runIntervalExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        compositeDisposable
            .add(
                Observable
                    .interval(0, 1000, TimeUnit.MILLISECONDS)
                    .map { it.toString() }
                    .subscribe(::timeStampedLog)
            )
    
        timeStampedLog("end task")
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 0 / message = start task
    thread name = main 실행 후 흐른 시간 = 17 / message = end task
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 17 / message = 0
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1017 / message = 1
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 2016 / message = 2
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 3016 / message = 3
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 4016 / message = 4
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 5016 / message = 5
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 6017 / message = 6
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 7017 / message = 7
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 8017 / message = 8
    ...
    ```
    
    - interval의 첫번째 인자에 initial delay 값을 추가 해 줌으로써 초기 delay 시간을 정할 수 있습니다.
    - 처음 예시 코드와 다르게 첫번째 인자에 0을 넣어줌으로써 초기 delay 없이 바로 실행 된 것을 확인 할 수 있습니다.

## 📖 Timer

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%207.png)
    
- 특징
    - 일정 시간이 지난 뒤, 아이템( `0` )을 방출하고 데이터 방출 완료 이벤트를 방출하는 operator 입니다.
    - 별다른 Scheduler 지정 하지 않을 시, Computation에서 데이터 방출합니다.
- 예시코드
    
    ```kotlin
    private fun runTimerExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        compositeDisposable
            .add(
                Observable
                    .timer(1000, TimeUnit.MILLISECONDS)
                    .map { it.toString() }
                    .subscribe(
                        ::timeStampedLog,
                        { it.printStackTrace() },
                        { timeStampedLog("on Complete !") })
            )
    
        timeStampedLog("end task")
    }
    
    // 실행 결과
    thread name = main 실행 후 흐른 시간 = 0 / message = start task
    thread name = main 실행 후 흐른 시간 = 22 / message = end task
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1022 / message = 0
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1025 / message = on Complete !
    ```
    
    - Timer로 정해 둔 시간이 지나면, 0이 발행되고 `onComplete` 이벤트가 발생 한 것을 확인 할 수 있습니다.
    - Computation Thread에서 발행 되고 있음을 확인 할 수 있습니다.

## 📖 Range

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-25-create-operator/Untitled%208.png)
    
- 특징
    - n부터 시작하고, 1씩 증가하는 숫자를 m개를 방출하는 operator 입니다.
    - for문을 대체 하는데 사용 될 수 있습니다.
- 예시코드
    
    ```kotlin
    private fun runRangeExample() {
        startedTime = System.currentTimeMillis()
        timeStampedLog("start task")
    
        compositeDisposable
            .add(
                Observable
                    .range(0, 10)
                    .map { it.toString() }
                    .subscribe(::timeStampedLog)
            )
    
        timeStampedLog("end task")
    }
    
    // 실행 결과 
    thread name = main 실행 후 흐른 시간 = 0 / message = start task
    thread name = main 실행 후 흐른 시간 = 10 / message = 0
    thread name = main 실행 후 흐른 시간 = 11 / message = 1
    thread name = main 실행 후 흐른 시간 = 11 / message = 2
    thread name = main 실행 후 흐른 시간 = 11 / message = 3
    thread name = main 실행 후 흐른 시간 = 11 / message = 4
    thread name = main 실행 후 흐른 시간 = 11 / message = 5
    thread name = main 실행 후 흐른 시간 = 11 / message = 6
    thread name = main 실행 후 흐른 시간 = 11 / message = 7
    thread name = main 실행 후 흐른 시간 = 12 / message = 8
    thread name = main 실행 후 흐른 시간 = 12 / message = 9
    thread name = main 실행 후 흐른 시간 = 12 / message = end task
    ```
    
    - 0부터 9까지 데이터가 발행 되었음을 확인 할 수 있습니다.