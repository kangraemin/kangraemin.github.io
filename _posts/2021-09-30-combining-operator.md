---
title: RxJava 강의 11 - 합성 연산자 ( merge / concat / concatEager / zip / combineLatest )
date: 2021-09-30
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava에서 여러개의 Stream을 합쳐 하나의 Stream으로 만들어주는 Operator들 ( merge / concat / concatEager / zip / combineLatest )의 특징과 예시 코드들을 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Merge

- 여러개의 스트림을 하나의 스트림으로 합칠 때 사용하는 operator
- 각 스트림에서 이벤트가 발행 된 즉시 합쳐진 스트림에서 이벤트가 발행됨

## 📖 Concat

- 여러개의 스트림을 하나의 스트림으로 합칠때 사용하는 operator
- 스트림을 합칠 때, 첫번째 인자로 넣은 스트림의 발행이 끝나면 두번째 스트림에서 발행된 데이터의 발행이 시작됨
- 즉, 인자로 넣은 스트림들의 순서를 지켜가며 스트림을 합쳐 발행함
- 첫번째 스트림이 onComplete를 호출하지 않는다면, 두번째 스트림에서 데이터 발행은 이루어 지지 않음

## 📖 ConcatEager

- 여러개의 스트림을 하나의 스트림으로 사용하는 operator
- 앞 스트림의 `onComplete` 이벤트가 호출되기 전 뒤 스트림을 미리 구독하여 데이터를 발행 해 놓음
- 앞 스트림의 `onComplete` 이벤트가 호출되면, 뒤 스트림에서 이미 발행되었던 데이터는 한번에 방출되고, 앞으로 발행할 데이터가 남아 있다면 추가적으로 발행

## 📖 Zip

- 여러개의 스트림을 하나로 합치는데, 각 스트림에서 발행된 데이터들을 조합하여 새로운 데이터를 만들어 발행하는 operator
- 여러개의 스트림에서 발행된 데이터들의 순서가 짝이 맞아야 데이터가 발행됨

## 📖 CombineLatest

- zip과 비슷하게 여러개의 스트림을 하나로 합치는데, 각 스트림에서 발행된 데이터들을 조합하여 새로운 데이터를 만들어 발행하는 operator
- zip은 여러개의 스트림에서 발행된 데이터들의 순서가 짝이 맞아야 데이터가 발행되었지만, `combineLatest`는 짝이 맞지 않더라도 다른 스트림에서 이전에 발행 되었던 데이터를 활용하여 데이터가 발행됨

# 📚 합성 연산자

## 📖 합성 연산자 개요

 Stream과 다른 Stream을 합쳐 새로운 Stream을 만들 때 사용하는 Operator 들을 알아보고자 합니다. 

 예시 코드를 통해 각 operator의 특징들을 살펴 볼 예정이며 아래에 쓰인 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/blob/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/operator/combining/CombiningExampleActivity.kt)에서 확인 하실 수 있습니다. 

 또 아래의 예시 코드에서 공통적으로 쓰인 코드는 아래와 같으니 참고하여 코드 확인하시면 되겠습니다. 

```kotlin
// 예시 코드를 실행한 시간을 나타내기 위한 변수
// 예시 코드를 실행하면, 해당 변수가 실행한 시각으로 변경됩니다. 
private var startedTime = 0L

// 예시 코드가 실행된 Thread name과 실행 한 시각과 로그가 찍힌 시각의 차이를 표기하기 위한 로그입니다.  
private fun timeStampedLog(message: Any) {
    Log.d(
        TAG_TRANSFORM_OPERATOR,
        "thread name = ${Thread.currentThread().name} 실행 후 흐른 시간 = ${System.currentTimeMillis() - startedTime} / message = $message"
    )
}
```

## 📖 Merge

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-30-combining-operator/Untitled.png)
    
    - 첫번째 스트림에서, 20, 40, 60, 80, 100을 발행합니다.
    - 두번째 스트림에서 1, 3을 발행합니다.
    - merge 결과 스트림은 첫번째 스트림, 두번째 스트림에서 발행된 아이템들이 각 스트림에서 발행된 즉시 발행되고 있는 것을 확인 할 수 있습니다.
    - 즉 merge operator는, 스트림에서 발행되는 아이템들을 하나의 스트림으로 단순히 합쳐 주는 역할을 진행하고 있음을 확인 할 수 있습니다.
