---
title: Android - Bundle 이란  
date: 2025-04-13
categories:
 - Android
tags:
 - Android 
 - Bundle
---
 

<!-- more -->

### Bundle 이란

- Android에서 Parcelable 인터페이스를 구현한 데이터 컨테이너
- 키-값 쌍의 형태로 데이터를 저장하고 전달하는 데 사용
- Intent, Fragment, Activity 간 데이터를 전달하거나 저장할 때 사용

### 주의사항

- 데이터 크기 제한: Bundle에 저장할 수 있는 데이터 크기는 제한적
  - 너무 큰 데이터를 저장하면 TransactionTooLargeException이 발생할 수 있음
- Parcelable은 Android에서 더 효율적이지만, 구현이 복잡할 수 있음
- Serializable은 간단하지만 성능이 떨어질 수 있음

### Bundle의 활용

- Activity 간 데이터 전달
  - Intent의 putExtras()를 통해 전달
    - 예시 코드

      ```kotlin
      // 현재 Activity에서
      val bundle = Bundle().apply {
          putString("key", "value")
          putInt("number", 123)
      }
      val intent = Intent(this, NewActivity::class.java)
      intent.putExtras(bundle)
      startActivity(intent)
      
      // NewActivity에서 데이터 꺼내기
      val value = intent.extras?.getString("key")
      val number = intent.extras?.getInt("number")
      
      ```

- Fragment 간 데이터 전달
  - Fragment의 setArguments()와 getArguments()를 통해 전달
    - 예시 코드

      ```kotlin
      val fragment = ExampleFragment().apply {
          arguments = Bundle().apply {
              putString("argKey", "argValue")
          }
      }
      // Fragment 내부에서 데이터 꺼내기
      val argValue = arguments?.getString("argKey")
      ```

- 상태 저장
  - onSaveInstanceState()와 onRestoreInstanceState()에서 사용
    - 예시 코드

      ```kotlin
      override fun onSaveInstanceState(outState: Bundle) {
          super.onSaveInstanceState(outState)
          outState.putString("stateKey", "stateValue")
      }
      
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          if (savedInstanceState != null) {
              val stateValue = savedInstanceState.getString("stateKey")
              // 저장된 상태 사용 함
          }
      }
      ```