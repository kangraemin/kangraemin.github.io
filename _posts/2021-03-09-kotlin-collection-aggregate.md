---
title: Kotlin collection - count / max / min / fold / reduce 
date: 2021-03-09
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 Aggregate operations operator에 대해 알아봅니다. 

<!-- more -->

> 아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/Aggregate.kt) 에서 확인 하실 수 있습니다. 

## Aggregate operations operator

- collection의 item들에 대한 하나의 값을 도출 해내는 operator
- 다른 언어들과 비슷하게 동작 함

```kotlin
fun sampleCodeForDefaultAggregate() {
    val numbers = listOf(6, 42, 10, 4)

    println("Count: ${numbers.count()}")
    println("Max: ${numbers.maxOrNull()}")
    println("Min: ${numbers.minOrNull()}")
    println("Average: ${numbers.average()}")
    println("Sum: ${numbers.sum()}")
}
```

- Comparator를 활용하여 max / min 값 정할 수 있음

```kotlin
fun sampleCodeForUsingCustomComparator() {
    val numbers = listOf(5, 42, 10, 4)
    val min3Remainder = numbers.minByOrNull { it % 3 }
    println(min3Remainder)

    val strings = listOf("one", "two", "three", "four")
    val longestString = strings.maxWithOrNull(compareBy { it.length })
    println(longestString)
}
```

- Transform 한다음 sum 가능

```kotlin
fun sampleCodeForSumOfOperator() {
    val numbers = listOf(5, 42, 10, 4)
    println(numbers.sumOf { it * 2 })
    println(numbers.sumOf { it.toDouble() / 2 })
}
```

### Fold and reduce

- 연산의 결과 값을 그 다음 인자로써 활용 할 수 있는 operator
- `fold()` operator는 initial value 값을 받아 첫번째 인자로써 활용하고, `reduce()` operator는 초기 값 없이 진행

```kotlin
fun sampleCodeForDefaultFoldReduce() {
    val numbers = listOf(5, 2, 10, 4)

    val sum = numbers.reduce { sum, element -> sum + element }
    println(sum)

    val sumDoubled = numbers.fold(0) { sum, element -> sum + element * 2 }
    println(sumDoubled)

    val sumDoubledReduce = numbers.reduce { sum, element -> sum + element * 2 } //incorrect: the first element isn't doubled in the result
    println(sumDoubledReduce)
}
```

- `foldRight()` / `reduceRight()` 는, `fold()` / `reduce()` 가 왼쪽 에서 부터 시작하는것과는 반대로 오른쪽 부터 값의 축적이 시작된다.

```kotlin
fun sampleCodeForFoldReduceRight() {
    val numbers = listOf(5, 2, 10, 4)
    val sumDoubledRight = numbers.foldRight(0) { element, sum -> sum + element * 2 }
    println(sumDoubledRight)
}
```

- `fold()` / `reduce()` 연산 도중에, index를 parameter로 활용 하고 싶을 때 사용

```kotlin
fun sampleCodeForFoldReduceIndexed() {
    val numbers = listOf(5, 2, 10, 4)
    val sumEven = numbers.foldIndexed(0) { idx, sum, element -> if (idx % 2 == 0) sum + element else sum }
    println(sumEven)

    val sumEvenRight = numbers.foldRightIndexed(0) { idx, element, sum -> if (idx % 2 == 0) sum + element else sum }
    println(sumEvenRight)
}
```

- `reduce()` 는 emptyCollection에서 `UnsupportedOperationException`을 발생 시키는데, 이를 방지하고자 `*OrNull()` 를 활용 ( `fold()` 는 emptyCollection에서도 exception 발생 x )
    - ex_ `reduceOrNull()` / `reduceRightOrNull()` ...
- `runningReduceSum()` / `runningFoldSum()` 함수를 활용하여, operator 중간 중간에 계산되는 sum에 대한 list를 생성 할 수 있음

```kotlin
fun sampleCodeForFoldReduceNull() {
    val numbers = listOf(0, 1, 2, 3, 4, 5)
    val runningReduceSum = numbers.runningReduce { sum, item -> sum + item }
    val runningFoldSum = numbers.runningFold(10) { sum, item -> sum + item }
}
```