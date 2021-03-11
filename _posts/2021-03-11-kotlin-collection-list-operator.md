---
title: Kotlin collection - List operator - indexOf / binary search / / sort
date: 2021-03-11
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 retrieve collection parts operator에 대해 알아봅니다. 

<!-- more -->

> 아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/RetrieveSingleElement.kt) 에서 확인 하실 수 있습니다. 

## List-specific operator

### Retrieve elements by index

- 참고 : [https://www.notion.so/Collection-944ef43da01a4ae4bdb12c0e08bc9e19#ae2b03224db845eeb74ff546008d2a91](https://www.notion.so/Collection-944ef43da01a4ae4bdb12c0e08bc9e19#ae2b03224db845eeb74ff546008d2a91)

### Retrieve list parts

- Index들을 담아놓은 list를 index list로 활용 가능

```kotlin
fun sampleCodeForRetrieveListParts() {
    val numbers = (0..13).toList()
    println(numbers)
    println(numbers.subList(3, 6))
}
```

### Find element positions

- `indexOf()` / `lastIndexOf()` 검색 후 처음으로 / 마지막으로 검색되어 나오는 index 값 하나만 나온다.
- 조건에 맞는 element를 찾지 못했을 경우 -1 리턴
- `indexOfFirst()` / `indexOfLast()` operator를 활용하여 조건 식을 통해 검색 가능

```kotlin
fun sampleCodeForLinearSearch() {
    val numbers = listOf(1, 2, 3, 4, 2, 5)
    println(numbers.indexOf(2))
    println(numbers.lastIndexOf(2))

    val numbersWithFunction = mutableListOf(1, 2, 3, 4)
    println(numbersWithFunction.indexOfFirst { it > 2})
    println(numbersWithFunction.indexOfLast { it % 2 == 1})
}
```

### Binary search in sorted lists

- binary search를 활용하여 **정렬 된** list에서 index 값을 리턴 해 준다.
- 정렬 하는 ordering 규칙이 있는 경우엔 정렬에 ordering 규칙을 사용하고, 없는 경우에 comparator를 제공하여 ordering 규칙 제공 가능
- 정렬 된 list가 아닌 list를 제공한 뒤 나오는 값은 정의 되지 않은 값이다. ( ref : [undefined](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/binary-search.html) )
- 같은 값이 여러개 있는 경우라면, 어떤 위치에 있는 값의 index가 나올 지는 장담 할 수 없다.
- 정렬 된 list에서, 검색하고자 하는 element를 찾지 못했을 때엔 (-insertionPoint - 1)값이 리턴된다. ( insertionPoint : 정렬된 list에서, 검색하고자 하는 element가 삽입 되어야 할 index 값 )

```kotlin
fun sampleCodeForBinarySearch() {
    val numbers = mutableListOf("one", "two", "three", "four")
    numbers.sort()
    println(numbers)
    println(numbers.binarySearch("two"))  // 3
    println(numbers.binarySearch("z")) // -5
    println(numbers.binarySearch("a")) // -1
    println(numbers.binarySearch("two", 0, 2))  // -3

    val numbersSecond = mutableListOf("one", "two", "three", "four", "aeae")
    numbersSecond.sort()
    println(numbersSecond)
    println(numbersSecond.binarySearch("two"))  // 4
    println(numbersSecond.binarySearch("z")) // -6
    println(numbersSecond.binarySearch("two", 0, 2))  // -3

    val numberNotSorted = mutableListOf("one", "two", "three", "four", "aeaze")
    println(numberNotSorted)
    println(numberNotSorted.binarySearch("two"))  // -6
    println(numberNotSorted.binarySearch("z")) // -6
    println(numberNotSorted.binarySearch("two", 0, 2))  // 1
}
```

- list에 ordering 규칙이 없는 경우, comparator 객체를 통해 ordering 규칙을 제공 할 수 있고, 반드시 이 규칙에 의해 sorting 된 상태여야 올바른 값을 내놓는다.

```kotlin
fun sampleCodeForCustomOrderingBinarySearch() {
    val productList = listOf(
            Product("WebStorm", 49.0),
            Product("AppCode", 99.0),
            Product("DotTrace", 129.0),
            Product("ReSharper", 149.0))

    println(productList.binarySearch(
            // thenBy -> Product의 price를 기준으로 값 비교를 하는데, 만약 두 객체를 비교 할 때 첫번째 조건식에서 같다는 값이 나왔을 경우 새롭게 비교 할 두번째 조건을 제시 해 줌   
            Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name }
    ))
}

data class Product(val name: String, val price: Double)
```

- Custom comparator를 활용하여 좀 더 쉽게 검색 가능

```kotlin
fun sampleCodeForCustomCondition() {
    val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
    println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3
}
```

- 비교 기준 함수를 제공 해 줌으로써 해당 함수를 통한 binary search를 진행 할 수 있다.
- 비교 기준 함수는 반드시 Int를 리턴하는 함수여야 한다.

```kotlin
fun sampleCodeForComparisonBinarySearch() {
    val productList = listOf(
            Product("WebStorm", 49.0),
            Product("AppCode", 99.0),
            Product("DotTrace", 129.0),
            Product("ReSharper", 149.0))

    println(productList.binarySearch { priceComparison(it, 99.0) })
}

fun priceComparison(product: Product, price: Double) = (product.price - price).toInt()

data class Product(val name: String, val price: Double)
```

### Update

- index 값을 통해 update 가능
- fill 함수를 통해 모든 값에 대한 update 가능

```kotlin
fun sampleCodeForUpdateListOperator() {
    val numbers = mutableListOf("one", "five", "three")
    numbers[1] =  "two"
    println(numbers)

    val filledNumbers = mutableListOf(1, 2, 3, 4)
    filledNumbers.fill(3)
    println(filledNumbers)
}
```

### Sort

- 리스트를 정렬 하기 위한 operator
- original collection의 순서들을 변경한다

```kotlin
val numbers = mutableListOf("one", "two", "three", "four")

numbers.sort()
println("Sort into ascending: $numbers")
numbers.sortDescending()
println("Sort into descending: $numbers")

numbers.sortBy { it.length }
println("Sort into ascending by length: $numbers")
numbers.sortByDescending { it.last() }
println("Sort into descending by the last letter: $numbers")

numbers.sortWith(compareBy<String> { it.length }.thenBy { it })
println("Sort by Comparator: $numbers")

numbers.shuffle()
println("Shuffle: $numbers")

numbers.reverse()
println("Reverse: $numbers")
```