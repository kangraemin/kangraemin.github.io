---
title: Kotlin collection - Filter / Partition / Test
date: 2021-03-06
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 Filter operator에 대해 알아봅니다. 

<!-- more -->

> 아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/Filter.kt) 에서 확인 하실 수 있습니다. 

## Filter operator

### Filter

- 원본 Collection은 수정하지 않고 조건에 맞는 collection을 생성하는 operator
- 조건 함수를 인자로 받으며, 조건 함수의 값이 true인 element들만으로 새로운 collection을 생성
- `filter()` / `filterIndexed()` 등의 operator 활용

```kotlin
fun checkDefaultFilterOperator() {
    val numbers = listOf("one", "two", "three", "four")
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)

    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
}
```

- index를 활용하여 filter 함수를 생성 하고 싶은경우 `filterIndexed()` 함수를 활용
- `filter()` operator는 조건 함수의 결과 값이 true인 element의 collection 인데, 조건 함수의 결과 값이 false로 collection을 만들고 싶다면 `filterNot()` 활용

```kotlin
fun checkOtherFilterOperator() {
    val numbers = listOf("one", "two", "three", "four")

    val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
    val filteredNot = numbers.filterNot { it.length <= 3 }

    println(filteredIdx)
    println(filteredNot)
}
```

- `filterIsInstance<T>` operator를 활용하여 Type check에 대한 filter 가능

```kotlin
fun typeCheckFilter() {
    val numbers = listOf(null, 1, "two", 3.0, "four")
    println("All String elements in upper case:")
    numbers.filterIsInstance<String>().forEach {
        println(it.toUpperCase())
    }
}
```

- Nullable 검사 filter

```kotlin
fun checkNullableFilter() {
    val numbers = listOf(null, "one", "two", null)
    numbers.filterNotNull().forEach {
        println(it.length)   // length is unavailable for nullable Strings
    }
}
```

### Partition

- filter → 조건에 맞는 하나의 collection 생성 / partition → 조건에 맞는 collection, 조건에 맞지 않는 collection 각각 하나씩 생성

```kotlin
fun checkPartition() {
    val numbers = listOf("one", "two", "three", "four")
    val (match, rest) = numbers.partition { it.length > 3 }

    println(match)
    println(rest)
}
```

### Test predicates

- Collection 안에 element가 특정 조건을 만족하는 것이 있는지 검사하는 operator
- `any()` / `none()` / `all()` operator 등을 사용
- `any()` → 하나의 element라도 조건을 만족하면 true / 그렇지 않으면 false
- `none()` → 모든 element가 조건을 만족하지 않으면 true / 그렇지 않으면 false
- `all()` → 모든 element가 조건을 만족하면 true / 그렇지 않으면 false ( empty collection에 대해선 모두 true 반환 )
- 가독성을 위해`any` 와 `all` 앞에는 `!` 을 붙여 사용하는것을 지양하는것이 좋다 ( !any → all / !all → any로 바꿔 쓸 수 있기 때문 )

```kotlin
fun checkTestPredicateOperatorWithCondition() {
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.any { it.endsWith("e") })
    println(numbers.none { it.endsWith("a") })
    println(numbers.all { it.endsWith("e") })

    println(emptyList<Int>().all { it > 5 })   // vacuous truth
}
```

- `any()` / `none()` 은 조건 식 없이도 사용 가능한데, 조건 식 없이 사용 할 시 element가 collection에 단 하나라도 존재 하는지 / 안하는지를 검사한다 ( == isEmpty() )

```kotlin
fun checkTestPredicateOperatorWithoutCondition() {
    val numbers = listOf("one", "two", "three", "four")
    val empty = emptyList<String>()

    println(numbers.any())
    println(empty.any())

    println(numbers.none())
    println(empty.none())
}
```

출처 : [https://kotlinlang.org/docs/collection-filtering.html](https://kotlinlang.org/docs/collection-filtering.html)