- 특징
    - 여러개의 스트림을 하나의 스트림으로 합칠 때 사용하는 operator 입니다.
    - 각 스트림에서 이벤트가 발행 된 즉시 합쳐진 스트림에서 이벤트가 발행됩니다.
- 예시 코드
    
    ```kotlin
    // 1초마다 0, 1, 2, ..., 9 까지 숫자 데이터를 발행하는 observable
    private val integerInterval = Observable
        .interval(1000, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
    
    // 1.5초마다 "0 번째 String 데이터", "1 번째 String 데이터", ..., "9 번째 String 데이터" 까지
    // String 데이터를 발행하는 observable
    private val stringInterval = Observable
        .interval(1500, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
        .doOnNext { timeStampedLog("String 데이터 $it 발행 되었음 !!") }
        .map { "$it 번쨰 String 데이터" }
    
    private fun runMergeExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable
                    .merge(
                        integerInterval,
                        stringInterval
                    )
                    .subscribe({
                        timeStampedLog(it)
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 1003 / message = 0
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 1504 / message = String 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 1505 / message = 0 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 2003 / message = 1
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 3003 / message = 2
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 3004 / message = String 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 3004 / message = 1 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 4003 / message = 3
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 4504 / message = String 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 4504 / message = 2 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 5003 / message = 4
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 6003 / message = 5
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 6004 / message = String 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 6004 / message = 3 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 7003 / message = 6
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 7504 / message = String 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 7504 / message = 4 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 8003 / message = 7
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 9003 / message = 8
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 9004 / message = String 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 9004 / message = 5 번쨰 String 데이터
    thread name = RxComputationThreadPool-5 실행 후 흐른 시간 = 10003 / message = 9
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 10504 / message = String 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 10504 / message = 6 번쨰 String 데이터
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 12004 / message = String 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 12004 / message = 7 번쨰 String 데이터
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 13504 / message = String 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 13504 / message = 8 번쨰 String 데이터
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 15004 / message = String 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-6 실행 후 흐른 시간 = 15004 / message = 9 번쨰 String 데이터
    ```
    
    - 첫번째 스트림인 `integerInterval` 스트림과, 두번째 스트림인 `stringInterval` 스트림이 병렬적으로 실행되고 있음을 확인 할 수 있습니다.
    - 두 스트림에서 발행되는 데이터를 하나의 스트림의 결과값으로 단순히 합쳐서 발행 되고 있음을 확인 할 수 있습니다.

## 📖 Concat

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-30-combining-operator/Untitled%201.png)
    
    - 첫번째 스트림에선 1, 1, 1 데이터를 방출하고 `onComplete` 이벤트가 호출됩니다.
    - 두번째 스트림에선 2, 2 데이터를 방출하고 `onComplete` 이벤트가 호출됩니다.
    - concat 결과 스트림에선 첫번째 스트림의 데이터 방출이 끝난 뒤, 두번째 스트림의 데이터 방출이 이루어 짐을 확인 할 수 있습니다.
- 특징
    - 여러개의 스트림을 하나의 스트림으로 합칠때 사용하는 operator 입니다.
    - 스트림을 합칠 때, 첫번째 인자로 넣은 스트림의 발행이 끝나면 두번째 스트림에서 발행된 데이터의 발행이 시작됩니다.
    - 즉, 인자로 넣은 스트림들의 순서를 지켜가며 스트림을 합쳐 발행합니다.
    - 첫번째 스트림이 onComplete를 호출하지 않는다면, 두번째 스트림에서 데이터 발행은 이루어 지지 않습니다.
    - 따라서 concat을 활용하여 스트림을 여러개 합칠 때, 앞 stream의 데이터 발행이 끝났다면 앞 stream에서 `onComplete` 이벤트를 반드시 호출 해 주어야 뒤 stream에서 데이터 발행이 시작 되는것에 유의 하여야 합니다.
