---
title: 반응형 프로그래밍 이란, RxJava란 - Reactive Programming이 무엇인가 
date: 2021-08-16
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

Reactive Programming이 무엇인지, ReactiveX / RxJava는 무엇인지 대해서 설명합니다.

<!-- more -->

# 📚 TL;DR

## 📖 Reactive Programming 이란

- 먼저, 특정 이벤트가 발생 했을 때 어떤 행동을 할 지 서술하는 방식을 이벤트 기반 프로그래밍이라 함
- 옵저버 패턴은, 데이터의 변화가 감지되면 옵저버들에게 데이터가 변화했음을 알려 줌
- 따라서, 옵저버 패턴을 사용하면 데이터의 변화를 하나의 이벤트, 데이터 흐름으로 다룰 수 있게 됨
- 데이터가 변경 되는것을 하나의 이벤트로 보고, 이벤트가 발생 했을 때 ( == 데이터가 변경 되었을 때 ) 어떤 작업을 진행 할 지 서술 해놓는 방식으로 개발하는 것이 반응형 프로그래밍

## 📖 RactiveX, RxJava 란

- ReactiveX는 Reactive Programming을 쉽게 할 수 있도록 도와주는 API를 제공하는 라이브러리
- ReactiveX를 활용하면, 데이터 흐름들을 서로 엮거나, 변환하여 다른 이벤트 흐름을 쉽게 만들어 낼 수 있음
- ReactiveX를 Java를 활용하여 구현한 라이브러리 → RxJava

## 📖 Reactive Programming 의 이점

- 데이터의 변화에 대응하는 pipeline을 한눈에 볼 수 있다.
- 콜백을 제거하고, 데이터 흐름으로 만들 수 있다.
- 데이터의 변화들을 엮어서 하나의 데이터 흐름으로 통합 하기 쉽다.

# 📚 Reactive Programming 이란

## 📖 Event Driven Programming 이란

 어떤 프로그램에서 유저가 로그인 버튼을 클릭 했을 때, 로그인 작업을 진행하는 서비스를 개발한다고 가정 해 봅시다. 

 이 서비스를 개발 하는 방법으로는 크게 두가지가 있습니다.

1. 0.1초 ( 짧은 시간 )마다 유저가 버튼을 클릭 했는지 확인하는 작업을 진행하고, 클릭 했음이 확인 되면 로그인 작업을 시작합니다. 

    ```kotlin
    while (true) {
    		if (checkButtonHasClicked()) {
    				startLoginTask();
    				break;
    		}
    }

    private fun checkButtonHasClicked() : Boolean {
    		// 버튼이 클릭 되면 True, 클릭된 적이 없으면 false를 리턴 
    }
    ```

2. 유저가 버튼을 클릭 했다는 신호가 왔을 때, 로그인 작업을 시작합니다. ( 콜백 함수를 등록하는 것과 같습니다. ) 

    ```kotlin
    // Button / ButtonClickListener는 Android에서 따온 것이며
    // 아래의 코드는 이미 안드로이드 SDK에 구현 되어 있어 가져다 사용하시면 됩니다. 
    // 아래에 적힌 코드들은 사실 Button의 상위 클래스인 View에 구현되어 있습니다. 
    class Button : View {
    		private val buttonClickListener: ButtonClickListener? = null

    		fun setOnClickListener(listener : ButtonClickListener) {
    				buttonClickListener = listener
    		}

    		fun performClick() {
    				buttonClickListener?.onClick()
    		}
    }

    interface ButtonClickListener {
    		fun onClick()
    }

    class LoginView {
    		private val loginButton : Button 

    		init {
    				// Button에 Click Listener 등록 ( CallBack 등록 ) 
    				loginButton.setOnClickListener {
    						startLoginTask()
    				}
    		}
    }
    ```

 위 두가지 방법 중 첫번째 방법은, 프로그램이 지켜 보아야 할 컴포넌트가 많아 졌을 때 성능상의 이슈가 있을 수 있으며, 비효율적인 연산이 상당히 많을 수 있습니다. 

 따라서, 안드로이드 OS에서는 대부분의 UI 이벤트를 두번째 방식으로 처리합니다.

 안드로이드 OS에는, 버튼 ( View ) 가 클릭 되었을 때, 위에 서술된 `performClick` 함수가 호출되도록 개발이 되어 있으며, `performClick` 함수 안에서는 등록된 `ButtonClickListener` 가 있다면 해당 listener 에게 버튼이 클릭되었다는 신호가 가도록 개발 되어 있습니다.  

 그러면, 유저가 버튼을 클릭 했을 때, 어떤 행동을 할 지 서술 해 놓고 등록 해 놓는다면 ( `setOnClickListener` 를 통하여 ) 유저가 버튼을 클릭 헀을 때, 등록된 행동을 진행하게 됩니다. 

 이와 같이, 특정 이벤트가 발생 했을 때 어떤 행동을 할지 서술하는 프로그래밍 기법을 `이벤트 기반 프로그래밍` 이라고 합니다. 

