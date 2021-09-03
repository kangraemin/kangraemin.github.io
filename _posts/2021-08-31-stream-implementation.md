---
title: RxJava 강의 4 - Observable이란, Stream 구현체의 종류, Disposable 이란
date: 2021-08-31
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

RxJava2의 Observable이 무엇인지, Stream의 구현체의 종류는 어떤 것이 있는지, Dispsable 객체는 무엇인지에 대해서 알아봅니다.

<!-- more -->

# 📚 TL; DR

## 📖 General

- RxJava2 에서는, Stream의 구현체로 여러 종류를 만들어 놓았다.
- RxJava2 속의 Stream의 구현체로는 Observable / Single / Flowable / Completable / Maybe 가 있다.
- 각 Stream 구현체마다 용도별로 방출하는 event가 다르며, 용도에 맞게 사용하면 된다.

## 📖 배압

- 관찰 대상이 이벤트를 발행하는 속도가 구독자가 이벤트를 처리하는 속도보다 **현저히** 빠를 경우, 관찰 대상이 발행하는 이벤트를 구독자가 제대로 처리하지 못하는 이슈
- 이에 대한 내용은, 추후 multi threading 포스팅 이후에 다시 다룰 예정

## 📖 Observable

- Stream에서 item을 1개 이상 발행 할 수 있는 Stream의 구현체
- Android에서는, Flowable 대신에 Observable을 사용하는 경우가 잦음 ( Docs : [link](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0#when-to-use-observable) )

## 📖 Flowable

- Stream에서 item을 1개 이상 발행 할 수 있는 Stream의 구현체
- 배압 이슈가 발생 했을 때, 배압 이슈를 어떻게 처리 할 것인지에 대한 전략을 미리 선언 해 놓을 수 있음
- 대량의 I / O 작업이 있는 경우, 비동기 작업을 대량으로 처리 해야 하는 경우에 사용 ( Docs : [link](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0#when-to-use-flowable) )

## 📖 Single

- Stream에서 item을 하나만 발행 할 수 있는 Stream의 구현체 ( 단발성 이벤트 )
- Network 통신, DB 작업 등 작업이 단발성으로 진행 될 때 주로 사용

## 📖 Completable

- Stream에서 item을 발행 할 수 없고, 데이터 발행 여부가 성공적이었는지 아닌지를 나타내는 이벤트만 발행 할 수 있는 Stream의 구현체
- 데이터 발행 도중 발생한 에러가 없다면 onComplete, 에러가 발생 했다면 onError를 발생 시킴
- 데이터 발행 여부만 중요하고, 결과 데이터는 중요하지 않은 경우에 주로 사용

## 📖 Maybe

- Stream에서 item을 하나만 발행 할 수 있지만, item을 발행 하지 않고 데이터 발행 여부가 성공적이었는지 아닌지를 나타내는 이벤트만 발행 할 수도 있는 Stream의 구현체 ( 단발성 이벤트 )
- 즉, 데이터를 하나만 발행 할 수도 있고 데이터를 하나도 발행하지 않고 데이터 발행 여부만 발행 할 수도 있음
- Single과 Completable을 조합한 형태며, SingleObserver와 다르게 onComplete가 존재

## 📖 Disposable

- Stream을 observer가 구독하면, 구독을 취소할 수 있는 상태가 되는데 이 상태를 표현한 클래스
- 즉, 구독 취소를 할 수 있는 객체 ( Dispose를 할 수 있는 객체 )
- 효율적으로 컴퓨터 자원을 사용하기 위해, 더이상 진행 될 필요가 없는 / 더이상 구독 할 필요가 없는 구독 관계를 끊어줄 때 주로 사용
- compositeDisposable 객체에 담아 주로 사용

# 📚 Stream의 구현체 Observable

## 📖 Stream의 구현체

 [지난 포스팅](https://kangraemin.github.io/android/2021/08/23/reactive-stream-marble-diagram/)에서, Stream은 데이터가 변화되었을 때 관찰자들에게 이를 알려 줄 뿐 아니라, 데이터 변화 도중 생긴 이벤트들을 관찰자들에게 알려주는 역할도 맡는 객체라고 하였습니다. 

 RxJava2에서는 이러한 Stream의 구체적인 구현체로써 용도에 맞게 사용 할 수 있도록 여러가지를 미리 구현 해 놓았습니다. 

 RxJava2에는 Stream의 구현체로써 구현 해 놓은 Observable / Flowable / Single / Completable / Maybe클래스가 있으며, 각 Strem마다 발행되는 이벤트 또는, 이벤트를 처리하는 방식이 다르게 구현되어 있어 해당 Stream을 구독하는 observer들도 각 Stream에 맞도록 구현 되어 있습니다. 

 따라서 RxJava2에 존재하는 각 Stream 구현체들의 특성과, 해당 Stream 구현체들의 용도를 이번 포스팅에서 살펴보고자 합니다.

## 📖 배압 이슈에 대해서

 Stream의 구현체로 들어가기 전에, 배압 이슈가 어떤 것인지를 잠깐 다루고 가려 합니다. 

 배압 이슈가 어떤 것인지, 또 어떻게 다루는지에 대해 자세한 내용은 추후에 다룰 `observeOn` / `subscribeOn` 등의 함수를 다루는 thread handling 포스팅 이후에 다시 설명 드리도록 하겠습니다. 

 배압이슈가 어떤 것인지 대략적으로 설명드리면, 관찰 대상이 이벤트를 발행하는 속도가 구독자가 이벤트를 처리하는 속도보다 **현저히** 빠를 경우, 관찰 대상이 발행하는 이벤트를 구독자가 제대로 처리하지 못하는 이슈가 발생 할 수 있습니다. 

 현실세계의 예를 들어보겠습니다. 

- 병원에서 진료를 보는데 환자마다 걸리는 시간은 10분입니다.
- 환자가 1분에 한명씩 계속해서 병원에 들어옵니다.
- 위 상황이 지속되면, 병원이 환자로 가득 차 더이상 환자가 들어올 수 없는 현상이 발생 할 수 있습니다.

 위 상황을 Stream과 Stream을 구독하는 구독자에 빗대어 보겠습니다.

- Stream에서 이벤트를 방출하는데, 방출된 이벤트를 처리하는데 소요되는 시간은 약 10분입니다.
- 이벤트가 1분에 하나씩 계속해서 방출됩니다.
- 위 상황이 지속되면, 방출된 이벤트 중 처리되지 못한 이벤트를 담아놓는 공간 ( Buffer )이 가득 차버려 더이상 이벤트를 발행하지 못하는 상황이 발생 할 수 있습니다. 이는 out of memoery error 를 발생 시킬 수 있습니다.

 위와 같은 상황이 배압에 관련된 이슈이며, 이는 추후에 자세히 다룰 예정이므로 이러한 것이 있다는 것만 인지하고 포스팅을 계속 읽어 주시면 감사하겠습니다.

## 📖 Observable

### Observable의 특성

- Item을 여러개 방출 할 수 있습니다.
- Item의 발행 완료 이벤트를 방출 할 수 있습니다.
- Item의 발행 도중 에러가 발생했다면, 에러 이벤트를 방출 할 수 있습니다.

### Observable을 구독하는 관찰자의 형태

```java
public interface Observer<T> {
    void onSubscribe(@NonNull Disposable d);
    void onNext(@NonNull T t);
    void onError(@NonNull Throwable e);
    void onComplete();
}
```

- Observable에서는 Observer의 `onNext`함수를 통해 데이터를 발행합니다.
- Observable에서는 Observer의 `onError`함수를 통해 데이터 발행 도중 에러가 발생했음을 알립니다.
- Observable에서는 Observer의 `onComplete`함수를 통해 데이터 발행이 완료되었음을 알립니다.

### Observable을 주로 사용하는 용도

- 데이터가 여러번 발행 될 수 있는 곳에서 주로 사용합니다.
- 데이터가 1000번 미만으로 발행되는 경우에 사용하는것이 좋습니다.
- Out of Memory Error가 발생되지 않을 가능성이 높은 곳에서 사용하는것이 좋습니다.
- 터치 이벤트, 키보드 입력과 같은 GUI 이벤트를 받을 때 주로 사용합니다.
    - 하지만 1000 Hz의 주파수 이벤트를 받는 곳이라면, Observable을 사용하는 것을 다시 고려할 필요가 있으나, `debounce` / `sampling` 의 방식으로 해결 할 수 있습니다.
- Flowable 보다 overhead가 작으니, Flowable을 사용해야 하는 상황이 아닌 경우엔 대부분 Observable을 사용합니다.

## 📖 Flowable

### Flowable의 특성

- Item을 여러개 방출 할 수 있습니다.
- Item의 발행 완료 이벤트를 방출 할 수 있습니다.
- Item의 발행 도중 에러가 발생했다면, 에러 이벤트를 방출 할 수 있습니다.
- 배압 이슈를 처리 하는 전략을 정할 수 있습니다.

### Flowable을 구독하는 관찰자의 형태

```java
public interface Subscriber<T> {
		public void onSubscribe(Subscription s);
		public void onNext(T t);
		public void onError(Throwable t);
		public void onComplete();
}

public interface FlowableSubscriber<T> extends Subscriber<T> {
    @Override
    void onSubscribe(@NonNull Subscription s);
}
```

- Flowable에서는 Subscriber의 `onNext`함수를 통해 데이터를 발행합니다.
- Flowable에서는 Subscriber의 `onError`함수를 통해 데이터 발행 도중 에러가 발생했음을 알립니다.
- Flowable에서는 Subscriber의 `onComplete`함수를 통해 데이터 발행이 완료되었음을 알립니다.

### Flowable을 주로 사용하는 용도

- 데이터가 여러번 발행 될 수 있는 곳에서 주로 사용합니다.
- 데이터가 10000번 이상으로 발행되는 경우에 사용을 고려 해 볼 수 있습니다.
- 구독하는 곳에서 오래 걸리는 작업 ( 디스크에서 파일을 읽거나, 로컬 DB에서 쿼리를 진행하는 등 )을 진행중에 있는데, 데이터 발행이 데이터를 소비하는 속도보다 빠른 속도로 진행되는 경우에 사용합니다.

## 📖 Single

### Single의 특성

- Item을 하나만 발행 할 수 있으며, 발행 완료 이벤트와 함께 Item이 방출됩니다.
- Item의 발행 도중 에러가 발생했다면, 에러 이벤트를 방출 할 수 있습니다.

### Single을 구독하는 관찰자의 형태

```java
public interface SingleObserver<T> {
		void onSubscribe(@NonNull Disposable d);
		void onSuccess(@NonNull T t);
		void onError(@NonNull Throwable e);
}
```

- Single에서는 SingleObserver의 `onSuccess`함수를 통해 발행 완료 이벤트와 함께 데이터를 방출합니다.
- Single에서는 SingleObserver의 `onError`함수를 통해 데이터 발행 도중 에러가 발생했음을 알립니다.

### Single을 주로 사용하는 용도

- 데이터가 한번만 발행 되면 되는 곳에서 주로 사용합니다.
- 대부분의 API 통신은, request / response의 과정을 딱 한번 거치게 되는 경우가 많기 때문에 주로 API 통신을 진행할 때 주로 사용합니다.
- 한번만 실행되는 로컬 DB의 쿼리 작업에 주로 사용합니다.

## 📖 Completable

### Completable의 특성

- Item을 발행하는 작업을 진행 할 수 있지만, Item은 발행하지 않으며, 발행 완료 이벤트만 방출 할 수 있습니다.
- Item의 발행 도중 에러가 발생 했다면, 에러 이벤트를 방출 할 수 있습니다.

### Completable을 구독하는 관찰자의 형태

```java
public interface CompletableObserver {
    void onSubscribe(@NonNull Disposable d);
    void onComplete();
    void onError(@NonNull Throwable e);
}
```

- Completable에서는 CompletableObserver의 `onComplete`함수를 통해 데이터 발행이 완료되었음을 알립니다.
- Completable에서는 CompletableObserver의 `onError`함수를 통해 데이터 발행 도중 에러가 발생했음을 알립니다.

### Completable을 주로 사용하는 용도

- 함수의 결과 값은 중요하지 않고, 실행 완료 여부만 중요할 때 주로 사용합니다.
- 서버와 통신 할 때, 서버에서 온 값을 어디에서도 사용하지 않을 때 통신의 성공 / 실패 여부를 확인하기 위해 사용합니다.

## 📖 Maybe

### Maybe의 특성

- Item을 발행 완료 이벤트와 함께 하나만 발행 하거나, Item을 발행 하지 않고 데이터 발행이 완료되었음을 알리는 이벤트만 발행 할 수 있습니다.
- Item의 발행 도중 에러가 발생했다면, 에러 이벤트를 방출 할 수 있습니다.

### Maybe을 구독하는 관찰자의 형태

```java
public interface MaybeObserver<T> {
    void onSubscribe(@NonNull Disposable d);
    void onSuccess(@NonNull T t);
    void onError(@NonNull Throwable e);
    void onComplete();
}
```

- MaybeObserver에서는 MaybeObserver의 `onSuccess`함수를 통해 데이터를 발행합니다.
- MaybeObserver에서는 MaybeObserver의 `onError`함수를 통해 데이터 발행 도중 에러가 발생했음을 알립니다.
- MaybeObserver에서는 MaybeObserver의 `onComplete`함수를 통해 데이터 발행이 완료되었음을 알립니다.

### Maybe을 주로 사용하는 용도

- 방출되는 데이터가 nullable한 경우에 사용합니다.
- 즉 데이터가 있을수도 있고 없을수도 있는 경우에 사용하면 데이터가 없더라도 `onComplete` 함수가 호출 되기 때문에, 발행이 끝났음을 알 수 있습니다.
- 데이터로 null이 방출되면, `onComplete` 함수가 호출되며 null이 아닌 데이터가 방출되면 `onSuccess` 함수가 호출 됩니다.

## 📖 Disposable

 [Observer Pattern 포스팅](https://kangraemin.github.io/android/2021/08/16/observer-pattern/)의 마지막 부분에서, 구독 취소에 대해 언급 한 적이 있습니다. 

 위 이야기를 다시 보면, 더이상 구독을 하지 않아도 될 때, 또는 구독을 끊어야 할 때 구독을 끊는 작업을 말합니다. 

 Rx에서도 Stream을 구독하는 구독자가, 구독을 끊어야 할 때 `dispose` 라는 메소드를 통해 구독을 끊을 수 있습니다. 

 다시 말하면, Stream을 구독하고 있는 구독자는 `dispose` ( 구독 취소 )가 가능한 상태이고, 이 상태를 나타낸 클래스가 `Disposable` 클래스 입니다. 

 이를 코드로 살펴보면 아래와 같습니다. ( `just` operator rx marble 참조 : [link](https://raw.githubusercontent.com/wiki/ReactiveX/RxJava/images/rx-operators/Single.just.png) )

```kotlin
val disposableInstance = Single
    .just("12345")
    .subscribe({
        Log.d("result", it)
    }, { it.printStackTrace() })

disposableInstance.dispose()
```

- Single stream에서 `just` operator를 활용하여 `12345` 라는 아이템을 발행합니다.
- Single stream에서 아이템을 발행하면, 발행 된 아이템을 Log를 찍기 위해 Single stream을 구독합니다.
    - 추후 subscribe에 대해서는 `Comsumer` 개념과 함께 다룰 예정이니, 위 코드에서 `subscribe`함수 안에 있는 로직을 제대로 이해하지 못했다 하더라도 전혀 문제 없으며, 추후에 다룰 subscribe / consumer 포스팅을 기대 해 주세요 !
- 구독 한 이후에는, 해당 구독을 끊을 수 있는 상태가 된 것이며, `Disposable` 클래스의 객체가 된 것입니다.
- 다시 말하면, `subscribe` 함수의 리턴 형태가 `Disposable` 클래스인 것입니다.
- 구독을 한 상태의 객체를 `disposableInstance` 라는 변수에 담고, `dispose` ( 구독취소 ) 하고 싶을 때 `dispose` 해주시면 됩니다.

 대부분의 경우에는, Activity / Fragment가 완전히 제거 될 때 즉,`onDestroy` 가 호출 되었을 때 해당 뷰에서 구독하고 있던 구독 관계를 끊어 줍니다. 

 이를 편하게 구현하기 위해, rx에서는 `CompositeDisposable` 이라는 클래스를 제공 해 줍니다. 

```kotlin
val compositeDisposable = CompositeDisposable()

compositeDisposable.add(
		Single
				.just("12345")
				.subscribe({
		        Log.d("result", it)
		    }, { it.printStackTrace() })
)

compositeDisposable.add(
		Single
				.just("54321")
				.subscribe({
		        Log.d("result", it)
		    }, { it.printStackTrace() })
)

compositeDisposable.dispose()
```

 이 클래스의 객체에는 `disposable` 객체를 담아 둘 수 있으며, 객체에 담긴 `disposable` 객체를 한번에 `dispose` 해 줄 수 있는 기능이 구현되어 있습니다.  

 따라서, `CompositeDisposable` 객체를 생성 해 놓고, 각 뷰에서 생성되는 `disposable` 객체를 한 곳에 담아두고, `onDestroy` 가 호출되거나, 구독 관계를 끊어 주어야 할 때 `compositeDisposable` 객체에 담긴 모든 `disposable` 객체를 한번에 `dispose` 해준다면, 쉽게 구독 관계를 끊을 수 있습니다.