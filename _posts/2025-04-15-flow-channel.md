---
title: Kotlin - Channel     
date: 2025-04-15
categories:
 - Android
tags:
 - Android
 - Kotlin
 - Coroutine
 - Flow
 - Channel 
---

Coroutine 에서 자주 쓰이는 Channel 에 대해서 알아봅니다. 

<!-- more -->

### Send / Receive suspend 발생 이유

<aside>
💡 Channel 이 LinkedList 기반의 queue 를 사용하여 송, 수신자를 관리하기 때문
</aside>

- 송신자 ( sender )
  - send 는 데이터를 채널에 보냄
  - 채널이 가득 찼거나, 수신자가 없으면 송신자는 대기열에 추가되고 suspend 상태가 됨
  - 수신자가 데이터를 받을 준비가 되면, 송신자가 활성화 됨
- 수신자 ( receiver )
  - receive는 채널에더 데이터를 가져옴
  - 채널에 데이터가 없거나, 송신자가 데이터를 보내지 않았다면 대기열에 추가되고 suspend 상태가 됨
  - 송신자가 데이터를 보내면, 수신자는 다시 활성화 됨

### TrySend vs Send

<aside>
💡 - trysend : non-blocking 함수, 채널이 가득 차있으면 즉시 실패
💡 - send : suspend 함수, receiver 가 준비되지 않은 경우 suspend 됨
</aside>

- send
  - suspend 함수, 채널이 가득 차거나 소비자가 준비되지 않은 경우 일시 중단 됨
  - 채널이 닫혀있으면, `ClosedSendChannelException` 을 던짐
- trysend
  - non-blocking 함수로, 채널이 가득 차거나 닫혀 있는경우 즉시 실패
  - 반환값이 ChannelResult 로, 성공 여부를 나타냄

      ```kotlin
      val channel = Channel<Int>(capacity = 1)
      val result = channel.trySend(1) // 비차단 호출
      if (result.isSuccess) {
          println("성공적으로 전송됨")
      } else {
          println("전송 실패: ${result.exceptionOrNull()}")
      }
      ```


### RENDEZVOUS & Suspend

<aside>
💡 버퍼가 없으며, 송신자와 수신자가 동시에 만나야 데이터가 전달됨

</aside>

- 랑데부, 기본 용량이 0
- 사용 시기
  - 생산자 - 소비자가 동시에 만나야 하는 경우
  - 데이터가 즉시 처리되어야 하며, 버퍼링이 필요 없는 상황
  - 생산자 - 소비자 패턴에서, 데이터 손실 없이 동기화가 필요 한 경우
- 주의점
  - 생산자 - 소비자가 동시에 준비되지 않으면 suspend 발생
  - 높은 동시성이 요구되는 환경에서는 성능 저하 가능
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val channel = Channel<Int>(Channel.RENDEZVOUS)
    
        val startTime = System.currentTimeMillis()
    
        launch {
            println("Sending 1 at ${System.currentTimeMillis() - startTime}ms")
            channel.send(1) // 수신자가 없으면 일시 중단
            println("Sent 1 at ${System.currentTimeMillis() - startTime}ms")
        }
    
        launch {
            delay(1000) // 송신자가 기다리게 만듦
            println("Receiving at ${System.currentTimeMillis() - startTime}ms")
            println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms")
        }
    }
    
    // 출력 결과 
    Sending 1 at 0ms
    Receiving at 1000ms
    Received: 1 at 1000ms
    Sent 1 at 1000ms
    ```

- 사용 예시
  - 택배 배달 시스템에서 배달원이 패키지를 즉시 수령해야 하는 경우.
  - 패키지가 준비되면 배달원이 즉시 수령해야 하며, 배달원이 없으면 패키지는 대기 상태가 됨
  - 주의점 : 배달원이 없으면 패키지가 계속 대기하므로, 처리 속도가 느리면 병목 현상이 발생할 수 있음.
  - 예시 코드

      ```kotlin
      import kotlinx.coroutines.*
      import kotlinx.coroutines.channels.*
      
      fun main() = runBlocking {
          val channel = Channel<String>(Channel.RENDEZVOUS)
      
          launch {
              println("Package ready at ${System.currentTimeMillis()}ms")
              channel.send("Package 1")
              println("Package sent at ${System.currentTimeMillis()}ms")
          }
      
          launch {
              delay(1000) // 배달원이 늦게 도착
              println("Delivery person ready at ${System.currentTimeMillis()}ms")
              println("Received: ${channel.receive()} at ${System.currentTimeMillis()}ms")
          }
      }
      
      // 출력 결과 
      Package ready at 0ms
      Delivery person ready at 1000ms
      Received: Package 1 at 1000ms
      Package sent at 1000ms
      ```


### BUFFERED & Suspend

<aside>
💡 고정 크기의 버퍼를 가지며, 버퍼가 가득 차면 송신자가 대기

</aside>

- 사용 시기
  - 고정 크기의 버퍼를 사용하여, 데이터 흐름을 제어하고 싶을 때
  - 송신자와 수신자의 처리 속도가 다른 경우
  - 데이터가 일정량 까지는 버퍼링 되어도 괜찮은 경우
- 주의점
  - 버퍼 크기를 잘못 설정하면, 메모리 낭비 / 송신자 대기 발생
  - 버퍼가 가득 찬 상태에선, 송신자가 suspend 가능
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val channel = Channel<Int>(capacity = 2) // 버퍼 크기 2
        val startTime = System.currentTimeMillis()
    
        launch {
            channel.send(1)
            println("Sent 1 at ${System.currentTimeMillis() - startTime}ms")
            channel.send(2)
            println("Sent 2 at ${System.currentTimeMillis() - startTime}ms")
            channel.send(3) // 버퍼가 가득 차서 대기
            println("Sent 3 at ${System.currentTimeMillis() - startTime}ms")
        }
    
        launch {
            delay(1000)
            println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms")
            delay(1000)
            println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms")
            delay(1000)
            println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms")
        }
    }
    
    // 출력 
    Sent 1 at 0ms
    Sent 2 at 0ms
    Received: 1 at 1000ms
    Sent 3 at 1000ms
    Received: 2 at 2000ms
    Received: 3 at 3000ms
    ```

