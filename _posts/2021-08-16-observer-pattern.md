---
title: 옵저버 패턴 이란, Observer Pattern
date: 2021-08-16
categories:
 - Android
tags:
 - Observer Pattern
 - Design Pattern
---

Observer pattern이 무엇인지, 사용하면 어떤 장점이 있는지 등을 알아봅니다. 

[Github repo](https://github.com/kangraemin/RxJavaStudy/tree/main/AndroidProject) 에서 아래에 사용된 예시 코드를 확인 하실 수 있습니다. 


<!-- more -->

# 📚 TL;DR

## 📖 Observer Pattern 이란

- 옵저버 패턴에는 관찰 대상 ( Subject ) / 관찰자 ( Observer ) 가 있다.
- 각 관찰자에는, 관찰자의 상태가 변경 되었을 때 어떤 행동을 할 지 구현 해 둔다.
- 관찰 대상에는 관찰자를 등록 해 놓는다. 즉, 관찰자가 관찰 대상을 구독 ( Subscribe ) 하면, 관찰 대상에 관찰자를 등록 해 놓는다.
- 관찰 대상의 상태가 변경되면, 관찰자들에게 상태가 변경 되었음을 알린다 ( Notify / Update ... )
- 관찰 대상의 상태가 변경 되었을 때, 관찰자는 구현 해 놓은 행동을 진행한다.

## 📖 Observer Pattern을 사용 했을 때의 이점

- 관찰자 ( Observer ) 들간의 의존성을 떨어 뜨릴 수 있다.
- 관찰자 ( Observer ) 를 추가하는데 유연하게 대응 할 수 있다.
- 관찰 대상의 상태 변화에 따른 행동을 유연하게 처리 할 수 있다.

# 📚 Observer Pattern을 사용하지 않고 구현 한 경우

## 📖 상황 가정

아래의 조건을 만족시키는 UI를 구현 해야 하는 경우를 가정 해 봅시다.

- 유저는 숫자를 입력하거나 SeekBar 조절을 통해 투명도를 조절 할 수 있습니다.
- 유저가 투명도를 조절 할 때, 아래와 같이 이에 맞추어 각 UI가 변경 되어야 합니다. ( 투명도 입력 란의 글자 / SeekBar의 Percent / 검정색 네모의 투명도 / 투명도 수치 )

    ![first.gif](/assets/images/posts/2021-08-16-observer-pattern/first.gif)

## 📖 구현한 코드 설명

> 안드로이드 코드를 잘 몰라도, 구체적인 구현 사항이 전혀 이해가 안가도 됩니다. 아래에 구현된 코드가, 어떤 문제가 있는지만 이해를 하고 넘어가시면 됩니다.

```kotlin
package com.example.rxjavalecture.observerpattern.noneobserverpattern

import android.os.Bundle
import android.text.Editable
import android.text.TextWatcher
import android.util.Log
import android.widget.EditText
import android.widget.ImageView
import android.widget.SeekBar
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.example.rxjavalecture.R
import com.example.rxjavalecture.TAG_OBSERVER_PATTERN

class NoneObserverPatternActivity : AppCompatActivity() {

    private lateinit var etPercent: EditText
    private lateinit var sbPercent: SeekBar
    private lateinit var tvPercent: TextView
    private lateinit var imgOpacityResult: ImageView

		private var disableSeekBarChangeListener = false
    private var disableEditTextChangeListener = false

		// SeekBar가 변경 되었을 때 할 행동을 정의 합니다.
    private val seekBarChangeListener = object : SeekBar.OnSeekBarChangeListener {
				// SeekBar의 퍼센트가 변경 되었을 때 호출되는 부분입니다. 
        override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
						if (!disableSeekBarChangeListener) {
                Log.d(TAG_OBSERVER_PATTERN, "progress = $progress in SeekBar")
                updateUIBySeekBar(progress)
            }
        }

        override fun onStartTrackingTouch(seekBar: SeekBar?) {

        }

        override fun onStopTrackingTouch(seekBar: SeekBar?) {

        }
    }

		// 유저가 투명도의 값을 입력하였을 때 할 행동을 정의합니다.
    private val textWatcher = object : TextWatcher {
        override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {

        }

        override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {

        }

				// 퍼센트를 입력 하는 공간의 텍스트가 변경 되었을 때 호출되는 부분입니다. 
        override fun afterTextChanged(s: Editable?) {
						if (!disableEditTextChangeListener) {
                val percent = s.toInt()
                if (isItProperRangeNumber(percent)) {
                    Log.d(TAG_OBSERVER_PATTERN, "percent = $percent in EditText")
                    updateUIByEditText(percent)
                }
            }
        }
    }

		// 앱에서 화면이 create 될 때 할 행동을 정의합니다.
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_non_observer_pattern)

        etPercent = findViewById(R.id.et_percent)
        sbPercent = findViewById(R.id.sb_percent)
        tvPercent = findViewById(R.id.tv_percent)
        imgOpacityResult = findViewById(R.id.img_opacity_result)

        sbPercent.setOnSeekBarChangeListener(seekBarChangeListener)
        etPercent.addTextChangedListener(textWatcher)
    }

		// 유저가 입력한 숫자가 올바른 투명도 범위에 있는지 확인합니다.
    private fun isItProperRangeNumber(inputNumber: Int): Boolean {
        return inputNumber in 0..100
    }

		// 유저가 숫자를 입력함으로써 변경되는 UI update 작업을 서술합니다.
    private fun updateUIByEditText(percent: Int) {
				disableSeekBarChangeListener = true
        sbPercent.progress = percent
				disableSeekBarChangeListener = false
        updateOpacityUI(percent)
    }

		// 유저가 SeekBar를 조절함으로써 변경되는 UI update 작업을 서술합니다.
    private fun updateUIBySeekBar(percent: Int) {
				disableEditTextChangeListener = true
        etPercent.setText(percent.toString())
				disableEditTextChangeListener = false
        updateOpacityUI(percent)
    }

		// 유저가 숫자를 입력하거나, SeekBar를 조절함으로써 변경되는 공통적인 UI 작업을 서술합니다.
    private fun updateOpacityUI(percent: Int) {
        tvPercent.text =
            String.format(getString(R.string.first_week_observer_pattern_result_format), percent)
        imgOpacityResult.alpha = percent / 100f
    }
}

// 확장함수 서술합니다. ( text -> Int 변환, 모르셔도 상관 없습니다. ) 
fun Editable?.toInt(): Int {
    if (isNullOrEmpty()) {
        return 0
    }
    return toString().toInt()
}
```

## 📖 위 코드의 문제점

 위 코드를 쪼개 보면, 유저가 투명도 조절을 위해 숫자를 입력하는 부분과, SeekBar 조절을 통해 투명도를 조절 하는 부분으로 나눌 수 있습니다. 

### 💡 유저가 투명도 숫자를 입력을 통해 투명도 조절을 했을 때 진행 되는 Flow / 의존성 Diagram

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled.png)

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled%201.png)

