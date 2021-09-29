---
title: RxJava 강의 10 - RxBinding / Throttle / Debounce
date: 2021-09-29
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

Android Component에서 발행하는 UI Event를 rxStream으로 변환 해주는데 사용되는 라이브러리인 RxBinding을 알아보고, throttle / debounce를 활용하여 UI Event를 stream으로 변환 하였을 때 얻을 수 있는 이점 등을 알아봅니다. 

<!-- more -->

# 📚 TL; DR

## 📖 RxBinding

- Android에서 발행하는 UI Event를 rxStream으로 변환 해주는 라이브러리
- 이를 활용하여 UI Event를 Stream으로 처리 할 수 있게 되므로, RxJava의 다양한 operator를 활용 할 수 있음
- 각 rxBinding은 내부적으로 Android listener를 활용하고 있음
- `clicks()`, `throttle` 을 활용하여 다중 클릭 방지를 할 수 있으며, `textChanges()`, `debounce` 를 활용하여 불필요한 작업을 피할 수 있음

# 📚 RxBinding

## 📖 개요

 RxBinding Library ( github docs : [link](https://github.com/JakeWharton/RxBinding) )는 View의 Click, focusChange 등 Android에서 발행하는 UI Event를 reactiveX의 stream 형식으로 변환 해주는 라이브러리 입니다. 

 즉, 버튼 클릭 / EditText 포커스 해제 등에 대한 이벤트를 하나의 Stream으로 다룰 수 있게 해 주며, 이를 활용하여 Stream을 변형 하거나 합성하는 등에 활용 할 수 있습니다. 

 아래에서 사용된 코드는 [Github](https://github.com/kangraemin/RxJavaStudy/blob/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/rxbinding/RxBindingExampleActivity.kt)에서 확인 하실 수 있습니다. 

## 📖 설치

 RxBinding을 사용하실 모듈 ( app module )의 build.gradle 파일에 아래의 dependency를 추가 해 줍니다. 

```kotlin
// RxJava2를 활용하는 project 라면 rxbinding3를 설치 해 주세요.
implementation 'com.jakewharton.rxbinding3:rxbinding:3.1.0'

// RxJava3를 활용하는 project 라면 rxbinding4를 설치 해 주세요.
implementation 'com.jakewharton.rxbinding4:rxbinding:4.0.0'
```

 설치시 주의할 점은 RxBinding maven을 살펴보면 아래와 같습니다.

- RxBinding3의 maven ( [link](https://mvnrepository.com/artifact/com.jakewharton.rxbinding3/rxbinding/3.1.0) )
    
    ![Untitled](/assets/images/posts/2021-09-29-rxbinding/Untitled.png)
    
- RxBinding4의 maven ( [link](https://mvnrepository.com/artifact/com.jakewharton.rxbinding4/rxbinding/4.0.0) )
    
    ![Untitled](/assets/images/posts/2021-09-29-rxbinding/Untitled%201.png)
    

 위를 살펴보면, RxBinding 내부에 RxAndroid와 RxJava가 포함 된 것을 확인 할 수 있습니다. 

 따라서, RxJava2, RxJava3를 이미 활용하고 있는 project에 binding 라이브러리를 추가 할 때엔 RxJava2를 쓰고 있다면 RxBinding3를, RxJava3를 쓰고 있다면 RxBinding4를 설치 해 주세요. 

 이번 포스팅에선 RxJava2를 활용하고 있는 RxBinding3의 3.1.0 버전을 활용하여 서술하겠습니다. 

## 📖 RxBinding 기능 개요

 RxBinding은 앞서 말씀 드렸듯, 뷰에서 발생하는 이벤트들을 Stream으로 다룰 수 있도록 변환 해 주고 있습니다. 

 이번 포스팅에선 대표적인 `clicks()` / `textChanges()` operator를 알아 보겠습니다. 

## 📖 Clicks

- 특징
    - `view`의 클릭 이벤트가 발생 했을 때 `Unit` 객체를 emit 하는 observable을 생성 해 줍니다.
    - 실제 구현 코드는 [RxBinding3-ViewClickObservable](https://github.com/JakeWharton/RxBinding/blob/3.1.0/rxbinding/src/main/java/com/jakewharton/rxbinding3/view/ViewClickObservable.kt) 에서 확인 할 수 있습니다.
        
        ```kotlin
        @CheckResult
        fun View.clicks(): Observable<Unit> {
          return ViewClickObservable(this)
        }
        
        private class ViewClickObservable(
          private val view: View
        ) : Observable<Unit>() {
        
          override fun subscribeActual(observer: Observer<in Unit>) {
            if (!checkMainThread(observer)) {
              return
            }
            val listener = Listener(view, observer)
            observer.onSubscribe(listener)
            view.setOnClickListener(listener)
          }
        
          private class Listener(
            private val view: View,
            private val observer: Observer<in Unit>
          ) : MainThreadDisposable(), OnClickListener {
        
            override fun onClick(v: View) {
              if (!isDisposed) {
                observer.onNext(Unit)
              }
            }
        
            override fun onDispose() {
              view.setOnClickListener(null)
            }
          }
        }
        ```
        
    - 실제 구현 코드를 확인해보면, 내부적으로 OnClickListener interface를 구현하는 Listener를 생성하여, setOnClickListener의 인자로 해당 Listener를 대입 해 준것을 확인 할 수 있습니다.
    - 즉, RxBinding 라이브러리 내부에서 setOnClickListener를 활용 하고 있는것을 확인 할 수 있습니다.
- 예시 코드
    
    ```kotlin
    var clickedCount = 0
    
    compositeDisposable
        .add(
            binding
                .btnBindingClick
                .clicks()
                .subscribe({
                    binding.tvClickTime.text = String.format("클릭된 횟수 : %d", ++clickedCount)
                }, { it.printStackTrace() })
        )
    ```
    
    ![20210929_191006.gif](/assets/images/posts/2021-09-29-rxbinding/20210929_191006.gif)
    
    - 버튼을 클릭하면, `clicks()` operator에서 Unit 이벤트를 방출하고, 이벤트가 방출되어 subscribe 안에 코드가 동작 했음을 확인 할 수 있습니다.

## 📖 Clicks + Throttle 조합

- `ThrottleFirst()` operator
    - Marble Diagram
        
        ![Untitled](/assets/images/posts/2021-09-29-rxbinding/Untitled%202.png)
        
    - 방출되는 이벤트 중 첫번째 이벤트를 발행하고, 미리 정해둔 일정 시간동안 이벤트를 skip 하는 operator 입니다.
    - 이 operator와 clicks를 활용하여 다중 클릭을 방지 할 수 있습니다.
- 예시 코드
    
    ```kotlin
    var throttleClickedCount = 0
    
    compositeDisposable
        .add(
            binding
                .btnThrottle
                .clicks()
                .throttleFirst(1000, TimeUnit.MILLISECONDS)
                .subscribe({
                    binding.tvClickTimeThrottle.text = String.format("클릭된 횟수 : %d", ++throttleClickedCount)
                }, { it.printStackTrace() })
        )
    ```
    
    ![20210929_191052.gif](/assets/images/posts/2021-09-29-rxbinding/20210929_191052.gif)
    
    - ThrottleFirst operator를 통해, 첫번째 클릭 이벤트가 발생 한 뒤 1초동안은 클릭 이벤트를 발행하지 않도록 합니다.
    - 따라서 유저가 1초 내에 여러번 클릭을 하더라도 첫번째 클릭 밖에 발행되지 않았음을 확인 할 수 있습니다.

## 📖 TextChanges

- 특징
    - TextView ( EditText 는 TextView를 상속 받으므로 EditText도 포함됩니다. ) 컴포넌트에 담긴 text가 변경 될 때 변경되는 CharSequence를 방출 해주는 operator 입니다.
    - 실제 구현 코드는 [RxBinding3-TextViewTextChangesObservable](https://github.com/JakeWharton/RxBinding/blob/3.1.0/rxbinding/src/main/java/com/jakewharton/rxbinding3/widget/TextViewTextChangesObservable.kt) 에서 확인 할 수 있습니다.
        
        ```kotlin
        @CheckResult
        fun TextView.textChanges(): InitialValueObservable<CharSequence> {
          return TextViewTextChangesObservable(this)
        }
        
        private class TextViewTextChangesObservable(
          private val view: TextView
        ) : InitialValueObservable<CharSequence>() {
        
          override fun subscribeListener(observer: Observer<in CharSequence>) {
            val listener = Listener(view, observer)
            observer.onSubscribe(listener)
            view.addTextChangedListener(listener)
          }
        
          override val initialValue get() = view.text
        
          private class Listener(
            private val view: TextView,
            private val observer: Observer<in CharSequence>
          ) : MainThreadDisposable(), TextWatcher {
        
            override fun beforeTextChanged(s: CharSequence, start: Int, count: Int, after: Int) {
            }
        
            override fun onTextChanged(s: CharSequence, start: Int, before: Int, count: Int) {
              if (!isDisposed) {
                observer.onNext(s)
              }
            }
        
            override fun afterTextChanged(s: Editable) {
            }
        
            override fun onDispose() {
              view.removeTextChangedListener(this)
            }
          }
        }
        ```
        
    - 실제 구현 코드를 확인해보면, 내부적으로 TextWatcher interface를 구현하는 Listener를 생성하여, addTextChangedListener의 인자로 해당 Listener를 대입 해 준것을 확인 할 수 있습니다.
    - 즉, RxBinding 라이브러리 내부에서 addTextChangedListener를 활용 하고 있는것을 확인 할 수 있습니다.
- 예시 코드
    
    ```kotlin
    compositeDisposable
        .add(
            binding
                .etTextChange
                .textChanges()
                .subscribe({
                    binding.tvEditTextResult.text = String.format("입력된 text : %s", it)
                }, { it.printStackTrace() })
        )
    ```
    
    ![20210929_191120.gif](/assets/images/posts/2021-09-29-rxbinding/20210929_191120.gif)
    
    - EditText에 적힌 텍스트가 변경 될 때 마다 `입력된 text :` 의 뒷부분의 텍스트가 변경 되는것을 확인 할 수 있습니다.
    - 따라서, 유저가 텍스트를 입력 할 때 마다 입력한 텍스트가 발행되는것을 확인 할 수 있습니다.

## 📖 TextChanges + Debounce

- Debounce
    - Marble Diagram
        
        ![Untitled](/assets/images/posts/2021-09-29-rxbinding/Untitled%203.png)
        
    - 방출된 이벤트를, 미리 정해 둔 시간이 지난뒤에 발행 하는것을 확인 할 수 있습니다.
    - 노란색 이벤트를 발행하기 이전, 즉 노란색 이벤트 발행을 기다리고 있는 동안에 초록색 이벤트가 발행 되었는데, 노란색 이벤트가 발행되지 않고 일정시간이 지난 뒤 초록색 이벤트가 발행 된 것을 확인 할 수 있습니다.
    - 따라서 debounce는 이벤트가 발생하고 정해둔 시간 뒤에 이벤트를 발행하지만, 이벤트 발행 전 다른 이벤트가 발생 할 경우 이전에 발행되는 이벤트를 취소시키는 것을 확인 할 수 있습니다.
    - 이런 debounce의 특성과 textChanges operator를 활용하여 실시간 검색 editText 등에 활용 할 수 있습니다.
- 예시 코드
    
    ```kotlin
    compositeDisposable
        .add(
            binding
                .etSearch
                .textChanges()
                .debounce(1000, TimeUnit.MILLISECONDS)
                .subscribe({
                    binding.tvSearchText.text = String.format("입력된 text : %s", it)
                }, { it.printStackTrace() })
        )
    ```
    
    ![20210929_191140.gif](/assets/images/posts/2021-09-29-rxbinding/20210929_191140.gif)
    
    - debounce를 적용 하기 전 예시를 보면, 유저가 입력 할 때 마다 `입력된 text :` 부분이 변경 된 것을 확인 할 수 있습니다.
    - 하지만 debounce operator를 추가한 이후엔 유저가 텍스트를 입력 하더라도, 입력한 텍스트가 일정 시간 이후에 발행 되는 것을 확인 할 수 있습니다.
    - 이를 실시간 검색에 활용하는 예시를 위해, 유저가 텍스트를 입력할 때 마다 서버에 검색 api를 호출한다고 가정 해 봅시다.
    - debounce가 없는 경우엔 `rams` 를 입력 할 때 서버에 `r` / `ra` / `ram` / `rams` 이렇게 네번의 api 호출이 진행됩니다.
    - 하지만 debounce를 적용 해 놓으면, 유저가 1초 내에 `rams`를 입력 할 경우 마지막에 발행된 이벤트인 `rams` 데이터에 대해서만 api 호출이 진행됩니다.