- 사용 예시
  - 로그 시스템에서 로그 메시지를 버퍼링하여 비동기적으로 저장.
  - 로그 메시지가 빠르게 생성되지만, 저장 속도가 느릴 경우 버퍼를 사용해 메시지를 임시 저장.
  - 주의점 : 버퍼 크기를 초과하면 송신자가 대기하므로, 적절한 버퍼 크기를 설정해야 함
  - 예시 코드

      ```kotlin
      import kotlinx.coroutines.*
      import kotlinx.coroutines.channels.*
      
      fun main() = runBlocking {
          val channel = Channel<String>(capacity = 2)
      
          launch {
              repeat(5) {
                  println("Log message $it sent at ${System.currentTimeMillis()}ms")
                  channel.send("Log message $it")
              }
          }
      
          launch {
              repeat(5) {
                  delay(1000) // 로그 저장 속도
                  println("Saved: ${channel.receive()} at ${System.currentTimeMillis()}ms")
              }
          }
      }
      
      // 출력 결과 
      Log message 0 sent at 0ms
      Log message 1 sent at 0ms
      Log message 2 sent at 0ms
      Saved: Log message 0 at 1000ms
      Saved: Log message 1 at 2000ms
      Saved: Log message 2 at 3000ms
      Saved: Log message 3 at 4000ms
      Saved: Log message 4 at 5000ms
      ```


### CONFLATED & Suspend

<aside>
💡 마지막으로 보낸 데이터만 유지하며, 이전 데이터는 덮어씌워짐

</aside>

- 사용 시기
  - 최신 데이터만 중요하고, 이전 데이터는 무시해도 되는 경우
  - 상태 업데이트와 같이 마지막 값만 필요 한 상황에 활용
