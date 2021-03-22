---
title: Kotlin / Java에서의 추상화를 하는 이유 ( Inteface / Abstract Class )
date: 2021-03-23
categories:
 - Android
tags:
 - Kotlin
 - Collection
---

Java, Kotlin에서, 대부분은 한번 쯤 interface / abstract class ( 추상 클래스 )라는 말을 들어 본 적이 있을 것입니다. 이 둘은 왜 쓰는 것이고, 어떨 때 사용하는지 알아보도록 합시다. 


[Github repo](https://github.com/kangraemin/Tech_blog_Kotlin_example_code/tree/master/src/playground/abstraction) 에서 아래에 적힌 Kotlin 코드들을 확인 하실 수 있습니다. 

<!-- more -->

## 상황 가정

먼저 아래의 상황을 가정 해 봅시다. 

- 교실에는 선생님과 학생이 있습니다.
- 학생은 학생 마다 청소 담당 구역이 배정되어 있습니다.
- 선생님이 학생을 호출하면, 학생은 담당 구역을 청소를 시작 합니다.
- 청소 담당 구역은 `창문` / `바닥` / `칠판` 이 있습니다.
- 창문 청소는 `창문 닦기` 로, 바닥은 `바닥 쓸기` 로, 칠판은 `칠판 닦기` 로 청소를 진행합니다.

## Class로만 구현

### 구현

위 상황 속에서, 선생님이 학생에게 청소를 시키는 경우를 class만 가지고 구현 해 봅시다. 

- Teacher, Student 클래스를 생성 해 줍니다.
- 청소 담당 구역인 학생, 창문 담당 구역인 학생, 칠판 담당 구역인 학생에 대한 클래스를 각각 생성 해 줍니다.
- 학생들을 생성 해 주고, 학생들에게 청소를 시킵니다.

    ```kotlin
    package playground.abstraction.onlyclass

    class Teacher {
        fun sayStudentName(student: Student) {
            print("Hey ! ${student.name}! ")
        }

        fun makeStudentToClean(student: Student) {
            when (student) {
                is StudentForFloorCleaning -> {
                    println("바닥을 쓸거라")
                }
                is StudentForWindowCleaning -> {
                    println("창문을 닦거라")
                }
                is StudentForBlackboardCleaning -> {
                    println("칠판을 닦거라")
                }
            }
        }
    }
    ```

    ```kotlin
    package playground.abstraction.onlyclass

    open class Student(val name: String)

    class StudentForFloorCleaning(name: String) : Student(name) {
        fun startCleanUp() {
            println("$name : 바닥 쓸기 시작하겠습니다 !")
        }
    }
    class StudentForWindowCleaning(name: String) : Student(name) {
        fun startCleanUp() {
            println("$name : 창문 닦기 시작하겠습니다 !")
        }
    }
    class StudentForBlackboardCleaning(name: String) : Student(name) {
        fun startCleanUp() {
            println("$name : 칠판 닦기 시작하겠습니다 !")
        }
    }
    ```

    ```kotlin
    package playground.abstraction.onlyclass

    fun main() {
        // 선생님 생성
        val teacher = Teacher()

        // 학생 collection 생성
        val floorStudents = mutableListOf<StudentForFloorCleaning>()
        val windowStudents = mutableListOf<StudentForWindowCleaning>()
        val blackboardStudents = mutableListOf<StudentForBlackboardCleaning>()

        for (number in 1..5) {
            // 바닥 쓸기 학생 생성
            floorStudents.add(StudentForFloorCleaning("바닥 쓸기 학생 $number"))
            // 창문 닦이 학생 생성
            windowStudents.add(StudentForWindowCleaning("창문 닦이 학생 $number"))
            // 칠판 닦이 학생 생성
            blackboardStudents.add(StudentForBlackboardCleaning("칠판 닦이 학생 $number"))
        }

        // 바닥 쓸기 학생 청소
        floorStudents.forEach {
            teacher.sayStudentName(it)
            teacher.makeStudentToClean(it)
            it.startCleanUp()
        }

        // 창문 닦이 학생 청소
        windowStudents.forEach {
            teacher.sayStudentName(it)
            teacher.makeStudentToClean(it)
            it.startCleanUp()
        }

        // 칠판 닦이 학생 청소
        blackboardStudents.forEach {
            teacher.sayStudentName(it)
            teacher.makeStudentToClean(it)
            it.startCleanUp()
        }
    }
    ```

### 위 코드의 문제점

- 이 때, 새로운 청소구역 `복도` 가 들어오게 되면, 추가되어야 할 코드는 아래와 같습니다
    - 복도 닦이 클래스 생성 부분

        ```kotlin
        class StudentForHallwayCleaning(name: String) : Student(name) {
            fun cleanUpHallway() {
                println("$name : 복도 닦기 시작하겠습니다 !")
            }
        }
        ```

    - `makeStudentToClean` 메소드에 복도 닦이 케이스 추가 부분

        ```kotlin
        fun makeStudentToClean(student: Student) {
            print("Hey ! ${student.name}! ")
            when (student) {
                is StudentForFloorCleaning -> {
                    student.cleanUpFloor()
                }
                is StudentForWindowCleaning -> {
                    student.cleanUpWindow()
                }
                is StudentForBlackboardCleaning -> {
                    student.cleanUpBlackboard()
                }
        				// 추가 된 부분
                is StudentForHallwayCleaning -> {
                    student.cleanUpHallway()
                }
            }
        }
        ```

    - 복도 닦이 학생 추가 로직, 복도 청소당번 collection에서 선생님이 청소 시키는 로직, 청소를 시작하는 로직 부분

        ```kotlin
        // 복도 닦이 collection 생성
        val hallwayStudents = mutableListOf<StudentForHallwayCleaning>()

        for (number in 1..5) {
        		// 복도 닦이 학생 생성
        		hallwayStudents.add(StudentForHallwayCleaning("복도 닦이 학생 $number"))
        }

        // 복도 닦이 학생 청소
        hallwayStudents.forEach {
            teacher.makeStudentToClean(it)
        }
        ```

- 즉, 청소 구역을 하나 추가 할 뿐인데 추가 해야 할 코드 양도 많으며, 하나라도 제대로 추가되지 않으면 정상적으로 동작하지 않습니다.
- 또, 해당 복도 청소를 추가했다가 복도 구역의 이름을 `교실외` 로 변경하는 경우, 변경해야 할 코드의 양이 매우 많습니다.
- 만약, 청소 구역이 100개가 넘는 경우, `makeStudentToClean` 메소드가 매우 복잡해져 유지보수 하는데 힘이 들 수 있습니다.
- 만약, 청소에 대한 결과가 제대로 출력되지 않은 경우에, 개발자인 우리가 살펴보아야 할 곳은 Teacher 클래스의 `makeStudentToClean` 메소드와, 각 학생 클래스에 구현된 청소 메소드들 모두를 살펴 보아야 합니다.
- 이는 청소에 대한 결과가 Teacher 클래스의 `makeStudentToClean` 메소드에도 책임 소재가 있음을 의미하며, 책임 소재가 여러군데에 나뉘어 져 있는 코드는 유지보수가 쉽지 않습니다.

### 해결책

- 위 코드를 크게 나누어 보면, `청소구역이 배정된 학생 collection을 생성하는 부분` / `collection에 청소구역이 배정된 학생을 추가하는 부분` / `청소구역이 배정된 학생들에게 청소를 시키는 부분`으로 나눌 수 있습니다.
- 즉, `청소 배정된 학생` 이 겹치는 부분이며, 각 청소의 구체적인 방법 ( 칠판 닦기, 바닥 쓸기 ... 등 )만 다른 것을 확인 할 수 있습니다.
- 위 코드 상에서, `학생`이 `청소 배정된 학생` 이 되기 위해선 `청소`를 하는 부분이 필요합니다.
- 즉, 학생 클래스에 `청소 배정된 학생` 이 되기위한 자격인 `청소` 함수를 각 청소 구역에 알맞게 구현만 해 주면, 훨씬 유지보수가 쉬울 것 같습니다.
- 즉, 청소 구역별로 클래스를 만들어 따로 청소 함수를 구현하지 말고,  `청소 배정된 학생` 이라는 공통점으로 묶어 클래스를 관리한다면, `청소 배정된 학생.청소` 의 형식으로 청소를 시킬 수 있습니다.
- `창문 닦이 학생` / `칠판 닦이 학생` 등의 클래스에서, `청소 배정된 학생` , `청소` 라는 공통적인 의미를 뽑아 냈 듯이, 이렇게 여러 객체에서 공통적으로 사용되거나, 사용되어야 하는 내용을 뽑아 내는 것을 `추상화` 작업이라고 지칭합니다.
- 이를 위해 Java / Kotlin 에서는 Interface / Abstract class를 주로 활용합니다.

## Interface 추상화 활용

### Interface

- Java ( 출처 : [Oracle 공식 문서](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html) ) / Kotlin ( 출처 : [Kotlin 공식 문서](https://kotlinlang.org/docs/classes.html#abstract-classes) )에서 Interface는 아래와 같이 정의 되어 있습니다.

```
// JAVA
In the Java programming language, an interface is a reference type, similar to a class, that can contain only constants, method signatures, default methods, static methods, and nested types

// Kotlin 
Interfaces in Kotlin can contain declarations of abstract methods, as well as method implementations.
```

- 여기서 말하는 abstract 키워드는, 해당 interface를 구현하고 있는 클래스는 반드시 구현 해야 할 property / method를 의미합니다.
- 즉, abstract가 붙은 method는, 해당 interface를 구현하고 있는 클래스가 반드시 구현 해야 할 method 입니다.
- 일반적으로 Java / Kotlin에서 method ( property ) 앞에 abstract를 생략하여 사용하는 경우가 대부분이라 interface 안에 있는 method에 아무 것도 붙어있지 않다면, 해당 method ( property )는 abstract method ( property )로 간주됩니다.
- JAVA의 예시

    ```java
    interface ExampleInterface {
    		void exampleMethod();
    } 
    ```

    ```java
    // ExampleClass는 ExampleInerface를 구현한 클래스로 선언 
    // 하지만, exampleMethod를 구현하지 않았으므로 아래 사진과 같은 compile error 발생
    class ExampleClass implements ExampleInterface {

    }
    ```

    ![pic1.png](/assets/images/posts/2021-03-23-abstraction/pic1.png)

    ```java
    interface ExampleInterface {
        void exampleMethod();
    }

    // exampleMethod를 구현 해 주었으므로 compile error가 사라지고 정상 동작 
    class ExampleClass implements ExampleInterface {
        @Override
        public void exampleMethod() {
            
        }
    }
    ```

- Kotlin의 예시
    - Kotlin도 자바와 마찬가지로, 구현하지 않을시 compile error가 발생 합니다.

    ```java
    interface ExampleInterface {
        fun exampleMethod()
    }

    class ExampleClass : ExampleInterface {
        override fun exampleMethod() {}
    }
    ```

    ![pic2.png](/assets/images/posts/2021-03-23-abstraction/pic2.png)

- 즉, Interface의 함수 구현을 강제한다는 특성 때문에, 해당 interface를 구현 한 클래스는 반드시 abstract method ( property )를 구현 하고 있음을 알 수 있습니다.
- 쉽게 이해 하기 위해선, interface는 자격증, abstract method ( property )는 자격증을 따기위한 필수 조건 이라고 생각 하면 쉽습니다.
- 해당 interface를 구현하고 있는 클래스는, 해당 자격증을 취득 하기 위한 클래스들이며, 해당 자격증을 취득하기 위해 interface에 있는 모든 abstract method ( property )를 구현 해야 합니다.
- 모든 abstract method( property )를 구현 한다면, 해당 클래스는 자격증 ( interface )의 모든 자격 요건을 갖춘 것이라 볼 수 있습니다.
- 에를들어 응급 구조사, CPR, 구급대원에 대한 관계를 interface / class로 표시하면 아래와 같습니다.

    ```kotlin
    package playground.abstraction.useinterface

    interface 응급구조사 {
        fun CPR()
    }

    class 구급대원 : 응급구조사 {
        override fun CPR() {
            println("구급대원이 CPR을 시행합니다.")
        }
    }

    fun main() {
        val 구급대원: 응급구조사 = 구급대원()
        구급대원.CPR()
    }
    ```

    - `응급구조사` 는 `CPR` 를 할 수 있어야 합니다.
    - `구급 대원`이 `응급 구조사`가 되기 위해선, `CPR 하는 방법`을 `구급대원` 대로 익혀야 합니다. ( 아래의 코드에선, CPR에 대한 코드 구현이 있으므로 구급 대원이 CPR 하는 방법을 익힌 상태입니다. )
    - 구급 대원이 응급 구조사 interface를 구현하고 있어 CPR을 할 수 있으므로, `응급 구조사`가 필요한 곳에,  을 투입 할 수 있습니다.

### 구현

- 다시 학생 이야기로 돌아와서, 위에서 반복적인 작업들을 피하기 위해 `청소 배정된 학생`, `청소` 라는 공통적인 개념을 뽑아 냈습니다.
- 이때, `청소 배정된 학생 (청소 당번)`은 반드시 `청소`를 할 수 있어야 합니다.
- 또, `청소 배정된 학생 (청소 당번)` 은 반드시 `이름`이 있어야 합니다.
- 따라서, 이를 interface로 추출하면 아래와 같습니다.

    ```kotlin
    package playground.abstraction.useinterface

    interface CleaningCrew {
        val name: String
        fun startCleanUp()
    }
    ```

- CleanCrew( 청소 당번 )를 구현한 청소 담당 구역인 학생, 창문 담당 구역인 학생, 칠판 담당 구역인 학생에 대한 클래스를 각각 생성 해 줍니다.

    ```kotlin
    package playground.abstraction.useinterface

    class StudentForFloorCleaning(studentName: String) : CleaningCrew {
        override val name: String = studentName
        override fun startCleanUp() {
            println("$name : 바닥 쓸기 시작하겠습니다 !")
        }
    }

    class StudentForWindowCleaning(studentName: String) : CleaningCrew {
        override val name: String = studentName
        override fun startCleanUp() {
            println("$name : 창문 닦기 시작하겠습니다 !")
        }
    }

    class StudentForBlackboardCleaning(studentName: String) : CleaningCrew {
        override val name: String = studentName
        override fun startCleanUp() {
            println("$name : 칠판 닦기 시작하겠습니다 !")
        }
    }
    ```

    - 각 청소구역을 맡은 청소 당번 학생들은, 반드시 청소( startCleanUp ) 하는 방법과, 이름을 정하는 방법을 구현 해야 하고, 구현한 형태는 위와 같습니다.
- Teacher 클래스를 생성 해 줍니다.

    ```kotlin
    package playground.abstraction.useinterface

    class Teacher {
        fun makeStudentToClean(student: CleaningCrew) {
            println("Hey ! ${student.name}!")
            student.startCleanUp()
        }
    }
    ```

    - `makeStudentToClean` 메소드를 보면, 이제 학생의 청소 구역에 따라 청소를 위한 method 호출이 다르지 않은 것을 확인 할 수 있습니다.
    - 만약, 청소에 대한 결과가 제대로 출력되지 않은 경우에 Teacher 클래스의 `makeStudentToClean` 메소드를 더이상 볼 필요가 없어졌습니다.
    - 모든 청소에 대한 책임은 Student 클래스 에게로 전가되었습니다.
- 학생들을 생성 해 주고, 학생들에게 청소를 시킵니다.

    ```kotlin
    package playground.abstraction.useinterface

    fun main() {
        // 선생님 생성
        val teacher = Teacher()

        // 학생 collection 생성
        val cleaningCrewCollection = mutableListOf<CleaningCrew>()

        for (number in 1..5) {
            // 바닥 쓸기 학생 생성
            cleaningCrewCollection.add(StudentForFloorCleaning("바닥 쓸기 학생 $number"))
            // 창문 닦이 학생 생성
            cleaningCrewCollection.add(StudentForWindowCleaning("창문 닦이 학생 $number"))
            // 칠판 닦이 학생 생성
            cleaningCrewCollection.add(StudentForBlackboardCleaning("칠판 닦이 학생 $number"))
        }

        // 학생 청소
        cleaningCrewCollection.forEach {
            teacher.makeStudentToClean(it)
        }
    }
    ```

### 앞의 코드보다 나아진 점

- 위에서 설명 했듯, Teacher 클래스에 더이상 청소에 대한 책임 소재가 없습니다.
- 만약 복도 청소 당번을 추가 한다고 했을때, 추가되어야 할 코드는 아래와 같습니다
    - 복도 닦이 학생 추가 로직

    ```kotlin

    for (number in 1..5) {
    		// 복도 닦이 학생 생성
    		cleaningCrewCollection.add(StudentForHallWayCleaning("복도 닦이 학생 $number"))
    }

    ```

    - 복도 닦이 클래스 생성 부분

    ```kotlin
    class StudentForHallWayCleaning(studentName: String) : CleaningCrew {
        override val name: String = studentName
        override fun startCleanUp() {
            println("$name : 복도 닦기 시작하겠습니다 !")
        }
    }
    ```

    - 다시 보자면, 앞서 복도 닦는 당번을 추가 할 때 필요했던 부분은 아래와 같지만, 공통적인 부분 추출 ( 추상화 ) 작업으로 인해 수정 해야 할 부분이 상당히 줄어 든 것을 확인 할 수 있습니다.
        - 복도 닦이 클래스 생성 부분
        - `makeStudentToClean` 메소드에 복도 닦이 케이스 추가 부분 ( 공통적인 interface 사용으로 필요 x )
        - 복도 닦이 학생 추가 로직 추가 부분
        - 복도 청소당번 collection에서 선생님이 청소 시키는 로직 추가 부분 ( 공통적인 interface collection 사용으로 필요 x )
        - 청소를 시작하는 로직 부분 추가 부분 ( 공통적인 interface 사용으로 필요 x )
