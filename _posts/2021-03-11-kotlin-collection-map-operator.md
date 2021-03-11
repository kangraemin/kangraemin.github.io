---
title: Kotlin collection - Map operator - get / filter / plus / minus / put / remove
date: 2021-03-11
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

> Kotlin Collection 공식 문서를 정리 한 글입니다. Collection의 retrieve collection parts operator에 대해 알아봅니다. 

[Github repo](https://github.com/kangraemin/kotlin_study/blob/master/kangraemin/collection/src/MapOperator.kt) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Map-specific operator

### Retrieve keys / values

- key 값을 활용하여 value 값을 들고 온다.
- key값이 존재하지 않는 경우, exception을 발생 시킬 수 있으므로 default 값을 주거나, eles 구문을 활용하여 value를 도출 해 내는 함수를 활용 할 수 있다.

```kotlin
fun sampleCodeForMapOperator() {
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.get("one"))
    println(numbersMap["one"])
    println(numbersMap.getOrElse("four", { getSomethingInt() }))
    println(numbersMap.getOrDefault("four", 10))
    println(numbersMap["five"])              // null
    //numbersMap.getValue("six")     // exception!
}

fun getSomethingInt(): Int {
    return 100
}
```

- `keys` `values` 를 활용하여 map의 key / value 들을 한번에 살펴 볼 수 있다

```kotlin
val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
println(numbersMap.keys)
println(numbersMap.values)
```

### Filter

- list / set의 filter operator와 같이, map에서도 key / value를 기반으로 filter operator를 활용 할 수 있다.

```kotlin
fun sampleCodeForFilterForMap() {
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)

    val numbersMapForOneProperty = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredKeysMap = numbersMapForOneProperty.filterKeys { it.endsWith("1") }
    val filteredValuesMap = numbersMapForOneProperty.filterValues { it < 10 }

    println(filteredKeysMap)
    println(filteredValuesMap)
}
```

### Plus / Minus operators

- map에서도 set / list와 같이 `+` / `-` operator를 활용 할 수 있다.
- 대신에, `+` operator를 활용하여 추가하려고 하는 Pair 객체의 key값이 이미 map에 존재 하는 경우, 기존의 Pair객체를 추가하려고 하는 Pair객체로 대체한다.
- `-` operator를 활용하는 경우, 기존의 map을 수정하는 것이 아닌 새로운 map 객체를 생성한다.
- `-` operator를 활용하는 경우, key값을 활용하여 Pair객체를 제거 할 수 있다.

### Map write operations

- MutableMap 객체를 활용하여 map안의 데이터를 조작 할 수 있다.
- 데이터 조작 작업을 할 때, 정해진 규칙사항이 있다.
    - value는 값이 업데이트 될 수 있지만, key값은 절대 변할 수 없다. ( key값이 추가되거나, 제거 될 수 있지만 변경 시킬 수는 없음 )
    - key값은 항상 상수로 사용됨
    - key값에는 반드시 하나의 value가 존재한다.
    - key=value로 이루어진 Pair 객체를 읽기 / 추가 / 삭제 할 수 있다.
- Add
    - `put` 을 활용하여 데이터 추가 가능
    - `put` 함수는, map에 현재 추가하려는 Pair객체의 key값이 이미 존재하면 존재하는 key값의 value 값을 리턴하고, 그렇지 않으면 null 값을 리턴 ( `putAll` 함수는 값을 리턴하지 않음 ( Unit 리턴 ))

    ```kotlin
    fun sampleCodeForAddOperator() {
        val numbersMap = mutableMapOf("one" to 1, "two" to 2)
        numbersMap.put("three", 3)
        println(numbersMap)

        val numbersMapPutAll = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
        numbersMapPutAll.putAll(setOf("four" to 4, "five" to 5))
        println(numbersMapPutAll)

        val numbersMapChangeValue = mutableMapOf("one" to 1, "two" to 2)
        val previousValue = numbersMapChangeValue.put("one", 11)
        println("value associated with 'one', before: $previousValue, after: ${numbersMapChangeValue["one"]}")
        println(numbersMapChangeValue)

        val numbersMapShorthand = mutableMapOf("one" to 1, "two" to 2)
        numbersMapShorthand["three"] = 3    // calls numbersMapShorthand.put("three", 3)
        numbersMapShorthand += mapOf("four" to 4, "five" to 5)
        println(numbersMapShorthand)
    }
    ```

- Remove
    - `remove` 를 활용하여 데이터 삭제 가능
    - `remove` 를 활용하여, key값을 삭제하는 것도 remove와 동일한 역할
    - `remove` 를 활용하여, value값을 삭제하면 첫번째로 검색되는 값이 삭제 됨

    ```kotlin
    fun sampleCodeForRemoveOperator() {
        val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
        numbersMap.remove("one")
        println(numbersMap)
        numbersMap.remove("three", 4)           //doesn't remove anything
        println(numbersMap)

        val numbersMapRemoveElement = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)
        numbersMapRemoveElement.keys.remove("one")
        println(numbersMapRemoveElement)
        numbersMapRemoveElement.values.remove(3)
        println(numbersMapRemoveElement)

        val numbersMapMinusOperator = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
        numbersMapMinusOperator -= "two"
        println(numbersMapMinusOperator)
        numbersMapMinusOperator -= "five"            //doesn't remove anything
        println(numbersMapMinusOperator)
    }
    ```

출처 : [https://kotlinlang.org/docs/map-operations.html#plus-and-minus-operators](https://kotlinlang.org/docs/map-operations.html#plus-and-minus-operators)