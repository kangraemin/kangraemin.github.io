---
title: Kotlin collection - Set operator - union / intersect / subtract
date: 2021-03-11
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 Set specific operator에 대해 알아봅니다. 

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/SetOperator.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Set-specific operator
- set collection은 수학에서의 집합과 매우 닮아있다. 따라서, 집합에 관련된 연산들을 사용 할 수 있다. ( 합집합 (`union` / 교집합 `intersect` / 차집합 `subtract` )

```kotlin
fun sampleCodeForSetOperation() {
    val numbers = setOf("one", "two", "three")

    println(numbers union setOf("four", "five"))
    println(setOf("four", "five") union numbers)

    println(numbers intersect setOf("two", "one"))
    println(numbers subtract setOf("three", "four"))
    println(numbers subtract setOf("four", "three")) // same output
}
```

출처 : [https://kotlinlang.org/docs/set-operations.html](https://kotlinlang.org/docs/set-operations.html)