- 유저가 숫자를 입력 했을 때, 갱신 해주어야 하는 UI는 Image / 투명도 Text / SeekBar 입니다.

### 💡 유저가 SeekBar 조절을 통해 투명도 조절을 했을 때 진행되는 Flow / 의존성 Diagram

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled%202.png)

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled%203.png)

- 유저가 SeekBar의 퍼센트 조절을 통해 투명도를 조절 했을 때, 갱신 해 주어야 하는 UI는 EditText / Image / 투명도 Text 입니다.

UI를 갱신 해주는 부분을 코드로 살펴보면 아래와 같습니다. 

```kotlin
// 유저가 숫자를 입력함으로써 변경되는 UI update 작업을 서술합니다.
private fun updateUIByEditText(percent: Int) {
    disableSeekBarChangeListener = true
    sbPercent.progress = percent
    disableSeekBarChangeListener = false
    updateOpacityUI(percent)
}

// 유저가 SeekBar를 조절함으로써 변경되는 UI update 작업을 서술합니다.
private fun updateUIBySeekBar(percent: Int) {
    disableEditTextChangeListener = true
    etPercent.setText(percent.toString())
    disableEditTextChangeListener = false
    updateOpacityUI(percent)
}

// 유저가 숫자를 입력하거나, SeekBar를 조절함으로써 변경되는 공통적인 UI 작업을 서술합니다.
private fun updateOpacityUI(percent: Int) {
    tvPercent.text =
        String.format(getString(R.string.first_week_observer_pattern_result_format), percent)
    imgOpacityResult.alpha = percent / 100f
}
```