- 예시 코드
    
    ```kotlin
    // 1초마다 0, 1, 2, ..., 9 까지 숫자 데이터를 발행하는 observable
    private val integerInterval = Observable
        .interval(1000, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
    
    // 1.5초마다 "0 번째 String 데이터", "1 번째 String 데이터", ..., "9 번째 String 데이터" 까지
    // String 데이터를 발행하는 observable
    private val stringInterval = Observable
        .interval(1500, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
        .doOnNext { timeStampedLog("String 데이터 $it 발행 되었음 !!") }
        .map { "$it 번쨰 String 데이터" }
    
    private fun runConcatExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable
                    .concat(
                        integerInterval,
                        stringInterval
                    )
                    .subscribe({
                        timeStampedLog(it)
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 1005 / message = 0
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 2005 / message = 1
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3005 / message = 2
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 4005 / message = 3
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 5005 / message = 4
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 6005 / message = 5
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 7005 / message = 6
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 8004 / message = 7
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 9005 / message = 8
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 10005 / message = 9
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 12506 / message = String 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 12507 / message = 0 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 14006 / message = String 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 14006 / message = 1 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 15506 / message = String 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 15506 / message = 2 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 17006 / message = String 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 17006 / message = 3 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 18506 / message = String 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 18506 / message = 4 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 20006 / message = String 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 20007 / message = 5 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 21506 / message = String 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 21506 / message = 6 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 23006 / message = String 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 23006 / message = 7 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 24506 / message = String 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 24506 / message = 8 번쨰 String 데이터
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 26006 / message = String 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 26006 / message = 9 번쨰 String 데이터
    ```
    
    - 첫번째 스트림인 `integerInterval` 스트림의 데이터 발행이 끝난 뒤, 두번째 스트림인 `stringInterval` 스트림의 데이터 발행이 시작 되었음을 확인 할 수 있습니다.

## 📖 ConcatEager

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-30-combining-operator/Untitled%202.png)
    
    - 첫번째 스트림에선 빨간색, 초록색, 파란색 공 데이터가 발행되고, `onComplete` 이벤트가 발행 됩니다.
    - 두번째 스트림에선 노란색, 하늘색, 보라색 공 데이터가 발행되고, `onComplete` 이벤트가 발행 됩니다.
    - concatEager로 합친 결과 스트림에선, 첫번째 스트림에서 발행된 빨간색, 초록색, 파란색 공이 발행되고 노란색, 하늘색, 보라색 공 데이터가 발행됩니다.
    - concat 에서는 첫번째 스트림의 `onComplete` 이벤트가 호출 된 뒤에, 두번째 스트림을 구독하여 데이터 발행이 이루어 집니다.
    - concatEager 에서는 첫번째 스트림의 `onComplete` 이벤트가 호출되기 이전에도 두번째 스트림을 구독하여 데이터를 미리 발행 해 놓고, 첫번째 스트림의 `onComplete` 이벤트가 호출 되면 두번째 스트림에서 발행 되었던 데이터들을 방출 합니다.
- 특징
    - concat과 동일한 목적을 가진 operator 입니다.
    - concat과 차이점은 concat은 앞 스트림의 `onComplete` 이벤트가 호출 되기 전에는 뒤 스트림을 구독하지 않으나, concatEager는 앞 스트림의 `onComplete` 이벤트가 호출되기 전 뒤 스트림을 미리 구독하여 데이터를 발행 해 놓습니다.
    - 앞 스트림의 `onComplete` 이벤트가 호출되면, 뒤 스트림에서 이미 발행되었던 데이터는 한번에 방출되고, 앞으로 발행할 데이터가 남아 있다면 추가적으로 발행합니다.
