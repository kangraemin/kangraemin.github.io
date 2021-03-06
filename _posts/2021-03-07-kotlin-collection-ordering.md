---
title: Kotlin collection - sotring operator / custom sotring
date: 2021-03-07
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 ordering operator에 대해 알아봅니다. 

<!-- more -->

> 아래에 적힌 Kotlin 코드들은 [Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/Ordering.kt) 에서 확인 하실 수 있습니다. 

## Ordering operator

- collection element들 간의 collection 속 순서 정렬에 관련된 operator
- collection 속에는, element들 간의 정렬 순서를 정하는 rule이 있다.
    - natural order
        - Comparable interface를 상속하여 구현되어 있음
        - 다른 order 규칙이 정의되어 있지 않을 때 사용 됨
        - Numeric type ( 숫자 ... )는 숫자의 크기 순서를 사용 ( -1은 0보다 앞에 옴 ... )
        - Char / String 은 [lexicographical order](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%A0%84%EC%8B%9D_%EC%88%9C%EC%84%9C) ( 사전식 순서 )를 따른다. ( b는 a보다 뒤에 옴 ... )

        ```kotlin
        val numbers = listOf("one", "two", "three", "four")

        println("Sorted ascending: ${numbers.sorted()}")
        println("Sorted descending: ${numbers.sortedDescending()}")
        ```

    - user-defined order
        - Comparable interface를 상속하여 유저가 직접 구현하여야 함
        - `compareTo()` 함수를 반드시 구현하도록 되어 있음
        - `compareTo()` 함수는, 반드시 동일한 type의 다른 객체를 인자로써 받고, int 값을 return
        - `compareTo()` 에서 리턴 된 int값을 통하여 객체의 순서를 결정
            - int 값이 양수면, 앞의 값이 뒤의 값보다 크다 ( 뒤에 온다 )
            - int 값이 음수면, 앞의 값이 뒤의 값보다 작다 ( 앞에 온다 )
            - int 값이 0이면, 앞의 객체와 뒤의 객체는 동일한 순서를 가진다.

        ```kotlin
        fun sampleCodeForCustomCompareClass(){    
            println(Version(1, 2) > Version(1, 3))
            println(Version(2, 0) > Version(1, 5))
            println(Version(2, 0) == Version(2, 0))
            println(Version(2, 0) > Version(2, 0))
        }

        data class Version(val major: Int, val minor: Int) : Comparable<Version> {
            // compareTo -> 앞 객체와 뒷 객체의 크기 비교 원칙을 서술
            // compareTo 함수의 결과 값이 음수 -> 뒷 객체가 더 큰것을 의미
            // compareTo 함수의 결과 값이 양수 -> 앞 객체가 더 큰것을 의미
            // compareTo 함수의 결과 값이 0 -> 두 객체의 값이 같음을 의미
            override fun compareTo(other: Version): Int {
                return when {
                    this.major != other.major -> {
                        this.major - other.major
                    }
                    this.minor != other.minor -> {
                        this.minor - other.minor
                    }
                    else -> 0 // When version is same
                }
            }
        }
        ```

        - custom order를 통하여 내가 원하는 데로 순서를 정하는 규칙을 만들 수 있다.
        - sortedWith comparator를 직접 구현하여 비교 할 수 있다.
            - 이때, comparator에는 인자 두개가 오는데 각 인자는 비교하려는 객체들이 들어온다. ( 아래의 예에선 str1 / str2 )
            - str1 / str2는 str1이 비교하려는 객체에서 뒤에 있는 것, str2가 비교하려는 객체에서 앞에 있는 것에 유의. ( ex_ 아래의 예에서 "aaa", "bb"를 비교 할 때, str1은 "bb", str2는 "aaa"가 된다. )
            - 따라서, 오름차순으로 정렬 하고 싶으면 앞의 객체, 뒤의 객체를 대입 하였을 때 값이 양수가 나오도록 하면 된다.
                - str1.length - str2.length 를 comparator의 구현체로 대입 하였을 때
                    - str2가 앞의객체, str1이 뒤의 객체인데 첫번째 비교를 보면 str1은 "bb", str2는 "aaa"가 되고, 이는 2-3, 즉 음수가 나온다.
                    - 따라서 앞 객체의 값이 더 큰것이 되고 순서를 바꿔야 한다.
                    - 즉, 문자열이 긴 값이 가장 뒤로 가게 된다.
                - str2.length - str1.length 를 comparator의 구현체로 대입 하였을 때
                    - 위 경우 str1은 "bb", str2는 "aaa"  3-2가 되어, 즉 양수가 나온다.
                    - 따라서 앞 객체의 값이 더 큰것이 되고 순서를 바꾸지 않고 가만히 두게 된다.
                    - 즉, 문자열이 긴 값이 가장 앞으로 오게 된다.

                ```kotlin
                fun sampleCodeForCustomComparator() {
                    val lengthComparator = Comparator { str1: String, str2: String ->
                        str1.length - str2.length
                    }
                    println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))
                    println(listOf("aaa", "bb", "c").sortedWith { x, y -> x.length - y.length })

                val lengthComparatorReverse = Comparator { str1: String, str2: String ->
                        str2.length - str1.length
                    }
                    println(listOf("aaa", "bb", "c").sortedWith(lengthComparatorReverse))
                    println(listOf("aaa", "bb", "c").sortedWith { x, y -> y.length - x.length })
                }
                ```

        - compareBy 활용
            - comparator를 직접 구현 하는 것 대신 활용 가능
            - comparable을 상속받은 값을 리턴하는 함수를 활용하여 좀 더 쉽게 구현 가능
            - natural order ( Int / String 등 )에는 이미 comparable interface가 상속되어있고, 구현되어 있기 때문에 사용 가능
            - 아래의 예에선 it.length를 리턴하는데, 이는 Int 값이고, Int 값은 comparable interface를 상속 중이므로 사용 가능

            ```kotlin
            fun sampleCodeForCompareBy() {
                println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
            }
            ```

        - Natural order를 활용한 custom ordering

        ```kotlin
        fun sampleCodeForCustomOrderingUsingNaturalOrdering() {
            val numbers = listOf("one", "two", "three", "four")

            val sortedNumbers = numbers.sortedBy { it.length }
            println("Sorted by length ascending: $sortedNumbers")
            val sortedByLast = numbers.sortedByDescending { it.last() }
            println("Sorted by the last letter descending: $sortedByLast")

            val numbersCompareBy = listOf("one", "two", "three", "four")
            println("Sorted by length ascending: ${numbersCompareBy.sortedWith(compareBy { it.length })}")
        }
        ```