위 코드를 살펴보면, `updateUIByEditText` 함수 안에서, `SeekBar` 를 참조 하고 있는것을 볼 수 있으며, `updateUIBySeekBar` 함수 안에서는 `EditText` 에 대한 참조가 존재하는 것을 확인 할 수 있습니다. 

즉, 유저가 실제로 변경 한 부분이 아닌 다른부분에 대한 참조가 있어야 UI가 제대로 갱신 될 수 있음을 의미하며, 이는 각 컴포넌트 ( EditText / SeekBar ) 간의 의존성이 생기는 것을 의미합니다. 

만약 위와 같은 상황에서, 투명도를 입력받는 다른 방법이 추가된다면 어떻게 될까요 ?

예를들어, 아래와 같이 플러스 버튼을 누르면 투명도가 증가되고, 마이너스 버튼을 누르면 투명도가 감소하는 UI를 추가한다고 가정 해 봅시다. 그리고 이 버튼을 `plusMinusButton` 이라고 가정 해 봅시다. 

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled%204.png)

 그렇다면, 유저가 숫자를 입력하거나 SeekBar 조절을 통해 투명도를 조절 해 줄 때, `plusMinusButton`의 숫자도 변경 해 주어야 합니다.

따라서 `updateUIByEditText` 함수 안에는 `plusMinusButton` 에 대한 참조가 추가적으로 들어가야 하며, 또 `updateUIBySeekBar` 함수 안에도 `plusMinusButton` 에 대한 참조가 추가 되어야 합니다. 

 즉, 각 컴포넌트 ( SeekBar / EditText / PlusMinusButton ) 간의 의존성이 추가적으로 계속 생겨나며, 이는 하나 컴포넌트를 제거 / 추가하는데 변경 해야 할 부분이 많은 코드를 생산하게 됩니다.

 따라서 생산된 코드는 유지보수가 힘든 코드가 될 수 있습니다. 

# 📚 Observer Pattern을 사용 하여 구현 한 경우

## 📖 위 상황을 재해석 해보자

 위에서 생산한 코드가 문제가 있을 수 있음을 알아 보았습니다. 

 UI 변경에 있어 각 컴포넌트간의 의존성이 생겨, 컴포넌트가 추가 될 때 유지보수가 힘들 수 있는것이 문제점이었습니다. 

 그리고, 위 상황을 자세히 살펴보면, 변경되는 변수는 `투명도` 하나인 것을 알 수 있습니다. 

 따라서, 각 컴포넌트가 투명도가 변경되는 것을 지켜보다가, `투명도`가 변경 되었다고 알림을 받았을 때, 각 컴포넌트에 알맞는 UI 갱신만 해주게 된다면 UI 갱신에 있어 서로 독립적일 수 있습니다. 

 따라서, 아래와 같은 의존성을 가지도록 진행 하면, 각 컴포넌트간이 독립적으로 운영 될 수 있습니다.

![Untitled](/assets/images/posts/2021-08-16-observer-pattern/Untitled%205.png)

 