- 예시코드
    
    ```kotlin
    // 1초마다 0, 1, 2, ..., 9 까지 숫자 데이터를 발행하는 observable
    private val integerInterval = Observable
        .interval(1000, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
    
    // 1.5초마다 "0 번째 String 데이터", "1 번째 String 데이터", ..., "9 번째 String 데이터" 까지
    // String 데이터를 발행하는 observable
    private val stringInterval = Observable
        .interval(1500, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
        .doOnNext { timeStampedLog("String 데이터 $it 발행 되었음 !!") }
        .map { "$it 번쨰 String 데이터" }
    
    private fun runConcatEagerExample() {
        startedTime = System.currentTimeMillis()
        val observableList = listOf(integerInterval, stringInterval)
        compositeDisposable
            .add(
                Observable
                    .concatEager(observableList)
                    .subscribe({
                        timeStampedLog(it)
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1059 / message = 0
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 1560 / message = String 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 2059 / message = 1
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 3059 / message = 2
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 3060 / message = String 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 4059 / message = 3
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 4560 / message = String 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 5059 / message = 4
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 6059 / message = 5
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 6060 / message = String 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 7059 / message = 6
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 7560 / message = String 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 8059 / message = 7
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 9059 / message = 8
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 9060 / message = String 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 10059 / message = 9
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 10560 / message = String 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11059 / message = 0 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11059 / message = 1 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11060 / message = 2 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11061 / message = 3 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11061 / message = 4 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11062 / message = 5 번쨰 String 데이터
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11062 / message = 6 번쨰 String 데이터
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 12060 / message = String 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 12061 / message = 7 번쨰 String 데이터
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 13560 / message = String 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 13561 / message = 8 번쨰 String 데이터
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 15060 / message = String 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 15061 / message = 9 번쨰 String 데이터
    ```
    
    - 첫번째 스트림인 `integerInterval` 스트림이 발행이 완료되기 전, `stringInterval` 스트림의 데이터 발행이 시작 된 것을 확인 할 수 있습니다.
    - 첫번째 스트림인 `integerInterval` 스트림의 발행이 완료되면, 두번째 스트림인 `stringInterval` 에서 발행 해 두었던 데이터가 한번에 방출 되고, 남은 데이터를 추가적으로 방출하고 있음을 확인 할 수 있습니다.

## 📖 Zip

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-30-combining-operator/Untitled%203.png)
    
    - 첫번째 스트림에선 1, 2, 3, 4, 5 데이터를 방출하고 있습니다.
    - 두번째 스트림에선 A, B, C, D 데이터를 방출하고 있습니다.
    - zip의 결과인 세번째 스트림은 다음과 같은 규칙이 있습니다.
        - 첫번째 스트림에서 발행한 첫번쨰 아이템인 `1`과 두번쨰 스트림에서 발행한 첫번째 아이템인 `A`를 합쳐 `1A`를 발행 하였습니다.
        - 첫번째 스트림에서 발행한 두번째 아이템인 `2`와 두번쨰 스트림에서 발행한 두번째 아이템인 `B`를 합쳐 `2B`를 발행 하였습니다.
        - 첫번째 스트림에서 발행한 세번쨰 아이템인 `3`과 두번쨰 스트림에서 발행한 세번째 아이템인 `C`를 합쳐 `3C`를 발행 하였습니다.
        - 첫번째 스트림에서 발행한 다섯번째 아이템인 `5`는 두번째 스트림에서 짝을 찾을 수 없어 세번째 스트림에서 발행이 되지 않았습니다.
- 특징
    - 여러개의 스트림을 하나로 합치는데, 각 스트림에서 발행된 데이터들을 조합하여 새로운 데이터를 만들어 발행하는 operator 입니다.
    - 여러개의 스트림에서 발행된 데이터들의 순서가 짝이 맞아야 데이터가 발행됩니다.
        - 예를들어, 첫번째 스트림에서 발행된 첫번째 아이템은 두번째 스트림에서 첫번째 아이템과 결합되어 방출됩니다.
        - 첫번쨰 스트림에서 두번째 아이템을 발행 하였지만, 두번째 스트림에서 두번째 아이템을 발행하지 않는다면 결합 할 짝이 없으므로 방출되지 않습니다.
