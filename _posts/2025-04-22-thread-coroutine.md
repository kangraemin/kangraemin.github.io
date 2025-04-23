---
title: Kotlin - Thread vs Coroutine      
date: 2025-04-22
categories:
 - Android
tags:
 - Android
 - Kotlin
 - Coroutine
---

동시성 작업을 할 때 Coroutine 과 Thread 의 차이점을 알아봅니다.       

<!-- more -->

### JVM - Thread vs Coroutine

<aside>
💡Coroutine은 경량화된 멀티 태스킹 도구
</aside>

- JVM Thread
  - JVM 에서 실행 되는 각 쓰레드는, 보통 운영체제가 관리하는 실제 스레드에 매핑 됨
  - 각 JVM 쓰레드는, 자기만의 Stack ( 메모리 공간 ) / PC 레지스터 등 독립적인 Context 를 가짐
  - 일반적으로, 하나의 JVM 쓰레드는 수십만 ~ 수백만 바이트에 달하는 스택 메모리를 필요로 함 → 많은 수의 쓰레드를 생성하면 메모리 사용량과 Context switch 오버헤드가 크게 증가
- Coroutine
  - 코루틴은 실제 OS 스레드 “처럼” 행동하는 “작업 단위”
  - 코루틴은 새로운 스레드를 새로 생성 하지 않고, 기존 스레드 풀 위에서 협력적으로 실행 됨
  - suspend / resume 을 State machine / Continuation 등으로 구현 → 자신의 상태를 별도의 스택 없이도, 힙에 저장하여 나중에 이어서 실행 가능
- Thread 보다 Coroutine 의 장점
  - 낮은 메모리 오버헤드
    - OS / JVM Thread 는 각각 독립된 스택 / 자원을 가짐
    - 코루틴은, 상태 정보를 저장하는 비교적 작은 Continuation 객체를 힙에 할당 함
  - 협력적 스케쥴링
    - 코루틴은 CPU 를 활용 하다가, 자신이 suspend 될 때 명시적으로 실행 제어를 양도함
    - 이는 thread context switching 보다 훨씬 가볍고 빠른 방식
  - 스레드 풀 기반
    - 코루틴은 보통 한정된 수의 실제 JVM 쓰레드 위에서 다수의 코루틴이 동시에 실행 됨
    - 이를 통해, 동시에 수천개의 코루틴을 실행 할 수 있지만, 실제 OS 쓰레드 수는 적음
    - 따라서, 메모리와 컨텍스트 전환 오버헤드를 크게 줄일 수 있음
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.Job
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.launch
    import kotlinx.coroutines.runBlocking
    import kotlin.system.measureTimeMillis
    
    fun threadExample() {
        // 10,000개의 작업을 위해 스레드를 생성
        val threads = mutableListOf<Thread>()
        repeat(10_000) { i ->
            val thread = Thread {
                try {
                    Thread.sleep(10)
                } catch (e: InterruptedException) {
                    e.printStackTrace()
                }
            }
            threads.add(thread)
            thread.start()
        }
        threads.forEach { it.join() }
    }
    
    fun coroutineExample() = runBlocking {
        val jobs = mutableListOf<Job>()
        repeat(10_000) { i ->
            val job = launch {
                delay(10)
            }
            jobs.add(job)
        }
        jobs.forEach { it.join() }
    }
    
    fun main() {
        val threadTime = measureTimeMillis {
            threadExample()
        }
        println("스레드 방식 실행 시간: $threadTime ms")
    
        val coroutineTime = measureTimeMillis {
            coroutineExample()
        }
        println("코루틴 방식 실행 시간: $coroutineTime ms")
    }
    
    // 10000번 반복 후 출력 결과 
    스레드 방식 실행 시간: 420 ms
    코루틴 방식 실행 시간: 131 ms
    
    // 10000 * 10 번 반복 후 출력 결과 
    스레드 방식 실행 시간: 3880 ms
    코루틴 방식 실행 시간: 206 ms
    ```

  - 10000개의 작업 진행, 각 작업마다 10ms 씩 sleep
  - 스레드 방식
    - 모든 스레드가 생성되어 동시에 실행되지만, 각 스레드는 독립적인 스택과 실행 컨텍스트를 가지므로 스레드 생성 / 시작 / 스케쥴링에 많은 오버헤드 발생
  - 코루틴 방식
    - 스레드를 생성해서 작업을 진행 하는 것 보다, 훨씬 더 빠른 결과 생성
  - 만약 횟수를 10000이 아니라, 100000 으로 10배 더 많이 시행한다면 차이는 더 심함
  - 동시 작업을 처리 할 때, 스레드 생성 방식보다 왜 코루틴이 우세 한지를 볼 수 있음
- [다음 글](https://kangraemin.github.io/android/2025/04/23/coroutine-continuation/) 에서 이렇게 코루틴이 Thread 에 비해 가벼운 이유를 coroutine 의 동작 원리를 살펴보며 알아보겠습니다. 