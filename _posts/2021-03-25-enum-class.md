---
title: Kotlin Enum class
date: 2021-03-25
categories:
 - Android
tags:
 - Kotlin
---

Kotlin의 Enum class의 용도, 사용법 등에 대해서 알아봅니다. 

[Github repo](https://github.com/lu-lay/kotlin_study/tree/kangraemin/kangraemin/src/enumclass) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Enum class Default Method

### Enum Class

- 열거형을 다룰때 주로 사용하는 데이터 타입입니다.
- 쓰이는 쓰임새는 JAVA의 Enum과 크게 다르지는 않습니다.

### 아래에서 사용할 Enum 클래스

```kotlin
// JAVA
enum Animal {
    DUCK, PENGUIN, CAT, DOG
}

// Kotlin
enum class Animal {
    DUCK, PENGUIN, CAT, DOG
}
```

### 기본적인 Enum class property / method

- JAVA에서 제공하는 `name`, `ordinal` 등의 함수도 그대로 제공 해 줍니다.
- Kotlin의 `when` 은 Java의 `switch` 문법과 닮은 용도로 사용되고 있습니다.

```kotlin
// Kotlin 
fun sampleCodeForDefaultMethodInEnum() {
    // Return enum property's name
    println(CAT.name) // CAT

    // Return enum property's position in enum class
    println(CAT.ordinal) // 2

    // Get enum property by name
    println(valueOf("DUCK")) // DUCK
    // println(Animal.valueOf("DUCKY")) // throw IllegalArgumentException error ( no enum constants )

    // Return values to arrayList
    println(
        values()
            .map { "You are ${it.name} in position ${it.ordinal} !" }
            .toList()
    )
}
```

- Kotlin에서 `when`과 enum을 함께 활용 할 때와, Java에서 `switch`와 enum을 함께 활용할 때 IDE에 따라 warning을 다르게 보여 줄 수 있습니다.

```kotlin
// enum 에서 어떤 값이 분기처리가 되지 않았는지 확인 할 수 있음
// 임의의 enum property 에 대한 예외처리를 하지않았을 때 아래의 warning 문구가 표출이 됨
// 'when' expression on enum is recommended to be exhaustive, add 'PENGUIN' branch or 'else' branch instead
when (values().random()) {
    DUCK -> {
        println("DUCK")
    }
    CAT -> {
        println("CAT")
    }
    DOG -> {
        println("DOG")
    }
}
```

```java
// PENGUIN에 대한 case가 없음에도 불구하고 switch에 warning을 표출시켜 주지 않음
int randomIndex = new Random().nextInt(4);
Animal animal = Animal.values()[randomIndex];
switch (animal) {
    case CAT -> System.out.println("CAT !");
    case DOG -> System.out.println("DOG !");
    case DUCK -> System.out.println("DUCK !");
}
```

### Enum with constructor

- enum class의 생성자를 활용하여, enum 상수들의 값들을 선언 해 줄 수 있습니다.

```kotlin
enum class AnimalWithName(val cryingSound: String, val age: Int) {
    DUCK("Quack", 123),
    PENGUIN("Peng", 12),
    CAT("Meow", 1),
    DOG("Bow", 31)
}

fun sampleCodeForEnumWithConstructor() {
    println(AnimalWithName.CAT.age) // 1
    println(AnimalWithName.CAT.cryingSound) // Meow
}
```

### Enum with Abstract function / property

- enum class 안에, abstract function / property 을 정의하여 각 enum 상수들 안에 property / function의 구현체를 강제 할 수 있습니다.

```kotlin
fun sampleCodeForEnumWithAbstractClass() {
    AnimalWithAbstractFunction.CAT.cry() // Meow
    println(AnimalWithAbstractFunction.DOG.age) // 31 
}

enum class AnimalWithAbstractFunction() {
    DUCK {
        override val age: Int
            get() = 123

        override fun cry() {
            println("Quack")
        }
    },
    PENGUIN {
        override val age: Int
            get() = 12

        override fun cry() {
            println("Peng")
        }
    },
    CAT {
        override val age: Int
            get() = 1

        override fun cry() {
            println("Meow")
        }
    },
    DOG {
        override val age: Int
            get() = 31

        override fun cry() {
            println("Bow")
        }
    };
    abstract fun cry()
    abstract val age: Int
}
```

### Enum with Interface

- enum class에 interface를 구현하여 각 enum 상수들 안에 property / function의 구현체를 강제 할 수 있습니다.

```kotlin
fun sampleCodeWithInterface() {
    AnimalWithInterface.CAT.cry() // Meow
    println(AnimalWithAbstractFunction.CAT.age) // 1
}

interface NormalAnimal {
    val age: Int
    fun cry()
}

enum class AnimalWithInterface: NormalAnimal {
    DUCK {
        override val age: Int
            get() = 123

        override fun cry() {
            println("Quack")
        }
    },
    PENGUIN {
        override val age: Int
            get() = 12

        override fun cry() {
            println("Peng")
        }
    },
    CAT {
        override val age: Int
            get() = 1

        override fun cry() {
            println("Meow")
        }
    },
    DOG {
        override val age: Int
            get() = 31

        override fun cry() {
            println("Bow")
        }
    },
}
```

### Enum class with Companion object

- Enum 클래스 안에 companion object를 활용하여 각 enum class에 활용할 property / function을 사용 할 수 있습니다.

```kotlin
fun sampleCodeForEnumWithNameAndCompanionObject() {
    println("oldest animal = ${AnimalWithNameAndCompanionObject.getOldestAnimalName()}")
}

enum class AnimalWithNameAndCompanionObject(val cryingSound: String, val age: Int) {
    DUCK("Quack", 123),
    PENGUIN("Peng", 12),
    CAT("Meow", 1),
    DOG("Bow", 31);

    companion object {
        fun getOldestAnimalName(): String {
            println(values().toList())
            return values().maxWithOrNull(compareBy { it.age })?.name ?: "none !"
        }
    }
}
```