- 예시 코드
    
    ```kotlin
    private val integerInterval = Observable
        .interval(1000, TimeUnit.MILLISECONDS)
        .doOnNext { timeStampedLog("integer 데이터 $it 발행 되었음 !!") }
        .takeWhile { it < 10 }
    
    private val stringInterval = Observable
        .interval(1500, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
        .doOnNext { timeStampedLog("String 데이터 $it 발행 되었음 !!") }
        .map { "$it 번쨰 String 데이터" }
    
    private fun runZipExample() {
        startedTime = System.currentTimeMillis()
        compositeDisposable
            .add(
                Observable
                    .zip(
                        integerInterval,
                        stringInterval,
                        BiFunction { integerResult: Long, stringResult: String ->
                            return@BiFunction String.format(
                                "String 데이터는 %s 이고 / integer 데이터는 %d 번째 데이터가 발행 됨",
                                stringResult,
                                integerResult
                            )
                        }
                    )
                    .subscribe({
                        timeStampedLog(it)
                    }, { it.printStackTrace() })
            )
    }
    
    // 실행 결과
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 1053 / message = integer 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 1554 / message = String 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 1560 / message = String 데이터는 0 번쨰 String 데이터 이고 / integer 데이터는 0 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 2052 / message = integer 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 3053 / message = integer 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 3054 / message = String 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 3057 / message = String 데이터는 1 번쨰 String 데이터 이고 / integer 데이터는 1 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 4053 / message = integer 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 4554 / message = String 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 4557 / message = String 데이터는 2 번쨰 String 데이터 이고 / integer 데이터는 2 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 5053 / message = integer 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 6053 / message = integer 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 6054 / message = String 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 6057 / message = String 데이터는 3 번쨰 String 데이터 이고 / integer 데이터는 3 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 7053 / message = integer 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 7554 / message = String 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 7557 / message = String 데이터는 4 번쨰 String 데이터 이고 / integer 데이터는 4 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 8053 / message = integer 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 9053 / message = integer 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 9054 / message = String 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 9057 / message = String 데이터는 5 번쨰 String 데이터 이고 / integer 데이터는 5 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 10053 / message = integer 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 10554 / message = String 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 10557 / message = String 데이터는 6 번쨰 String 데이터 이고 / integer 데이터는 6 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-1 실행 후 흐른 시간 = 11052 / message = integer 데이터 10 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 12054 / message = String 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 12057 / message = String 데이터는 7 번쨰 String 데이터 이고 / integer 데이터는 7 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 13554 / message = String 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 13557 / message = String 데이터는 8 번쨰 String 데이터 이고 / integer 데이터는 8 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 15054 / message = String 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-2 실행 후 흐른 시간 = 15057 / message = String 데이터는 9 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    ```
    
    - BiFunction 함수를 활용하여 첫번째 스트림에서 발행하는 integerData와 두번째 스트림에서 발행하는 stringData를 합쳐 새로운 String 데이터로 만들어 주고 있습니다.
    - 첫번째 스트림의 데이터가 발행되고, 두번째 스트림의 데이터가 발행 되었을 때 짝이 맞아야 subscribe 안으로, 즉 구독자에게 데이터가 발행되는것을 확인 할 수 있습니다.

## 📖 CombineLatest

- Marble Diagram
    
    ![Untitled](/assets/images/posts/2021-09-30-combining-operator/Untitled%204.png)
    
    - 첫번째 스트림에선 1, 2, 3, 4, 5를 방출하고 있습니다.
    - 두번째 스트림에선 A, B, C, D를 방출하고 있습니다.
    - combineLatest의 결과인 세번째 스트림은 다음과 같은 규칙이 있습니다.
        - 첫번째 스트림에서 `1`, 두번째 스트림에서 `A`가 발행 되었을 때, 그 둘을 합쳐 `1A` 가 발행되었습니다.
        - 첫번째 스트림에서 `2` 가 발행되었을 때, 두번째 스트림에서 이전에 발행했던 값인 `A` 와 결합하여 `2A` 가 발행되었습니다.
        - 두번째 스트림에서 `B`가 발행되었을 때, 첫번째 스트림에서 이전에 발행했던 값인 `2` 와 결합하여 `2B` 가 발행되었습니다.
- 특징
    - zip과 비슷하게 여러개의 스트림을 하나로 합치는데, 각 스트림에서 발행된 데이터들을 조합하여 새로운 데이터를 만들어 발행하는 operator 입니다.
    - zip은 여러개의 스트림에서 발행된 데이터들의 순서가 짝이 맞아야 데이터가 발행되었지만, `combineLatest`는 짝이 맞지 않더라도 다른 스트림에서 이전에 발행 되었던 데이터를 활용하여 데이터가 발행됩니다.
        - 예를들어, 첫번째 스트림에서 발행된 첫번째 아이템은 두번째 스트림에서 첫번째 아이템과 결합되어 방출됩니다.
        - 첫번쨰 스트림에서 두번째 아이템을 발행 하였지만, 두번째 스트림에서 두번째 아이템을 발행하지 않은 경우엔 zip은 데이터 발행이 되지 않았지만, combineLatest 에서는 첫번째 스트림의 두번째 아이템과 두번쨰 스트림의 첫번째 아이템을 결합하여 데이터를 방출합니다.
