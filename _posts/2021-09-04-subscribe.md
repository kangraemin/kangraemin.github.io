---
title: RxJava 강의 5 - Subscribe operator / Consumer, Action interface
date: 2021-09-04
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava2에서, 관찰 대상에 관찰자를 등록하는 행위인 subscribe와, 이를 편하게 구현하기 위한 Consumer / Action 인터페이스에 대해서 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## Subscribe

- 관찰 객체에 관찰자를 등록하는 행위
- 즉, Stream에 Observer / Subscriber를 등록하여 이벤트 발행 시 각 이벤트별로 어떤 행동을 할 것인지 등록하는 행위
- Stream의 구현체 별로 관찰자의 형태가 다름에 유의

## Consumer / Action Interface

- 관찰자를 개발자가 직접 생성하지 않도록 도와주는 Interface
- Consumer / Action의 구현체를 넘기면, subscribe 내부 동작에서 Stream에 맞는 관찰자를 생성 해 주고 있음

# 📚 Subscribe

## 📖 Subscribe

 Stream에서 발행하는 이벤트를 전달받아 그에 이벤트에 따라 행동을 하기 위해선, 구독자를 관찰 대상에 등록 해야 합니다. 

 이를 위해 RxJava에선 `subscribe`라는 operator가 있으며, 이를 operator를 통해 관찰 대상과 관찰자를 연결 할 수 있습니다. ( [docs](http://reactivex.io/documentation/operators/subscribe.html) )

 앞서 살펴본 [Observer Pattern 포스팅](https://kangraemin.github.io/android/2021/08/16/observer-pattern/)에서는, Custom UI에 Observer interface를 구현하는 방식으로 Observer 구현 객체를 만들었습니다. 

```kotlin
public interface RamsObserver<T> {
    void notifyDataIsChanged(T value);
}
```

```kotlin
package com.example.rxjavalecture.observerpattern.oberserverpattern.customui

import android.content.Context
import android.util.AttributeSet
import androidx.appcompat.widget.AppCompatTextView
import com.example.rxjavalecture.R
import com.example.rxjavalecture.observerpattern.oberserverpattern.RamsObserver
import com.example.rxjavalecture.observerpattern.oberserverpattern.ObserverPatternActivity

class PercentTextView : AppCompatTextView, RamsObserver<Int> {

    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

    init {
				// ObserverPatternActivity에 있는 progressSubject에 구독 신청
        ObserverPatternActivity.progressSubject.subscribe(this)
    }

		// Percent 값이 변경 되었을 때 할 행동을 정의
    override fun notifyDataIsArrived(value: Int?) {
        value?.let {
            text = String.format(context.getString(R.string.first_week_observer_pattern_result_format), it)
        }
    }
}
```

 이렇게 Observer를 직접 만들어도 되지만, 앞선 포스팅 ( [Stream 구현체의 종류 ( Observable / Flowable / Single ... )](https://kangraemin.github.io/android/2021/08/31/stream-implementation/) )을 살펴보면, Stream의 구현체 마다 발행하는 이벤트가 달라 관찰자에 필요한 함수가 다를 수 있음을 살펴 보았습니다. 

 따라서, RxJava2에서는 각 Stream의 관찰자 객체를 개발자가 직접 개발하지 않기 위해, 각 이벤트 마다 할 행동을 정의하여 subscribe operator의 인자로 넘겨 주면, subscribe 함수의 내부적으로 Observer를 생성 해 주고 있습니다. 

 각 이벤트마다 할 행동을 인자로 받기 위해, RxJava2에서 사용하고 있는 Consumer / Action Interface를 알아 본 뒤, Observable 클래스의 subscribe 함수를 다시 살펴 보도록 하겠습니다. 

### 💡 Consumer

```java
/**
 * A functional interface (callback) that accepts a single value.
 * @param <T> the value type
 */
public interface Consumer<T> {
    /**
     * Consume the given value.
     * @param t the value
     * @throws Exception on error
     */
    void accept(T t) throws Exception;
}
```

- 제네릭 타입의 변수를 인자로 받는 accept 함수가 존재 하며, 이 함수는 Exception을 던질 수 있음을 확인 할 수 있습니다.
- accept 함수의 구현을 통해 제네릭 타입의 변수를 인자로 받았을 때, 어떤 행동을 할 지 서술 할 수 있습니다.

### 💡 Action

```java
/**
 * A functional interface similar to Runnable but allows throwing a checked exception.
 */
public interface Action {
    /**
     * Runs the action and optionally throws a checked exception.
     * @throws Exception if the implementation wishes to throw a checked exception
     */
    void run() throws Exception;
}
```

- Consumer와 다르게 제네릭 타입의 변수를 받지 않으며, Exception을 던질 수 있음을 확인 할 수 있습니다.
- run 함수의 구현을 통해 어떤 행동을 할 지 서술 할 수 있습니다.

### 💡 Subscribe in Observable

 Observable 클래스의 subscribe operator의 구현을 확인해보면 아래와 같습니다.

```java
public abstract class Observable<T> implements ObservableSource<T> {
    @SchedulerSupport(SchedulerSupport.NONE)
    public final Disposable subscribe() {
        return subscribe(Functions.emptyConsumer(), Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION, Functions.emptyConsumer());
    }

    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.NONE)
    public final Disposable subscribe(Consumer<? super T> onNext) {
        return subscribe(onNext, Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION, Functions.emptyConsumer());
    }

    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.NONE)
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {
        return subscribe(onNext, onError, Functions.EMPTY_ACTION, Functions.emptyConsumer());
    }

    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.NONE)
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
    Action onComplete) {
        return subscribe(onNext, onError, onComplete, Functions.emptyConsumer());
    }

    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.NONE)
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
    Action onComplete, Consumer<? super Disposable> onSubscribe) {
        ObjectHelper.requireNonNull(onNext, "onNext is null");
        ObjectHelper.requireNonNull(onError, "onError is null");
        ObjectHelper.requireNonNull(onComplete, "onComplete is null");
        ObjectHelper.requireNonNull(onSubscribe, "onSubscribe is null");

        LambdaObserver<T> ls = new LambdaObserver<T>(onNext, onError, onComplete, onSubscribe);

        subscribe(ls);

        return ls;
    }
}
```

 위에서 부터 4번째 함수 까지는 subscribe를 위해 각 이벤트별로 할 행동을 받는 부분이고, 맨 아래에 있는 함수는 각 이벤트별로 할 행동을 받은 뒤, 옵저버를 생성 해 주는 것을 확인 할 수 있습니다.  

 즉, 실제로 Observable ( 관찰 대상 )에 Observer ( 관찰자 )를 등록 시킬 때, Observer ( 관찰자 )를 직접 생성 하지 않고도 이벤트 별로 어떤 행동을 할 지 서술하는 것 만으로도 구독을 시작 할 수 있음을 의미합니다. 

 따라서, 실제로 Observable을 구독 할 때 아래의 형태로 많이 사용합니다. 

```java
// onNext 이벤트를 받았을 때 할 행동을 정의 한 경우
Observable
    .just("54321")
    .subscribe({ data: String ->

    })

// onNext, onError 이벤트를 받았을 때 할 행동을 정의 한 경우
Observable
    .just("54321")
    .subscribe({ data: String ->

    }, { throwable: Throwable ->

    })

// onNext, onError, onComplete 이벤트를 받았을 때 할 행동을 정의 한 경우
Observable
    .just("54321")
    .subscribe({ data: String ->

    }, { throwable: Throwable ->

    }, {
				// onComplete 이벤트가 발생 했을 때 실행 할 코드 
    })
```

 위와 같이, Observable과 다른 이벤트를 방출하는 Single / Completable 등의 Stream도 각 이벤트별 할 행동을 정의하여 subscribe를 진행 할 수 있으니 꼭 한번 확인 해보시길 추천드립니다.