- 주의점
  - 이전 데이터가 덮어씌워지므로, 데이터 손실 가능
  - 모든 데이터 처리할땐 부적합
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val channel = Channel<Int>(Channel.CONFLATED)
        val startTime = System.currentTimeMillis()
    
        launch {
            channel.send(1)
            println("Sent 1 at ${System.currentTimeMillis() - startTime}ms")
            channel.send(2) // 이전 데이터(1)는 덮어씌워짐
            println("Sent 2 at ${System.currentTimeMillis() - startTime}ms")
        }
    
        launch {
            delay(1000)
            println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms") // 2만 수신
        }
    }
    
    // 결과 
    Sent 1 at 0ms
    Sent 2 at 0ms
    Received: 2 at 1000ms
    ```

- 사용 예시
  - 주식 가격 업데이트 시스템에서 최신 가격만 유지.
  - 주식 가격이 자주 변경되지만, 사용자에게는 최신 가격만 중요할 때 사용.
  - 주의점 : 이전 가격 데이터는 덮어씌워지므로, 모든 데이터를 처리해야 하는 경우에는 부적합.
  - 예시 코드

      ```kotlin
      import kotlinx.coroutines.*
      import kotlinx.coroutines.channels.*
      
      fun main() = runBlocking {
          val channel = Channel<Int>(Channel.CONFLATED)
      
          launch {
              println("Sending 1 at ${System.currentTimeMillis()}ms")
              channel.send(1)
              println("Sending 2 at ${System.currentTimeMillis()}ms")
              channel.send(2) // 이전 데이터(1)는 덮어씌워짐
              println("Sending 3 at ${System.currentTimeMillis()}ms")
              channel.send(3) // 이전 데이터(2)는 덮어씌워짐
          }
      
          launch {
              delay(1000) // 수신 지연
              println("Receiving at ${System.currentTimeMillis()}ms")
              println("Received: ${channel.receive()} at ${System.currentTimeMillis()}ms")
          }
      }
      
      // 출력 결과 
      Sending 1 at 0ms
      Sending 2 at 0ms
      Sending 3 at 0ms
      Receiving at 1000ms
      Received: 3 at 1000ms
      ```


### UNLIMITED & Suspend

<aside>
💡 무제한 버퍼를 가지며, 메모리가 허용하는 한 데이터를 계속 저장할 수 있음

</aside>

- 사용 시기
  - 데이터 손실 없이 모든 데이터를 버퍼링 해야 하는 경우
  - 송신자 / 수신자의 처리 속도가 크게 다를 때
- 주의점
  - 메모리 사용량이 제한되지 않으므로, 메모리 부족 위험
  - 메모리 누수 가능성 존재
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val channel = Channel<Int>(Channel.UNLIMITED)
        val startTime = System.currentTimeMillis()
    
        launch {
            for (i in 1..5) {
                channel.send(i)
                println("Sent: $i at ${System.currentTimeMillis() - startTime}ms")
            }
        }
    
        launch {
            delay(2000)
            for (i in 1..5) {
                println("Received: ${channel.receive()} at ${System.currentTimeMillis() - startTime}ms")
            }
        }
    }
    
    // 출력 결과 
    Sent: 1 at 0ms
    Sent: 2 at 0ms
    Sent: 3 at 0ms
    Sent: 4 at 0ms
    Sent: 5 at 0ms
    Received: 1 at 2000ms
    Received: 2 at 2000ms
    Received: 3 at 2000ms
    Received: 4 at 2000ms
    Received: 5 at 2000ms
    ```

- 사용 예시
  - 대규모 데이터 처리 시스템에서 모든 데이터를 손실 없이 처리.
  - 데이터가 빠르게 생성되지만, 처리 속도가 느릴 경우 모든 데이터를 버퍼링하여 손실을 방지.
  - 주의점: 메모리 사용량이 제한되지 않으므로 메모리 부족 위험이 있음
  - 예시 코드

      ```kotlin
      import kotlinx.coroutines.*
      import kotlinx.coroutines.channels.*
      
      fun main() = runBlocking {
          val channel = Channel<Int>(Channel.UNLIMITED)
      
          launch {
              repeat(5) {
                  println("Sending $it at ${System.currentTimeMillis()}ms")
                  channel.send(it)
              }
          }
      
          launch {
              repeat(5) {
                  delay(1000) // 데이터 처리 속도
                  println("Received: ${channel.receive()} at ${System.currentTimeMillis()}ms")
              }
          }
      }
      
      // 출력 결과 
      Sending 0 at 0ms
      Sending 1 at 0ms
      Sending 2 at 0ms
      Sending 3 at 0ms
      Sending 4 at 0ms
      Received: 0 at 1000ms
      Received: 1 at 2000ms
      Received: 2 at 3000ms
      Received: 3 at 4000ms
      Received: 4 at 5000ms
      ```


### Drop Oldest / Drop Latest

<aside>
💡 - Drop Oldest : 버퍼가 가득 찼을 때, 가장 오래된 요소를 삭제, 새로운 요소를 추가합니다.
💡 - Drop Latest : 가장 최근에 들어온 항목 제거
</aside>