## 📖 Observer Pattern 이란

 Observer Pattern이 어떤 것인지는, 지난 포스팅에서 자세히 서술 해 두었으므로 [지난 포스팅](https://kangraemin.github.io/android/2021/08/16/observer-pattern/)을 참조 해 주세요 

 위 포스팅의 내용을 살펴보면, Observer Pattern은 관찰 대상 ( value in Subject )이 변경 되었을 때, 관찰 대상을 구독하고 있는 관찰자 ( Observer ) 들에게 값이 변경되었음을 알려준다고 나와 있습니다. 

 따라서, Observer Pattern을 활용하면, 데이터의 변화가 이벤트를 발생 시키는 것을 확인 할 수 있으며, 이는 데이터를 변경하는 것이 이벤트로 전환될 수 있음과 같은 이야기 입니다.

 또, Observer Pattern에서 관찰자 ( Observer )에는 값이 변경 되었을 때 어떤 행동을 할지 서술 해 놓으므로 값 변경에 대한 이벤트 기반 프로그래밍으로 볼 수 있습니다. 

## 📖 Reactive Programming 이란

  위에서 설명한 것과 같이, Observer Pattern을 활용하면 데이터가 변경 되었을 때 이를 하나의 이벤트로 변경 할 수 있습니다. 

 반응형 프로그래밍이란, Observer Pattern과 Event Programming을 엮어, 데이터가 변경 되었을 때 어떤 작업을 할 지 미리 선언해놓는 개발 방식입니다. 

 다시 말하면, 데이터가 변경 될 때 어떤 동작을 할 지 선언 해놓는 방식으로 개발하는 형식이 반응형 프로그래밍 인 것입니다. 

 반응형 프로그래밍에서는 Observer Pattern에서 Subject와 같이, 데이터의 변화를 방출하는 객체를 Data Stream 이라고 칭합니다. 

 반응형 프로그래밍에서는 데이터 스트림을 관찰자 ( Observer )가 구독 ( Subscribe )하고, 데이터 변경 시 관찰자가 어떤 행동을 할 지 서술 해 놓는 방식으로 개발을 진행하게 됩니다. 

> 반응형 프로그래밍은 데이터 스트림과 변화에 대한 선언적 프로그래밍 패러다임 - WIKI

 위키 백과에 보면, Reactive Programming이 위와 같이 정의 되어 있습니다. 

위에 적힌 말을 다시 해석 해 본다면, 

> 데이터 스트림 ( 데이터의 변화를 방출하는 객체 )를 통해 값이 변할 때 어떤 행동을 할지 미리 서술 해 놓는 프로그래밍 패러다임 - Rams

으로 해석 할 수 있습니다. 

# 📚 ReactiveX, RxJava 란

## 📖 ReactiveX 란

 ReactiveX는, Reactive Programming을 좀 더 편하게 구현하기 위해 만들어진 라이브러리 입니다. ( Docs : [link](http://reactivex.io/) ) 

 이 라이브러리를 활용하면, `데이터 스트림 생성` / `데이터 스트림 구독` / `데이터 스트림 구독 취소` / `데이터 스트림 구독 중에 에러가 발생 했을 시 할 행동 정의` / `데이터 스트림 병합` / `데이터 스트림 발행을 어떤 쓰레드에서 할 지, 데이터 스트림 구독을 어떤 쓰레드에서 할 지 지정` 등 데이터 스트림에 대한 작업들을 편리하고 쉽게 다룰 수 있습니다. 

## 📖 RxJava 란

 ReactiveX 라이브러리를 Java 기반으로 구현 한 것입니다. 

 ReactiveX 는 RxJava 뿐 아니라, RxSwift / RxPY / RxJS / RxScala 등 다른 언어로도 구현 되어 있습니다. 

 또 ReactiveX를 구현 할 때, 데이터 스트림을 다루는 메소드의 이름 / 데이터 스트림을 다루는 방식이 동일한 부분이 많아 RxJava를 익혀 놓는다면 크게 어려움 없이 다른 ReactiveX 라이브러리의 코드를 읽을 수 있습니다.