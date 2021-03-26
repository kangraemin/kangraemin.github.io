---
title: Kotlin Sealed class 
date: 2021-03-26
categories:
 - Android
tags:
 - Kotlin
---

Kotlin의 Sealed class의 용도, 사용법 등에 대해서 알아봅니다. 

[Github repo](https://github.com/lu-lay/kotlin_study/tree/kangraemin/kangraemin/src/sealedclass) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Sealed Class

- 봉인된 클래스, 클래스들을 묶은 클래스 입니다.
- enum class의 확장형, enum class의 특수한 형태이라고 생각하면 이해하기 쉽습니다.

### Sealed class의 특징

- Kotlin 코드를 컴파일 할 때, Sealed 클래스를 상속받은 클래스들이 어떤 것들이 있는지 모두 알려 집니다.
- 컴파일 이후에, 해당 sealed 클래스를 상속받는 클래스는 존재하지 않습니다.
- 따라서, 컴파일 이후 해당 sealed 클래스의 subclass들이 어떤 것들이 있는지 모두 명확하게 알 수 있습니다.
- enum과 용도가 비슷하지만, enum의 property들은 모두 single instance 이지만, sealed class의 property들은 클래스 이기 때문에 multiple instance를 지원 합니다.
- Kotlin의 `when` 키워드와 함께 사용할 때 매우 강력합니다.
- sealed class 자체는 abstract class의 속성을 갖고 있습니다. ( 본인 객체 생성 불가, abstract property 소유 가능 )

### Sealed class의 기본 형태

- sealed class를 생성하고, 해당 sealed class로 묶을 클래스들에 sealed class를 상속 해 줍니다.
- class를 상속 할 수 있는 형태라면 모두 sealed class의 subclass로 활용 가능 합니다. ( class / object / data class )

```kotlin
sealed class SealedThrowable : Throwable()
open class IOException : SealedThrowable()
class InvalidParameter : SealedThrowable()
class Forbidden : SealedThrowable()
object InternalServerError : SealedThrowable()
data class DataClassThrowable(val throwable: Throwable, val responseCode: Int) : SealedThrowable()
```

- 아래와 같은 형태로도 서술 할 수 있습니다.

```kotlin
sealed class Animal {
    class Duck : Animal()
    data class Dog(val name: String) : Animal()
    object Penguin : Animal()
}
```

### Sealed class를 사용할 때 주의점

- sealed class의 subclass를 sealed class를 직접 상속 받은 direct subclass / sealed class를 상속받은 subclass를 상속받은 indirect subclass 로 나눌 수 있습니다.
- direct subclass는, 반드시 sealed class와 같은 파일에 서술되어 있어야 합니다. 그렇지 않으면 compile error를 발생 시킵니다.
- indirect subclass는 sealed class와 같은 파일에 서술되어 있지 않아도 됩니다.

```kotlin
/*
* SealedClass.kt file 
* */
sealed class SealedThrowable : Throwable()
open class IOException : SealedThrowable()
```

```kotlin
/*
* OtherKotlinFile.kt file 
* */
object OtherThrowable: IOException()
// Compile error !
// object AnotherThrowable: SealedThrowable()
```

### Sealed class와 when 활용

- `when`을 활용한 sealed class의 분기처리 예시

```kotlin
fun sampleCodeForHandlingThrowable() {
    handleError(InternalServerError) // has InternalServerError !
}

fun handleError(sealedThrowable: SealedThrowable) = when (sealedThrowable) {
    is IOException -> println("IOException")
    is InvalidParameter -> println("InvalidParameter")
    is Forbidden -> println("Forbidden")
    InternalServerError -> println("has InternalServerError !")
    is DataClassThrowable -> {
        println(sealedThrowable.responseCode)
        sealedThrowable.throwable.printStackTrace()
    }
}
```

- sealed class의 subclass들은, compile시에 어떤 subclass가 있는지 모두 알 수 있습니다.
- 따라서, when을 활용하여 sealed class의 subclass 마다 작업을 분리 해 줄 경우, 임의의 subclass에 대한 분기 처리가 되어 있지 않을 시 compile error를 띄워줍니다.

```kotlin
fun sampleCodeForHandlingThrowable() {
    handleError(InternalServerError)
}

// Forbidden에 대한 분기 처리가 이루어 지지 않았기 때문에 Compile error를 발생
fun handleError(sealedThrowable: SealedThrowable) = when (sealedThrowable) {
    is IOException -> println("IOException")
    is InvalidParameter -> println("InvalidParameter")
    // is Forbidden -> println("Forbidden")
    InternalServerError -> println("InternalServerError")
    is DataClassThrowable -> {
        println(sealedThrowable.responseCode)
        sealedThrowable.throwable.printStackTrace()
    }
}
```

- 위와 같은 특성 때문에, sealed class에 subclass가 늘어 났을 때, 추가된 경우에 대해 대비하는것을 잊지 않을 확률이 매우 높아 버그를 발생 시킬 확률을 많이 낮출 수 있습니다.
