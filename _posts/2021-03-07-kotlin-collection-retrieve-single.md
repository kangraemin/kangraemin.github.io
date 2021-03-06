---
title: Kotlin collection - elementAt / first / find / contains
date: 2021-03-07
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 retrieve single elements operator에 대해 알아봅니다.

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/RetrieveSingleElement.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Retrieve single elements operator

### Retrieve by position

- Index에 있는 element 하나를 가지고 오는데 사용되는 operator
- `elementAt()` set / list 에 있는 element를 가져오는데 사용

```kotlin
fun getElementFromSetSampleCode() {
    val numbers = linkedSetOf("one", "two", "three", "four", "five")
    println(numbers.elementAt(3))

    val numbersSortedSet = sortedSetOf("one", "two", "three", "four")
    println(numbersSortedSet.elementAt(0)) // elements are stored in the ascending order
}
```

- `elementAtOrNull()` 등의 operator를 활용하여 안정성을 높일 수 있음

```kotlin
fun getElementFromListSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers[0])
    println(numbers.get(0))
    println(numbers.elementAt(0))

    println(numbers.first())
    println(numbers.last())

    println(numbers.elementAt(20)) // runtime error
    println(numbers.elementAtOrNull(20))
    println(numbers.elementAtOrElse(20, { index -> "The value for index $index is undefined"}))
}
```

### Retrieving by condition

- 조건 식에 맞는 값들 중 한가지 element ( first / last )를 가져옴
- `first()` / `last()` 가 사용되며, 조건 식에 맞는 값 중 첫번째 / 마지막 element를 리턴
- 조건에 맞는 값이 collection에 없을 시 `NoSuchElementException` 을 발생 시킴

```kotlin
fun firstLastSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.first { it.length > 3 })
    println(numbers.last { it.startsWith("f") })
}
```

- `NoSuchElementException` 방지를 위해 `firstOrNull()` 등의 operator 사용 ( 조건에 맞는 element 없을 시 null 리턴 )

```kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.firstOrNull { it.length > 6 })
```

- `find()` operator로도 사용 가능

```kotlin
fun findSampleCode() {
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.find { it % 2 == 0 }) // ( == firstOrNull ) 
    println(numbers.findLast { it % 2 == 0 }) // ( == lastOrNull ) 
}
```

### Random element

- collection에서 랜덤한 element 하나를 가져오는 operator
- empty collection 에서는 `NoSuchElementException` 을 발생 시키기 때문에, `randomOrNull()` operator 사용 가능

```kotlin
fun checkRandomOperatorCode() {
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.random())
    println(numbers.randomOrNull())
}
```

### Check element existence

- Collection에 특정 element가 존재 하는지 안하는지 확인 해주는 operator
- `contains()` 등을 활용

```kotlin
fun checkExistenceSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.contains("four"))
    println("zero" in numbers)

    println(numbers.containsAll(listOf("four", "two")))
    println(numbers.containsAll(listOf("one", "zero")))
}
```

- `isEmpty()` collection이 비었는지, 안비었는지 확인 할 수 있음

```kotlin
fun isEmptySampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.isEmpty())
    println(numbers.isNotEmpty())

    val empty = emptyList<String>()
    println(empty.isEmpty())
    println(empty.isNotEmpty())
}
```

출처 : [https://kotlinlang.org/docs/collection-elements.html](https://kotlinlang.org/docs/collection-elements.html)