## 📖 Observer Pattern 이란

  자, 여기서 Observer Pattern의 정의를 알아 보도록 합시다. 

 Wiki에 적힌 Observer Pattern에 대한 정의는 아래와 같습니다. 

> 옵서버 패턴(observer pattern)은 객체의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다. - Wiki

 이를 다시 해석 해보면, 아래와 같습니다. 

1. 옵저버 패턴에는, 관찰 대상이 되는 객체 ( Subject )와 / 이를 관찰하는 관찰자 ( Observer )들이 있습니다. 
2. 또, 관찰 대상이 되는 객체에, 이 객체를 관찰하는 관찰자들을 등록합니다. ( subscribe )
3. 관찰 대상이 되는 객체의 상태 변화가 있을 때, 각 관찰자들에게 이를 통지합니다. ( notify / update )

 통상적으로, 관찰 대상이 되는 객체를 `Subject` 라고 부르며, 이를 관찰하는 관찰자를 `Observer` 라고 부르며, 관찰자를 관찰 대상이 되는 객체에 등록하는 작업을 `Subscribe` 라고 표기하는 경우가 잦습니다. 

 이를 유튜브 채널과, 채널을 구독하는 구독자에 비유를 해보면

- 유튜브 채널을 구독하는 구독자는 `Observer` 가 됩니다.
- 유튜브 채널은 `Subject` 가 됩니다.
- 구독자가 유튜브 채널의 구독 버튼을 누르는 행위를 `subscribe` 라 할 수 있습니다.
- 새로운 유튜브 영상이 올라 오면, 유튜브 채널은 구독자들에게 새로운 영상이 올라왔다고 알립니다. ( `update` / `notify` )

 Observer / Subject 를 간단하게 구현 해보면 아래와 같습니다. 

```kotlin
class RamsSubject<K> {

    private val observerList: MutableList<RamsObserver<K>> = ArrayList()

		var value: K? = null
        set(newValue) {
            if (field != newValue) {
                field = newValue
                for (ramsObserver in observerList) {
                    ramsObserver.notifyDataIsChanged(newValue)
                }
            }
        }

    fun subscribe(ramsObserver: RamsObserver<K>) {
        observerList.add(ramsObserver)
    }
}
```

```java
public interface RamsObserver<T> {
    void notifyDataIsChanged(T value);
}
```

### 💡 Subject ( 유튜브 채널 )

- Subject에는 observerList객체에 Observer 객체들을 저장 해 둡니다.
    - 유튜브 채널에 구독자의 목록을 저장 해 둡니다.
- Subject에 어떤 값 ( value )이 변경 되었을 때, observerList에 있는 observer들에게 데이터가 변경 되었음을 알립니다.
    - 유튜브 채널에 새로운 동영상이 업로드 되었을 때, 구독자들에게 동영상이 업로드 되었음을 알립니다.
- Subscribe 메소드를 통해, subject에 observer를 등록합니다.
    - 구독 버튼을 통해, 구독자가 채널을 구독합니다.

```kotlin
ramsSubject.value = someValue 
```

- 위 형식을 통해, Subject 속의 값을 변경 시킬 수 있으며, 이 값이 이전 값과 다른 경우 Observer들에게 값이 변경 되었음을 알립니다.
- 옵저버 패턴에서는, subject 객체에 접근 할 수 있는 곳에서 value의 값을 변경 시킬 수 있습니다.
    - 이를 비유하자면, 유튜브 채널에 누군가 ( 구독자 여도 상관 없습니다. )가 유튜브 채널에 동영상을 새로 올리는 행위로 빗댈 수 있습니다.
    - 이 부분이 유튜브 채널과 구독자의 관계에서는 존재하지 않는 관계여서 헷갈릴 수 있으나, 옵저버 패턴에서는 Subject에 접근 할 수 있는 곳이라면 어디서든 값을 발행 할 수 있습니다.

