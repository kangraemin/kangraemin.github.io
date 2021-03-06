---
title: Kotlin collection - Slice / Take and drop / Chunked / Windowed
date: 2021-03-07
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 retrieve collection parts operator에 대해 알아봅니다. 

아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/RetrieveCollectionElement.kt) 에서 확인 하실 수 있습니다. 

<!-- more -->

## Retrieve collection parts operator

- Element들을 가져오는데 쓰이는 operator
- `slice()` / `take()` ... 등을 사용

### Slice

- Index를 기반으로 collection을 잘라 새로운 collection을 생성한다.
- range / collection을 인자로 받을 수 있다.

```kotlin
fun sliceCollectionSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.slice(1..3))
    println(numbers.slice(0..4 step 2))
    println(numbers.slice(setOf(3, 5, 0)))
}
```

### Take and drop

- collection의 앞 / 뒤에서 부터 n번째 element 까지 잘라 새로운 collection을 생성하는 operator

```kotlin
fun takeDropSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.take(3))
    println(numbers.takeLast(3))
    println(numbers.drop(1))
    println(numbers.dropLast(5))
}
```

- Boolean 값을 리턴하는 함수와 함께 쓸 수 있음
- `takeWhile()` → element의 조건식이 false가 나오기 전까지 element들을 take ( false가 나온 이후의 element들은 생략 )
- `takeLastWhile()` → collection의 뒤에서 부터 element의 조건식이 false가 나오기 전까지 element들을 take ( false가 나온 이후의 element들은 생략 )

```kotlin
fun takeDropSampleCodeWithPredicates() {
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.takeWhile { !it.startsWith('f') })
    println(numbers.takeLastWhile { it != "three" })
    println(numbers.dropWhile { it.length == 3 })
    println(numbers.dropLastWhile { it.contains('i') })
}
```

### Chunked

- collection을 n개씩 잘라 새로운 collection을 만듬
- n개씩 자르다 남는 마지막 collection의 크기는 n보다 작거나 같음

```kotlin
fun chunkSampleCode() {
    val numbers = (0..13).toList()
    println(numbers.chunked(3))
}
```

- chunked된 collection을 transform 할 수 있음

```kotlin
fun chunkWithTransformSampleCode() {
    val numbers = (0..13).toList()
    println(numbers.chunked(3) { it.sum() })  // `it` is a chunk of the original collection
}
```

### Windowed

- collection을 앞에서 부터 n개 묶어 list를 생성하고, step size ( default size = 1 ) 만큼씩 index를 건너 뛰어 n개씩 묶은 list를 생성 한 뒤 생성된 list를 element로 가지는 collection 생성
- 예제를 보면 이해가 쉬울 것

```kotlin
fun windowedSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers.windowed(3))
}
```

- step / partialWindows 파라메터를 활용하여 묶는 조건 추가 가능
- transform 가능

```kotlin
fun windowedWithParamsSampleCode() {
    val numbers = (1..10).toList()
    println(numbers.windowed(3, step = 2, partialWindows = false))
    println(numbers.windowed(3, step = 2, partialWindows = true))
    println(numbers.windowed(3) { it.sum() })
}
```

- `zipWithNext()` 을 활용하여 list로 반환하는 window와 달리, element를 묶어 Pair객체로 생성하고, transform 함수도 활용 가능

```kotlin
fun zipWithNextSampleCode() {
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers.zipWithNext())
    println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})
}
```

출처 : [https://kotlinlang.org/docs/collection-parts.html](https://kotlinlang.org/docs/collection-parts.html)