### Reversed order

- collection의 순서를 뒤바꾸는 operator
- 아래의 예에서 list의 `asReversed()` operator는, `ReversedList()` 객체를 return 하는데, `ReversedList()` 는 add시 list의 뒤쪽에 element를 삽입하는 것이 아닌, 앞쪽에 삽입한다. ( 따라서 역순 유지 )

```kotlin
fun sampleCodeForReverseOrder() {
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.reversed()) // [four, three, two, one]

    val mutableNumbers = mutableListOf("one", "two", "three", "four")
    println(mutableNumbers.reversed()) // [four, three, two, one]
    mutableNumbers.add("five")
    println(mutableNumbers) // [one, two, three, four, five]

    val numbersAsReversed = listOf("one", "two", "three", "four")
    val reversedNumbers = numbersAsReversed.asReversed()
    println(reversedNumbers) // [four, three, two, one]

    val mutableNumbersAsReversed = mutableListOf("one", "two", "three", "four")
    val mutableReversedNumbers = mutableNumbersAsReversed.asReversed()
    println(mutableReversedNumbers) // [four, three, two, one]
    mutableNumbersAsReversed.add("five")
    println(mutableReversedNumbers) // [five, four, three, two, one]
}
```

### Random order

- collection의 순서를 랜덤하게 섞어버리는 operator

```kotlin
fun sampleCodeForRandomOrder() {
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.shuffled())
}
```

출처 : [https://kotlinlang.org/docs/collection-ordering.html](https://kotlinlang.org/docs/collection-ordering.html)