- 예시 코드
    
    ```kotlin
    private val integerInterval = Observable
        .interval(1000, TimeUnit.MILLISECONDS)
        .doOnNext { timeStampedLog("integer 데이터 $it 발행 되었음 !!") }
        .takeWhile { it < 10 }
    
    private val stringInterval = Observable
        .interval(1500, TimeUnit.MILLISECONDS)
        .takeWhile { it < 10 }
        .doOnNext { timeStampedLog("String 데이터 $it 발행 되었음 !!") }
        .map { "$it 번쨰 String 데이터" }
    
    private fun runCombineLatestExample() {
       startedTime = System.currentTimeMillis()
       compositeDisposable
           .add(
               Observable
                   .combineLatest(
                       integerInterval,
                       stringInterval,
                       BiFunction { integerResult: Long, stringResult: String ->
                           return@BiFunction String.format(
                               "String 데이터는 %s 이고 / integer 데이터는 %d 번째 데이터가 발행 됨",
                               stringResult,
                               integerResult
                           )
                       }
                   )
                   .subscribe({
                       timeStampedLog(it)
                   }, { it.printStackTrace() })
           )
    
    // 실행 결과
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 1004 / message = integer 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 1505 / message = String 데이터 0 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 1508 / message = String 데이터는 0 번쨰 String 데이터 이고 / integer 데이터는 0 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 2004 / message = integer 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 2007 / message = String 데이터는 0 번쨰 String 데이터 이고 / integer 데이터는 1 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3004 / message = integer 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 3004 / message = String 데이터 1 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3007 / message = String 데이터는 0 번쨰 String 데이터 이고 / integer 데이터는 2 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 3009 / message = String 데이터는 1 번쨰 String 데이터 이고 / integer 데이터는 2 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 4004 / message = integer 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 4007 / message = String 데이터는 1 번쨰 String 데이터 이고 / integer 데이터는 3 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 4504 / message = String 데이터 2 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 4507 / message = String 데이터는 2 번쨰 String 데이터 이고 / integer 데이터는 3 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 5004 / message = integer 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 5007 / message = String 데이터는 2 번쨰 String 데이터 이고 / integer 데이터는 4 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 6004 / message = integer 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 6004 / message = String 데이터 3 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 6007 / message = String 데이터는 2 번쨰 String 데이터 이고 / integer 데이터는 5 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 6009 / message = String 데이터는 3 번쨰 String 데이터 이고 / integer 데이터는 5 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 7004 / message = integer 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 7006 / message = String 데이터는 3 번쨰 String 데이터 이고 / integer 데이터는 6 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 7504 / message = String 데이터 4 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 7507 / message = String 데이터는 4 번쨰 String 데이터 이고 / integer 데이터는 6 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 8004 / message = integer 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 8007 / message = String 데이터는 4 번쨰 String 데이터 이고 / integer 데이터는 7 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 9004 / message = integer 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 9004 / message = String 데이터 5 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 9007 / message = String 데이터는 4 번쨰 String 데이터 이고 / integer 데이터는 8 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 9009 / message = String 데이터는 5 번쨰 String 데이터 이고 / integer 데이터는 8 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 10004 / message = integer 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 10006 / message = String 데이터는 5 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 10504 / message = String 데이터 6 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 10507 / message = String 데이터는 6 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-3 실행 후 흐른 시간 = 11004 / message = integer 데이터 10 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 12004 / message = String 데이터 7 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 12007 / message = String 데이터는 7 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 13504 / message = String 데이터 8 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 13507 / message = String 데이터는 8 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 15004 / message = String 데이터 9 발행 되었음 !!
    thread name = RxComputationThreadPool-4 실행 후 흐른 시간 = 15007 / message = String 데이터는 9 번쨰 String 데이터 이고 / integer 데이터는 9 번째 데이터가 발행 됨
    ```
    
    - zip과 마찬가지로 BiFunction 함수를 활용하여 첫번째 스트림에서 발행하는 integerData와 두번째 스트림에서 발행하는 stringData를 합쳐 새로운 String 데이터로 만들어 주고 있습니다.
    - 첫번째 스트림의 데이터가 발행되고, 두번째 스트림의 데이터가 발행 되었을 때 짝이 맞지 않더라도, 이전의 값을 활용하여 데이터가 구독자에게 전달되고 있음을 확인 할 수 있습니다.