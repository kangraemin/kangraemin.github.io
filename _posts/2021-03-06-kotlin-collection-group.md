---
title: Kotlin collection - group by 
date: 2021-03-06
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 group operator에 대해 알아봅니다. 

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/Group.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Group elements operator

- Collection의 item들을 묶어 주는 operator
- `groupBy()` 등의 operator가 사용 됨
- map collection의 형태로 가공 해 준다
- key / value를 transform 해줄 수 있음

```kotlin
fun groupBySampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five")

    println(numbers.groupBy { it.first().toUpperCase() })
    println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))
}
```

- `groupingBy()` operator를 사용하여 한번에 모든 그룹에 똑같은 작업( `eachCount()` / `reduce()` / `fold()` / `aggregate()` )을 진행할 수 있다.

```kotlin
fun groupingBySampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six", "fffive")
    val groupingResult = numbers.groupingBy { it.first() }
    println(groupingResult.eachCount())
    println(groupingResult.fold("one") { a, b ->
        println("----")
        println(a)
        println(b)
        println("----")
        a + b })
    println("@@@")
    println(groupingResult.reduce { a, b, c ->
        println("----")
        println(a)
        println(b)
        println(c)
        println("----")
        a + b + c
    })

    val numberAggregated = listOf(3, 4, 5, 6, 7, 8, 9)

    val aggregated = numberAggregated.groupingBy { it % 3 }.aggregate { key, accumulator: StringBuilder?, element, first ->
        if (first)
            StringBuilder().append(key).append(":").append(element)
        else
            accumulator!!.append("-").append(element)
    }
    println(aggregated)
    println(aggregated.values)
}
```