### 💡 Observer ( 구독자 )

- Observer는 값이 변경 되었을 때, 어떤 행동을 할 지 서술 해야 합니다.
- 따라서, Observer 라면 반드시 `notifyDataIsChanged` 를 구현하도록 interface 형태로 서술 해 줍니다.

## 📖 그래서, Observer pattern을 활용해서 어떻게 문제를 해결하나

 코드로 먼저 살펴보겠습니다. 

### 💡 View ( ObserverPatternActivity )

```kotlin
package com.example.rxjavalecture.observerpattern.oberserverpattern

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.example.rxjavalecture.R
import com.example.rxjavalecture.observerpattern.oberserverpattern.customui.PercentEditText
import com.example.rxjavalecture.observerpattern.oberserverpattern.customui.PercentImageView
import com.example.rxjavalecture.observerpattern.oberserverpattern.customui.PercentSeekBar
import com.example.rxjavalecture.observerpattern.oberserverpattern.customui.PercentTextView

class ObserverPatternActivity : AppCompatActivity() {

    private lateinit var etPercent: PercentEditText
    private lateinit var sbPercent: PercentSeekBar
    private lateinit var tvPercent: PercentTextView
    private lateinit var imgOpacityResult: PercentImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_observer_pattern)

        etPercent = findViewById(R.id.et_percent)
        sbPercent = findViewById(R.id.sb_percent)
        tvPercent = findViewById(R.id.tv_percent)
        imgOpacityResult = findViewById(R.id.img_opacity_result)
    }

    companion object {
        val progressSubject = RamsSubject<Int>()
    }
}
```

- progress ( 투명도의 퍼센트 )상태를 관리하는 Subject를 생성합니다.

```kotlin
val progressSubject = RamsSubject<Int>()
```

- Percent 값이 변경 될 때, 변경 되어야 할 UI를 Observer로 두고, 옵저버에서 이 Subject를 구독합니다.

### 💡 Custom EditText ( EditText Observer )

```kotlin
package com.example.rxjavalecture.observerpattern.oberserverpattern.customui

import android.content.Context
import android.util.AttributeSet
import android.util.Log
import androidx.appcompat.widget.AppCompatEditText
import androidx.core.widget.doAfterTextChanged
import com.example.rxjavalecture.TAG_OBSERVER_PATTERN
import com.example.rxjavalecture.observerpattern.oberserverpattern.RamsObserver
import com.example.rxjavalecture.observerpattern.oberserverpattern.ObserverPatternActivity.Companion.progressSubject
import com.example.rxjavalecture.util.toInt

class PercentEditText : AppCompatEditText, RamsObserver<Int> {

    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

    private var disableEditTextChangeListener = false

    init {
				// ObserverPatternActivity에 있는 progressSubject에 구독 신청
        progressSubject.subscribe(this)

        doAfterTextChanged {
            if (!disableEditTextChangeListener) {
                val percent = it.toInt()
                if (isItProperRangeNumber(percent)) {
                    Log.d(TAG_OBSERVER_PATTERN, "percent = $percent in EditText")
                    progressSubject.value = percent
                }
            }
        }
    }

    private fun isItProperRangeNumber(inputNumber: Int): Boolean {
        return inputNumber in 0..100
    }

		// Percent 값이 변경 되었을 때 할 행동을 정의
    override fun notifyDataIsArrived(value: Int?) {
        value?.let {
            Log.d(TAG_OBSERVER_PATTERN, "value = $value in EditText")
            disableEditTextChangeListener = true
            setText(value.toString())
            disableEditTextChangeListener = false
        }
    }
}
```

### 💡 Custom ImageView ( ImageView Observer )

