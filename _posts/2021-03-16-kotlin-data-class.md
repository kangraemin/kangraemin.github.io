---
title: Kotlin data class
date: 2021-03-16
categories:
 - Android
tags:
 - Kotlin
---

POJO 클래스와 비교하며 Kotlin의 data class에 대해 알아봅니다.

[Github repo](https://github.com/lu-lay/kotlin_study/tree/kangraemin/kangraemin/src/dataclass) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## Java Class For Data

### POJO Class

- 자바에서는, Data를 담아두기 위한 용도로 POJO 클래스 ( Plain Old Java Object )를 자주 활용합니다.

#### * 아래에서 사용할 POJO 예시

```java
public class PojoClass {
    String textData;
    int numberData;
    public String getTextData() {
        return textData;
    }

    public void setTextData(String textData) {
        this.textData = textData;
    }

    public int getNumberData() {
        return numberData;
    }

    public void setNumberData(int numberData) {
        this.numberData = numberData;
    }
}
```

### * POJO 클래스에서 개선하면 좋을 요소들

- 하지만 위의 예시 처럼 POJO 클래스를 정의하여 사용한다면 아래의 문제점들이 있습니다. 

#### 1. 데이터를 표현하는 `toString()` 메소드의 결과값이 명확한 의미를 전달하지 않습니다.  

```java
public class JavaMain {
    public static void main(String[] args) {
        PojoClass pojoClass = new PojoClass();
        pojoClass.setNumberData(100);
        pojoClass.setTextData("text");

        System.out.println(pojoClass); // PojoClass@6b884d57
    }
}
```

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

- 위의 코드에서, pojoClass를 콘솔 출력 했을 때 ( String으로 변환 했을 때 ), `toString()` 함수를 통해 변환 됩니다.
- 하지만, String 클래스에 정의된 `toString()` 함수는, 클래스의 이름과 해당 객체의 hash 값으로 이루어 져 있어 해당 객체에 대한 정보 ( 아래의 예에선 numberData / textData 에 대한 정보 )를 유추 할 수 없습니다. 

#### 2. 두 객체의 동등성 비교에 사용될 비교 기준이 명확하지 않습니다.

```java
public class JavaMain {
    public static void main(String[] args) {
        PojoClass pojoClass = new PojoClass();
        pojoClass.setNumberData(100);
        pojoClass.setTextData("text");

        PojoClass otherPojoClass = new PojoClass();
        otherPojoClass.setNumberData(100);
        otherPojoClass.setTextData("text");

        System.out.println(pojoClass.toString()); // PojoClass@1be6f5c3        
        System.out.println(otherPojoClass); // PojoClass@6b884d57
        System.out.println("is it equal two instance = " + (pojoClass == otherPojoClass)); // false 
        System.out.println("is it equal two instance = " + (pojoClass.equals(otherPojoClass))); // false
    }
}
```

- 위에서 생성한 `pojoClass` 객체와, `otherPojoClass` 객체는 객체 속 데이터들이 모두 동일합니다. 
- 하지만, `==`, `equals()`를 활용한 동등성 비교에서 두 객체의 내부 property 값들이 모두 같다는 것을 확인 할 수 없는 문제점이 있습니다.
- 참고 : 자바의 `==` 연산자 / `equals()` 연산자 
    - 자바 `==` 연산
        - 원시형 타입 끼리의 `==` 연산인 경우, 두 객체의 값이 동일 한 지 검사합니다.
        - 참조형 타입 끼리의 `==` 연산인 경우, 두 객체의 주소 값이 같은지 검사합니다. 
        - 따라서 위 예제에서, pojoClass == otherPojoClass 는 두 객체의 주소 값이 다르므로 false를 리턴합니다.
    - 자바 `equals()` 연산
        - 참조형 타입의 비교에서는 `==` 연산자가, 두 객체의 주소 값이 같은지를 검사하고 값이 같은지를 보장 하지 않으므로 `equals()` 메소드를 활용하여 두 객체의 값이 같은지를 검사합니다. 
        - String 끼리의 비교에서도 값이 같음을 비교 할 때에는 `equals()` 메소드를 활용하는 이유도 이와 같습니다.
        - `equals()` 메소드는 overiding 될 수 있으며, 일반 클래스에서 overiding 되지 않았을 시 object 클래스에 있는 `equals` 메소드를 그대로 사용합니다.

        ```java
        // Object class
        public boolean equals(Object obj) {
                return (this == obj);
        }
        ```

        - 위 예제를 살펴보면, PojoClass 는 `equals()` 메소드를 overiding 하지 않았습니다.
        - 따라서 `pojoClass.equals(otherPojoClass)` 는 object의 `equals`를 그대로 사용하므로 false를 리턴합니다. 
    - 따라서, PojoClass의 두 객체에 있는 모든 값이 같은지를 검사하려면, `equals` 함수를 overide 해서 사용해야 합니다.
- 코틀린의 `==` / `equals()`
    - 코틀린 `==` 연산
        - 코틀린에서 `==` 연산은, 내부적으로 `equals` 를 호출하여 객체의 값을 비교 합니다.
    - 코틀린 `===` 연산
        - 참조형 변수 끼리의 주소값 비교를 위한 자바의 `==` 와 같은 역할 입니다. 
        - 객체의 주소값이 같은 지를 확인 하고 싶을 때 사용 합니다.

```java
import java.util.Objects;

public class PojoClassEqual extends PojoClass {
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PojoClassEqual pojoClass = (PojoClassEqual) o;
        return numberData == pojoClass.numberData && Objects.equals(textData, pojoClass.textData);
    }
}
```

```java
public class JavaMain {
    public static void main(String[] args) {
        PojoClassEqual pojoClassEqual = new PojoClassEqual();
        pojoClassEqual.setNumberData(100);
        pojoClassEqual.setTextData("text");

        PojoClassEqual otherPojoClassEqual = new PojoClassEqual();
        otherPojoClassEqual.setNumberData(100);
        otherPojoClassEqual.setTextData("text");

        System.out.println(pojoClassEqual.toString());
        System.out.println(otherPojoClassEqual);
        System.out.println("is it equal two instance = " + (pojoClassEqual == otherPojoClassEqual)); // false
        System.out.println("is it equal two instance = " + (pojoClassEqual.equals(otherPojoClassEqual))); // true
    }
}
```

- 위 경우, `equals()` 가 정의 되어 있고, 객체의 타입과 객체 속 모든 값이 같다면 true를 반환 하기 때문에, `pojoClassEqaul` 객체와, `otherPojoClassEqaul` 객체는 동일하다고 판단합니다.

#### 3. `hashCode()` 함수에 대한 구현이 정의되어 있지 않습니다.

- 위 `equals()` 함수를 정의한 클래스에서, `hashCode()` 함수는 정의 되지 않았습니다.
- `hashCode()` 함수가 정의 되지 않아, hash함수를 먼저 비교하는 `hashSet` 등의 클래스에서 해당 객체를 다룰 때 문제가 있을 수 있습니다.
- hashSet 클래스에서 contains 메소드는, 대입되는 객체의 hashCode 값이 hashSet 안의 hashMap의 key 값으로 있는지 부터 진행 합니다. 
- 즉, 두 객체의 hashCode 값이 같지 않으면, false를 리턴 합니다.
- 이때, pojoEqual 객체와, otherPojoEqual 객체의 hash값이 다르므로 contains 메소드에선 false값을 리턴 합니다.
- 따라서, hashCode 메소드도 반드시 overide 하여 재정의 해 주어야 합니다.

```java
public class PojoClassEqualHashCode extends PojoClassEqual {
    @Override
    public int hashCode() {
        return textData.hashCode() * 31;
    }
}
```

```java
import java.util.HashSet;

public class JavaMain {
    public static void main(String[] args) {
        PojoClassEqualHashCode pojoClassEqualHashCode = new PojoClassEqualHashCode();
        pojoClassEqualHashCode.setNumberData(100);
        pojoClassEqualHashCode.setTextData("text");

        PojoClassEqualHashCode otherPojoClassEqualHashCode = new PojoClassEqualHashCode();
        otherPojoClassEqualHashCode.setNumberData(100);
        otherPojoClassEqualHashCode.setTextData("text");

        HashSet<PojoClassEqualHashCode> hashSetWithHashCode = new HashSet<>();
        hashSetWithHashCode.add(otherPojoClassEqualHashCode);
        System.out.println(hashSetWithHashCode.contains(otherPojoClassEqualHashCode)); // true
    }
}
```

### Data Class

- Data를 갖고 있기 위한 코틀린 클래스 입니다. 
- Data를 위한 기본적인 기능, 함수들을 제공 해 줍니다.

#### * DataClass를 생성하기 위해선 아래의 조건을 반드시 따라야 합니다.

- 주 생성자에는, 반드시 하나 이상의 parameter가 들어가 있어야 합니다.
- 주 생성자의 모든 parameter들은, `val` 또는 `var` 로 선언 되어야 합니다.
- DataClass는 `abstract`, `open`, `sealed`, `inner` 클래스가 될 수 없습니다.

#### * 예시에 쓰일 Data class 

```kotlin
data class User(val name: String, var age: Int)

val user = User(name = "rams", age = 29)
```

- 위 처럼 data class를 선언 시, 아래의 함수들을 기본적으로 제공 해 줍니다.

#### 1. Property Get / Set 

- 위와 같이 data class를 정의 했을 때, age / name 데이터를 get / set 하는 메소드를 따로 정의하지 않아도 직관적으로 바로 사용 가능 합니다. 

```kotlin
// Get property 확인
println("----Get property 확인----")
println(user.age) // 29
println(user.name) // rams
```

```kotlin
// Set property 확인
println("----Set property 확인----")
user.age = 30 // val 로 선언된 property 에서는 Compile error ( var / val 차이 )
println(user) // User(name=rams, age=30)
user.age = 29
println(user) // User(name=rams, age=29)
```

#### 2. 좀 더 직관적인 형태의 toString 메소드 제공

- POJO 클래스 객체의 toString 값이 클래스 이름 + hashCode 값이 출력되는것과는 달리, 객체의 값을 확인 할 수 있도록 자동으로 toString 함수를 구현 해 줍니다.

```kotlin
// toString 함수 확인
println("----toString 함수 확인----")
println(user) // User(name=rams, age=29)
println(user.toString()) // User(name=rams, age=29)
```

#### 3. 좀 더 직관적인 형태의 equals 메소드 제공 

- Java 클래스에서는, 따로 equals를 overide하여 정의하지 않을 시, object 클래스에 있는 equals 메소드를 사용 합니다. 
- POJO 클래스 객체 끼리의 동등성 비교를 할 때엔, 객체 안의 값이 모두 같아도 false가 리턴 됩니다. 
- 하지만 data class에서는, 객체 안의 모든 값이 같으면, 두 객체는 동일하다고 판단 해주는 equals 함수를 구현 해 줍니다. 

```kotlin
// equals 함수 확인
println("----equals 함수 확인----")
val sameUser = User(age = 29, name = "rams")
val otherUser = User(age = 20, name = "rams")
println(user == sameUser) // true
println(user == otherUser) // false
println(user === sameUser) // false
```

#### 4. hashCode() 메소드 제공 

- Java 클래스 에서는, 따로 hashCode를 overide 하여 정의하지 않을 시, object 클래스에 있는 hashCode 메소드를 사용 하게 됩니다.
- 따라서, 같은 객체라 하더라도 hashCode의 값이 다를 수 있습니다. 
- 하지만 data class 에서는, 객체 안의 모든 값이 같으면, 동일한 hashCode 값을 리턴 하도록 hashCode 함수를 구현 해 줍니다. 

```kotlin
// hashCode 확인
println("----hashCode 확인----")
val hashSet = mutableSetOf<User>()
hashSet.add(user)
println(user.hashCode() == sameUser.hashCode()) // true
println(hashSet.contains(sameUser)) // true
```

#### 5. Component N 메소드 제공 

- Data class 안의 property 중 N번쨰 값을 리턴해주는 함수 입니다. 

```kotlin
// ComponentN 확인
println("----ComponentN 확인----")
println(user.component1()) // rams
println(user.component2()) // 29
```

#### 6. destructuring declarations

- data class 안의 property들을 한번에 꺼내어 변수에 할당 할 수 있습니다. 
- 내부적으로 componentN을 활용하여 꺼내기 때문에, 선언하는 변수들의 순서가 중요하게 됩니다. 

```kotlin
// destructuring declarations 확인
println("----destructuring declarations 확인----")
val (name, age) = user
println(name) // rams
println(age) // 29

val (ageTest) = user
println(ageTest) // rams

// Compile error
// val (name, age, error) = user
```

#### 7. copy 메소드 제공

- 생성된 data class 객체와 대부분의 값은 비슷한데, 특정 값만 바꾼 다른 객체를 생성 하고 싶을 때 주로 사용 합니다. 
- 복사 된 객체는, 원래의 객체와는 독립적인 객체가 됩니다. 

```kotlin
// copy 확인
println("----copy 확인----")
val copiedUser = user.copy(name = "RAMS")
val sameCopiedUser = user.copy()
println(copiedUser) // User(name=RAMS, age=29)
println(user == copiedUser) // false
println(user == sameCopiedUser) // true
println(user.hashCode() == sameCopiedUser.hashCode()) // true
println(user === sameCopiedUser) // false
```

#### 8. Inheritance

- data class에 클래스를 상속 시키고, 부모 클래스에서 `equals` / `hashCode` / `toString` 함수 등을 final 키워드를 붙여 override 하면, 부모 클래스에서 override한 함수를 그대로 사용 하게 됩니다.( 자동 생성 해주지 않아 원하는 동작을 하지 않을 수 있습니다. )

```kotlin
data class User(val name: String, var age: Int) : UserSuperClass()

open class UserSuperClass {
    final override fun equals(other: Any?): Boolean {
        return super.equals(other)
    }

    final override fun hashCode(): Int {
        return super.hashCode()
    }

    final override fun toString(): String {
        return super.toString()
    }
}
```

```kotlin
----Get property 확인----
29
rams
----Set property 확인----
User@2f2c9b19
User@2f2c9b19
----toString 함수 확인----
User@2f2c9b19
User@2f2c9b19
----equals 함수 확인----
false
false
false
----hashCode 확인----
false
false
----ComponentN 확인----
rams
29
----destructuring declarations 확인----
rams
29
rams
----copy 확인----
User@1c20c684
false
false
false
false
```

#### 9. Interface / Abstract class

- Data class에 Interface / Abstract class를 상속 하여 사용 할 수 있습니다. 

```kotlin
fun sampleCodeForInterface() {
    val duck = Duck("Donald", 123)
    duck.cry() // Quack

    val dog = Dog("Snoopy", 3000)
    dog.cry() // Bow

    val penguin = Penguin("Peng", 13)
    penguin.eat() // Nyam
    penguin.somethingNormal
}

abstract class AbstractAnimal {
    abstract fun cry()
}

interface Animal {
    fun cry()
}

open class AnimalClass {
    val somethingNormal = "somethingNormal"

    fun eat() {
        println("Nyam")
    }
}

data class Duck(val name: String, val age: Int): AbstractAnimal() {
    override fun cry() {
        println("Quack")
    }
}

data class Dog(val name: String, val age: Int): Animal {
    override fun cry() {
        println("Bow")
    }
}

data class Penguin(val name: String, val age: Int): AnimalClass() {

}

// 데이터 클래스 끼리의 상속은 불가능 합니다. ( open keyword 가 안붙음 )
data class ParentAnimal(val name: String, val age: Int)

// Function 'component1' generated for the data class conflicts with member of supertype 'ParentAnimal'
//data class Dog(val firstElement: String): ParentAnimal()
```