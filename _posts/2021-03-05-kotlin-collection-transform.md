---
title: Kotlin collection - Map / Zip / Associate / Flatten / String representation
date: 2021-03-05
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 Transform operator에 대해 알아봅니다. 

<!-- more -->

> 아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/Transform.kt) 에서 확인 하실 수 있습니다. 

## Transform operator

### Map

- Item들을 특정 함수를 활용하여 변형 시키고, 그 결과값에 대한 Collection을 반환 해 줌
- `map()` / `mapIndexed()`/ `mapNotNull()` 등의 operation이 존재

```kotlin
fun checkDefaultMapOperator() {
    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    println(numbers.mapIndexed { idx, value -> value * idx })

    val numbersNullCheck = setOf(1, 2, 3)
    println(numbersNullCheck.mapNotNull { if ( it == 2) null else it * 3 })
    println(numbersNullCheck.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
}
```

- map collection을 변형 시킬 때는, value는 놔둔 채 key를 변형시키는 `mapKey()`, 그와 반대로 작동하는 `mapValues()` 가 존재

```kotlin
fun makeMapCollectionUsingMapOperator() {
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    println(numbersMap.mapKeys { it.key.toUpperCase() })
    println(numbersMap.mapValues { it.value + it.key.length })
}
```

### Zip

- 각각 Collection의 동일한 position에 있는 item들을 결합 ( Pair의 형태 ) 하여 새로운 Collection을 반환
- collection과 collection을 zip으로 엮으면, Pair객체를 반환

```kotlin
fun checkDefaultZipOperator() {
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")
    println(colors zip animals)

    val twoAnimals = listOf("fox", "bear")
    println(colors.zip(twoAnimals))
}
```

- collection 두개를 zip으로 생성한 결과 collection을, 변형 함수를 통해 변형하여 사용 할 수 있다.
- 이때 변형 함수의 인자는 첫번째, 두번째 collection의 item이 들어간다.

```kotlin
fun checkZipUsingTransformFunction() {
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")

    println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})
}
```

- Pair 객체를 element의 type으로 갖는 List는, List와 List의 zip을 통해 만들어 진 것과 같다고 볼 수 있으므로 `unzip()` 함수를 통해 두개의 List로 분할이 가능하다

```kotlin
fun checkUnzipFunction() {
    val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
    println(numberPairs.unzip())
}
```

### Associate

- Collection의 item으로 부터 map 객체를 생성 해 준다.
- `associateWith()` / `associateBy()` 등의 함수를 사용

```kotlin
fun checkAssociateWithOperator() {
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.associateWith { it.length })
}
```

- associateBy 함수를 통해, key / value 값도 조작 가능

```kotlin
fun checkAssociateByOperator() {
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.associateBy { it.first().toUpperCase() })
    println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
}
```

- Pair 객체를 리턴하는 lambda 함수를 associate() 함수의 인자로 넣어주어 map collection 생성 사능

```kotlin
fun associateOperator() {
    val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
    println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })
}

fun parseFullName(name: String): FullName {
    return FullName(name.split(" ")[0], name.split(" ")[1])
}

data class FullName(val lastName: String, val firstName: String)
```

### Flatten

- `flatten()` → 평탄화 작업 operation, 중첩된 collection이 있을 때, 내부 collection의 element들을 외부 collection의 element로 사용 하여 새로운 collection 생성

```kotlin
fun flattenOperator() {
    val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
    println(numberSets.flatten())
}
```

- `flatMap()` → 중첩된 collection 안의 element를 map 함수와 같이 변형시켜 새로운 collection을 생성

```kotlin
fun flatMapOperator() {
    val containers = listOf(
            StringContainer(listOf("one", "two", "three")),
            StringContainer(listOf("four", "five", "six")),
            StringContainer(listOf("seven", "eight"))
    )
    println(containers.flatMap { it.values })
}

data class StringContainer(val values: List<String>)
```

### String representation

- collection을 읽기 편한 양식으로 만들어 주는 operation
- `joinToString()` / `joinTo()` 등을 활용

```kotlin
fun checkJoinToOperator() {
    val numbers = listOf("one", "two", "three", "four")

    println(numbers)
    println(numbers.joinToString())

    val listString = StringBuffer("The list of numbers: ")
    numbers.joinTo(listString)
    println(listString)
}
```

- separator / prefix / postfix / limit / truncated / transform 등의 값들을 활용하여 custom joinToString 함수를 활용 할 수 있음

```kotlin
fun checkJoinToOperatorWithParams() {
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))

    val numbersToCheckLimit = (1..100).toList()
    println(numbersToCheckLimit.joinToString(limit = 10, truncated = "<...>"))

    val numbersToCheckTransform = listOf("one", "two", "three", "four")
    println(numbersToCheckTransform.joinToString { "Element: ${it.toUpperCase()}"})
}
```

출처 : [https://kotlinlang.org/docs/collection-transformations.html](https://kotlinlang.org/docs/collection-transformations.html)