- DROP_OLDEST
  - 버퍼가 가득 찼을 때, 가장 오래된 요소를 삭제, 새로운 요소를 추가
  - 장점
    - 최신 데이터를 유지 할 수 있음
    - 오래된 데이터가 중요하지 않은 경우 적합
  - 단점
    - 오래된 데이터가 삭제되므로, 데이터의 순서나 과거 데이터가 중요한 경우 부적합
  - 사용 예시
    - 실시간 주식 가격 업데이트: 주식 가격은 최신 정보가 중요하므로 오래된 데이터를 삭제하고 최신 데이터를 유지
    - IoT 센서 데이터: 센서에서 지속적으로 데이터를 수집할 때, 최신 상태만 필요하다면 적합
  - 사용하면 안되는 경우
    - 과거 데이터가 중요하거나, 데이터의 순서가 중요한 경우 (예: 로그 데이터 처리)
- DROP_LATEST
  - 버퍼가 가득 찼을 때, 새로 추가하려는 요소를 삭제하고 기존 버퍼를 유지
  - 장점
    - 기존 데이터를 보존할 수 있음
    - 데이터의 순서가 중요한 경우 적합
  - 단점
    - 최신 데이터가 손실될 수 있음
    - 실시간성이 중요한 경우 부적합
  - 사용 예시
    - 로그 데이터 처리: 로그는 순서가 중요하므로 기존 데이터를 유지하고 새 데이터를 삭제하는 것이 적합
    - 배치 처리 시스템: 데이터가 순차적으로 처리되어야 하는 경우.
  - 사용하면 안되는 경우
    - 최신 데이터가 중요하거나, 실시간 처리가 필요한 경우(예: 실시간 알림 시스템).
