---
title: Kotlin collection - plus / minus
date: 2021-03-06
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 plus / minus operator에 대해 알아봅니다. 

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/PlusMinus.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Plus / Minus operator

### Plus / Minus

- Collection은 `+` / `-` 기호도 지원 ( collection과 collection, collection과 동일한 type의 element 끼리 지원 )

```kotlin
fun plusMinus() {
    val numbers = listOf("one", "two", "three", "four")

    val plusList = numbers + "five"
    val minusList = numbers - listOf("three", "four")
    println(plusList)
    println(minusList)
}
```

- augmented Assignment operator ( `-=` / `+=` )는 list 자체를 재정의하는 것과 같아 `var` / read only collection ( immuttable ) 타입에서만 지원

```kotlin
fun augmentedAssignment() {
    val valNumberCollection = listOf("one", "two", "three", "four")
    // Compile error
    valNumberCollection += "five"
    valNumberCollection -= listOf("three", "four")
    valNumberCollection -= "four"

    // mutableList로 바꿀 것을 권장, + / - operator 자체가 재할당 하는 형식이기 때문  
    var varNumberCollection = listOf("one", "two", "three", "four")
    varNumberCollection += "five"
    varNumberCollection -= listOf("three", "four")
    varNumberCollection -= "four"

    val valNumberMutableCollection = mutableListOf("one", "two", "three", "four")
    valNumberMutableCollection += "five"
    valNumberMutableCollection -= listOf("three", "four")
    valNumberMutableCollection -= "four"
}
```

출처 : [https://kotlinlang.org/docs/collection-plus-minus.html](https://kotlinlang.org/docs/collection-plus-minus.html)