```kotlin
package com.example.rxjavalecture.observerpattern.oberserverpattern.customui

import android.content.Context
import android.util.AttributeSet
import androidx.appcompat.widget.AppCompatImageView
import com.example.rxjavalecture.observerpattern.oberserverpattern.RamsObserver
import com.example.rxjavalecture.observerpattern.oberserverpattern.ObserverPatternActivity

class PercentImageView : AppCompatImageView, RamsObserver<Int> {

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
            alpha = it / 100f
        }
    }
}
```

### 💡 Custom SeekBar ( SeekBar Observer )

```kotlin
package com.example.rxjavalecture.observerpattern.oberserverpattern.customui

import android.content.Context
import android.util.AttributeSet
import android.util.Log
import android.widget.SeekBar
import androidx.appcompat.widget.AppCompatSeekBar
import com.example.rxjavalecture.TAG_OBSERVER_PATTERN
import com.example.rxjavalecture.observerpattern.oberserverpattern.RamsObserver
import com.example.rxjavalecture.observerpattern.oberserverpattern.ObserverPatternActivity

class PercentSeekBar : AppCompatSeekBar, RamsObserver<Int> {

    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

    private var disableEditTextChangeListener = false

    init {
				// ObserverPatternActivity에 있는 progressSubject에 구독 신청
        ObserverPatternActivity.progressSubject.subscribe(this)
        setOnSeekBarChangeListener(object : OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                if (!disableEditTextChangeListener) {
                    Log.d(TAG_OBSERVER_PATTERN, "progress = $progress in SeekBar")
                    ObserverPatternActivity.progressSubject.value = progress
                }
            }

            override fun onStartTrackingTouch(seekBar: SeekBar?) {

            }

            override fun onStopTrackingTouch(seekBar: SeekBar?) {

            }
        })
    }

		// Percent 값이 변경 되었을 때 할 행동을 정의
    override fun notifyDataIsArrived(value: Int?) {
        value?.let {
            Log.d(TAG_OBSERVER_PATTERN, "value = $value in SeekBar")
            disableEditTextChangeListener = true
            progress = it
            disableEditTextChangeListener = false
        }
    }
}
```

### 💡 Custom TextView ( TextView Observer )

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

- RamsObserver를 구현하도록 custom UI를 만들고, Percent 값이 변경 되었을 때 할 행동들을 각 컴포넌트에 맞도록 정의 해 줍니다.

## 📖 위와 같이 Observer Pattern을 적용 했을 때의 장점

- 이렇게 Observer Pattern을 적용 했을 때 각 컴포넌트간의 의존성을 제거 할 수 있습니다.
    - 위 코드를 살펴보면, EditText가 SeekBar / TextView / ImageView에 의존성을 갖지 않습니다.
    - 다른 컴포넌트도 마찬가지로, 어떤 컴포넌트는 다른 컴포넌트에 의존성을 갖지 않습니다.
- 컴포넌트간 의존성이 없기 때문에, 다른 컴포넌트가 추가 되었을 때, 다른 컴포넌트에 영향을 전혀 주지 않습니다.
- 값이 변경 되었을 때 각 컴포넌트가 어떤 행동을 할 지 코드 분석이 매우 용이합니다.
- EditText에 있는 `isItProperRangeNumber` 와 같은 함수를 캡슐화 할 수 있습니다.

## 📖 구독 취소에 대해

```kotlin
class RamsSubject<K> {
    private val observerList: MutableList<RamsObserver<K>> = ArrayList()
    fun dispose(ramsObserver: RamsObserver<K>) {
        observerList.remove(ramsObserver)
    }
}
```

- Dispose 메소드를 통해, 구독하고 있는 observer를 subject의 observerList에서 제거합니다.
    - 구독자가 구독 취소 버튼을 통해 채널 구독을 끊습니다.
- 이를 통해, 만약 더이상 구독을 하지 않아도 될때 또는 구독을 끊어야 할 때, 올바르지 않은 갱신 작업을 하지 않을 수 있습니다.
- 이는 메모리 관리 측면에서도 매우 중요한 이슈 일 수 있습니다.