- 예시 코드

    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    /*
     * 실험 1: Rendezvous 채널 (capacity = RENDEZVOUS, DROP_OLDEST)
     *   - 기본 버퍼가 없으므로 receiver가 준비되기 전에는 sender의 send()가 suspend 됩니다.
     *
     * 실험 2: Buffered 채널 (capacity = 2, DROP_LATEST)
     *   - 버퍼가 가득 찰 경우 새로 send된 값은 DROP되므로 send()는 suspend 되지 않습니다.
     *
     * 실험 3: Unlimited 채널 (capacity = UNLIMITED, DROP_OLDEST 옵션은 무시됨)
     *   - 무제한 버퍼이므로 모든 값이 저장되고 send()는 suspend 되지 않습니다.
     *
     * 실험 4: Conflated 채널 (capacity = CONFLATED)
     *   - 항상 최신 값 하나만 보관하므로 send()는 suspend 되지 않고, receiver는 마지막 값만 받게 됩니다.
     *
     * 실험 5: Buffered 채널 (capacity = 2, 기본 동작 - SUSPEND)
     *   - onBufferOverflow 옵션 미지정 시, 버퍼가 꽉 차면 send()가 suspend되어 receiver가 처리할 때까지 대기합니다.
     */
    
    fun main() = runBlocking {
        println("=== 실험 시작 ===\n")
    
        // ---------------------------
        // Experiment 1: Rendezvous Channel with DROP_OLDEST
        // ---------------------------
        println("Experiment 1: Rendezvous Channel with DROP_OLDEST")
        val rendezvousChannel = Channel<Int>(
            capacity = Channel.RENDEZVOUS, 
            onBufferOverflow = BufferOverflow.DROP_OLDEST
        )
    
        launch {
            repeat(3) { i ->
                val sendStart = System.currentTimeMillis()
                println("Rendezvous: about to send $i at ${System.currentTimeMillis()} ms")
                // receiver가 준비되지 않으면 send()에서 suspend됨
                rendezvousChannel.send(i)
                val sendDuration = System.currentTimeMillis() - sendStart
                println("Rendezvous: Sent $i (send() suspending time: $sendDuration ms)")
            }
            rendezvousChannel.close()
        }
    
        delay(100) // sender가 suspend 되는 상황 유도 (모든 delay 100ms)
        launch {
            // for 루프 실행 전에 시작시간을 단 한 번 기록
            val recvStart = System.currentTimeMillis()
            for (value in rendezvousChannel) {
                delay(100)  // 각 값 처리 시 100ms delay
                val processingTime = System.currentTimeMillis() - recvStart
                println("Rendezvous: Received $value (processing delay: $processingTime ms)")
            }
        }
    
        delay(100) // Experiment 1 종료 대기
    
        // ---------------------------
        // Experiment 2: Buffered Channel (capacity=2) with DROP_LATEST
        // ---------------------------
        println("\nExperiment 2: Buffered Channel (capacity=2) with DROP_LATEST")
        val bufferedChannel = Channel<Int>(
            capacity = 2,
            onBufferOverflow = BufferOverflow.DROP_LATEST
        )
    
        launch {
            repeat(5) { i ->
                delay(100)  // 각 send 전 간격을 100ms로 통일
                val sendStart = System.currentTimeMillis()
                println("Buffered: about to send $i at ${System.currentTimeMillis()} ms")
                bufferedChannel.send(i)
                val sendDuration = System.currentTimeMillis() - sendStart
                println("Buffered: Sent $i (send() suspending time: $sendDuration ms)")
            }
            bufferedChannel.close()
        }
    
        launch {
            val recvStart = System.currentTimeMillis() // for 루프 실행 전 단 한 번 기록
            for (value in bufferedChannel) {
                delay(100)  // 각 값 처리 시 100ms delay
                val processingTime = System.currentTimeMillis() - recvStart
                println("Buffered: Received $value (processing delay: $processingTime ms)")
            }
        }
    
        delay(100) // Experiment 2 종료 대기
    
        // ---------------------------
        // Experiment 3: Unlimited Channel with DROP_OLDEST (drop policy ignored)
        // ---------------------------
        println("\nExperiment 3: Unlimited Channel with DROP_OLDEST (drop policy ignored)")
        val unlimitedChannel = Channel<Int>(
            capacity = Channel.UNLIMITED,
            onBufferOverflow = BufferOverflow.DROP_OLDEST
        )
    
        launch {
            repeat(5) { i ->
                delay(100)
                val sendStart = System.currentTimeMillis()
                println("Unlimited: about to send $i at ${System.currentTimeMillis()} ms")
                unlimitedChannel.send(i)
                val sendDuration = System.currentTimeMillis() - sendStart
                println("Unlimited: Sent $i (send() suspending time: $sendDuration ms)")
            }
            unlimitedChannel.close()
        }
    
        launch {
            val recvStart = System.currentTimeMillis() // for 루프 실행 전 단 한 번 기록
            for (value in unlimitedChannel) {
                delay(100)
                val processingTime = System.currentTimeMillis() - recvStart
                println("Unlimited: Received $value (processing delay: $processingTime ms)")
            }
        }
    
        delay(100) // Experiment 3 종료 대기
    
        // ---------------------------
        // Experiment 4: Conflated Channel
        // ---------------------------
        println("\nExperiment 4: Conflated Channel")
        val conflatedChannel = Channel<Int>(capacity = Channel.CONFLATED)
    
        launch {
            repeat(5) { i ->
                delay(100)
                val sendStart = System.currentTimeMillis()
                println("Conflated: about to send $i at ${System.currentTimeMillis()} ms")
                conflatedChannel.send(i)
                val sendDuration = System.currentTimeMillis() - sendStart
                println("Conflated: Sent $i (send() suspending time: $sendDuration ms)")
            }
            conflatedChannel.close()
        }
    
        launch {
            val recvStart = System.currentTimeMillis() // for 루프 실행 전 단 한 번 기록
            for (value in conflatedChannel) {
                delay(100)
                val processingTime = System.currentTimeMillis() - recvStart
                println("Conflated: Received $value (processing delay: $processingTime ms)")
            }
        }
    
        delay(100) // Experiment 4 종료 대기
    
        // ---------------------------
        // Experiment 5: Buffered Channel with SUSPEND (default behavior)
        // ---------------------------
        println("\nExperiment 5: Buffered Channel with SUSPEND (default behavior)")
        val suspendChannel = Channel<Int>(capacity = 2)  // 기본 동작: 버퍼가 꽉 차면 send()가 suspend 됨
    
        launch {
            repeat(5) { i ->
                delay(100)
                val sendStart = System.currentTimeMillis()
                println("Suspend: about to send $i at ${System.currentTimeMillis()} ms")
                // 버퍼가 꽉 차면 이 시점에서 send()가 suspend되어 receiver가 처리할 때까지 대기
                suspendChannel.send(i)
                val sendDuration = System.currentTimeMillis() - sendStart
                println("Suspend: Sent $i (send() suspending time: $sendDuration ms)")
            }
            suspendChannel.close()
        }
    
        launch {
            val recvStart = System.currentTimeMillis() // for 루프 실행 전 단 한 번 기록
            for (value in suspendChannel) {
                delay(100)
                val processingTime = System.currentTimeMillis() - recvStart
                println("Suspend: Received $value (processing delay: $processingTime ms)")
            }
        }
    
        delay(100) // Experiment 5 종료 대기
    
        println("\n=== 모든 실험 완료 ===")
    }
    
    // 출력 결과 
    === 실험 시작 ===
    
    Experiment 1: Rendezvous Channel with DROP_OLDEST
    Rendezvous: about to send 0 at 1672500000000 ms
    Rendezvous: Sent 0 (send() suspending time: 105 ms)
    Rendezvous: about to send 1 at 1672500000105 ms
    Rendezvous: Sent 1 (send() suspending time: 100 ms)
    Rendezvous: about to send 2 at 1672500000205 ms
    Rendezvous: Sent 2 (send() suspending time: 100 ms)
    Rendezvous: Received 0 (processing delay: 205 ms)
    Rendezvous: Received 1 (processing delay: 305 ms)
    Rendezvous: Received 2 (processing delay: 405 ms)
    
    Experiment 2: Buffered Channel (capacity=2) with DROP_LATEST
    Buffered: about to send 0 at 1672500001000 ms
    Buffered: Sent 0 (send() suspending time: 0 ms)
    Buffered: about to send 1 at 1672500001100 ms
    Buffered: Sent 1 (send() suspending time: 0 ms)
    Buffered: about to send 2 at 1672500001200 ms
    Buffered: Sent 2 (send() suspending time: 0 ms)
    Buffered: about to send 3 at 1672500001300 ms
    Buffered: Sent 3 (send() suspending time: 0 ms)
    Buffered: about to send 4 at 1672500001400 ms
    Buffered: Sent 4 (send() suspending time: 0 ms)
    Buffered: Received 0 (processing delay: 300 ms)
    Buffered: Received 1 (processing delay: 400 ms)
    
    Experiment 3: Unlimited Channel with DROP_OLDEST (drop policy ignored)
    Unlimited: about to send 0 at 1672500002000 ms
    Unlimited: Sent 0 (send() suspending time: 0 ms)
    Unlimited: about to send 1 at 1672500002100 ms
    Unlimited: Sent 1 (send() suspending time: 0 ms)
    Unlimited: about to send 2 at 1672500002200 ms
    Unlimited: Sent 2 (send() suspending time: 0 ms)
    Unlimited: about to send 3 at 1672500002300 ms
    Unlimited: Sent 3 (send() suspending time: 0 ms)
    Unlimited: about to send 4 at 1672500002400 ms
    Unlimited: Sent 4 (send() suspending time: 0 ms)
    Unlimited: Received 0 (processing delay: 200 ms)
    Unlimited: Received 1 (processing delay: 300 ms)
    Unlimited: Received 2 (processing delay: 400 ms)
    Unlimited: Received 3 (processing delay: 500 ms)
    Unlimited: Received 4 (processing delay: 600 ms)
    
    Experiment 4: Conflated Channel
    Conflated: about to send 0 at 1672500003000 ms
    Conflated: Sent 0 (send() suspending time: 0 ms)
    Conflated: about to send 1 at 1672500003100 ms
    Conflated: Sent 1 (send() suspending time: 0 ms)
    Conflated: about to send 2 at 1672500003200 ms
    Conflated: Sent 2 (send() suspending time: 0 ms)
    Conflated: about to send 3 at 1672500003300 ms
    Conflated: Sent 3 (send() suspending time: 0 ms)
    Conflated: about to send 4 at 1672500003400 ms
    Conflated: Sent 4 (send() suspending time: 0 ms)
    Conflated: Received 4 (processing delay: 400 ms)
    
    Experiment 5: Buffered Channel with SUSPEND (default behavior)
    Suspend: about to send 0 at 1672500004000 ms
    Suspend: Sent 0 (send() suspending time: 0 ms)
    Suspend: about to send 1 at 1672500004100 ms
    Suspend: Sent 1 (send() suspending time: 0 ms)
    Suspend: about to send 2 at 1672500004200 ms
    Suspend: Sent 2 (send() suspending time: 110 ms)
    Suspend: about to send 3 at 1672500004310 ms
    Suspend: Sent 3 (send() suspending time: 100 ms)
    Suspend: about to send 4 at 1672500004410 ms
    Suspend: Sent 4 (send() suspending time: 100 ms)
    Suspend: Received 0 (processing delay: 200 ms)
    Suspend: Received 1 (processing delay: 300 ms)
    Suspend: Received 2 (processing delay: 400 ms)
    Suspend: Received 3 (processing delay: 500 ms)
    Suspend: Received 4 (processing delay: 600 ms)
    
    === 모든 실험 완료 ===
    
    ```