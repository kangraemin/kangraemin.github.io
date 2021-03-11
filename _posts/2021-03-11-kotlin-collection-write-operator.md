---
title: Kotlin collection - Write operator - add / remove ( + / - )
date: 2021-03-11
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 retrieve collection parts operator에 대해 알아봅니다. 

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/PlusMinus.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Write operations operator

- mutableCollection의 element를 추가 / 삭제하는데 사용되는 operator

### Adding elements

- `add()` / `addAll()` / `+` 등의 연산자가 활용 됨

```kotlin
fun sampleCodeForAddingElements() {
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    println(numbers)

    val numbersAddAll = mutableListOf(1, 2, 5, 6)
    numbersAddAll.addAll(arrayOf(7, 8))
    println(numbersAddAll)
    numbersAddAll.addAll(2, setOf(3, 4))
    println(numbersAddAll)

    val numbersPlusOperator = mutableListOf("one", "two")
    numbersPlusOperator += "three"
    println(numbersPlusOperator)
    numbersPlusOperator += listOf("four", "five")
    println(numbersPlusOperator)
}
```

### Removing elements

- `remove()` / `removeAll()` / `-` 등의 연산자가 활용 됨
- `retain()` → operation 함수의 결과 값이 true 인 것만 남기고 나머지 element들은 제거

```kotlin
fun sampleCodeForRemovingElements() {
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers)
    numbers.retainAll { it >= 3 }
    println(numbers)
    numbers.clear()
    println(numbers)

    val numbersSet = mutableSetOf("one", "two", "three", "four")
    numbersSet.removeAll(setOf("one", "two"))
    println(numbersSet)

    val numbersMinusOperator = mutableListOf("one", "two", "three", "three", "four")
    numbersMinusOperator -= "three"
    println(numbersMinusOperator)
    numbersMinusOperator -= listOf("four", "five")
//   numbersMinusOperator -= listOf("four")   // does the same as above
    println(numbersMinusOperator)
}
```

출처 : [https://kotlinlang.org/docs/collection-plus-minus.html](https://kotlinlang.org/docs/collection-plus-minus.html)
