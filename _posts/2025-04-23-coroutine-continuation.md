---
title: Kotlin - 코루틴 동작 원리 ( Continuation / CPS / State Machine )      
date: 2025-04-23
categories:
 - Android
tags:
 - Android
 - Kotlin
 - Coroutine
---

Coroutine 이 어떤 방식 으로 suspend 를 구현 하는지 원리를 알아봅니다. 

<!-- more -->

### Continuation

<aside>
💡코루틴이 suspend 후 재개 될 때, 실행 상태와 결과를 저장하고 이어서 실행할 책임을 가진 인터페이스
</aside>

- 내부 코드

    ```kotlin
    public interface Continuation<in T> {
        public val context: CoroutineContext
        public fun resumeWith(result: Result<T>)
    }
    ```

  - 코루틴에서 suspend 이후, 나머지 실행을 이어갈 연속성을 나타내기 위한 interface
  - T: suspend 함수가 완료되었을 때 반환할 결과형입니다.
  - context → 코루틴의 실행 문맥 ( Coroutine context )를 포함하여, dispatcher / job 등을 제공
  - resumeWith(result: Result<T>) → 마지막 suspension 이후에 반환하거나, 예외를 던질 때 사용하는 메서드
- 역할 및 활용처
  - 상태 캡처 / 재개
    - suspend 함수가 중단 될 때, 로컬 변수 / 실행 위치 등 현재 실행 상태를 Continuation 객체에 저장 함
    - suspend 함수 재개 시, Continuation의 resumeWith() 메서드를 활용하여 중단된 상타에서 이어서 실행
  - 비동기 결과 전달
    - 비동기 작업의 결과를 콜백 방식으로 전달하는 역할을 진행
    - 네트워크 요청이 완료되어 결과를 전달 할 때, Continuation 의 resumeWith()를 호출하여 결과를 코루틴에 전달
  - 컨텍스트 전달
    - 각각의 Continuation은 자신의 CoroutineContext를 가지고 있어, 실행 환경 ( Dispatcher, Job 등 )을 관리하는데 활용

### Continuation Passing Style ( CPS )

<aside>
💡함수 실행 결과를 리턴 하지 않고, 다음에 해야 할 일을 나타내는 함수에 넘겨주는 방식
</aside>

- CPS
  - 함수 / 프로그램의 실행 흐름을 Continuation 이라는 형태로 전달하는 프로그래밍 스타일
  - 일반적인 함수 호출 방식과는 다르게, 함수가 결과를 직접 return 하는 대신에 다음에 실행할 작업을 매개변수로 받아 해당 작업을 함수의 끝에 호출하도록 만드는 방식
- 예시 코드

    ```kotlin
    // 직접 결과값 return 
    fun sum(a: Int, b: Int): Int = a + b 
    
    // continuation 에 넘겨 줌 
    fun sum(a: Int, b: Int, continuation: (Int) -> Unit : Int = continuation(a+b)
    ```


### State Machine

<aside>
💡시스템이 가질 수 있는 다양한 상태(state)와 그 상태 사이를 전이(transition)하는 규칙을 정의하여, 입력이나 이벤트에 따라 시스템이 어떻게 반응할지를 결정하는 모델
</aside>

- 상태 ( State )
  - 시스템이 특정 시점에 있을 수 있는 조건이나 상황을 나타 냄
  - 예를 들어, 전구의 상태는 "켜짐"과 "꺼짐" 두 가지 상태를 가질 수 있음
- 전이 ( Transition )
  - 한 상태에서 다른 상태로 이동하는 것을 의미 함
  - 예를 들어, 스위치를 누르면 전구가 "꺼짐"에서 "켜짐"으로 전이 됨
- 상태 머신 ( State Machine )
  - 이러한 상태와 전이를 체계적으로 정의하여, 입력 이벤트에 따라 시스템이 어떤 상태로 전이해야 하는지를 결정하는 모델
    - 일반적으로 상태 머신은 다음과 같은 요소를 포함
      - 상태 집합 : 가능한 모든 상태의 목록
      - 전이 규칙 : 각 상태에서 어떤 이벤트가 발생했을 때 어떤 상태로 전이할지에 대한 규칙
      - 초기 상태 : 시스템의 시작 상태
      - (선택적으로) **종료 상태**: 시스템이 종료되는 상태
- 코루틴에서 상태 머신
  - Kotlin의 suspend 함수는 여러 중단 가능 지점을 갖기 때문에, 컴파일러는 이를 상태 머신 패턴으로 변환함
  - suspend 함수는 내부적으로 다음과 같이 동작 함
    - 내부 상태 저장
      - suspend 함수는 여러 suspend 지점을 갖게 됨
      - 각 suspend 지점은 코드의 어느 부분에서 중단되었는지를 의미
    - label 필드
      - 컴파일러는 이를 위해 Continuation 객체 내에 `int label`과 같은 필드를 생성
      - 이 label 값은 현재 실행 상태를 나타내며, 예를 들어 label이 0이면 함수의 시작 상태, 1이면 첫 번째 suspend 지점 이후 상태 등을 나타냄
    - switch 문으로 분기
      - suspend 함수는 내부에 switch 문을 두어, label 값에 따라 실행을 재개할 때 어느 부분부터 코드를 계속 실행할지를 결정 함
  - 즉, 코루틴의 상태 머신은 suspend 함수가 중단될 때 자신의 현재 실행 상태(어느 suspend 지점에 있었는지)를 label 등 내부 상태값으로 저장하고, 재개할 때 그 상태에 따라 올바른 코드 분기로 진입하도록 하는 기법으로 활용 중

### Suspend 함수 Decompile 후 확인

<aside>
💡suspend 함수가 Continuation / State machine 을 활용한 CPS 로 변경 된 것을 확인 할 수 있음
</aside>

- 예시 코드

    ```kotlin
    suspend fun delayExample() {
        println("delay 이전 작업")
        delay(100)
        println("delay 이후 작업")
    }
    ```

  - 위 함수를 Decompile 하여 확인 해 보자
- Decompile 결과

    ```kotlin
    @Nullable
    public static final Object delayExample(@NotNull Continuation $completion) {
       Continuation $continuation;
       label20: {
          if ($completion instanceof <undefinedtype>) {
             $continuation = (<undefinedtype>)$completion;
             if (($continuation.label & Integer.MIN_VALUE) != 0) {
                $continuation.label -= Integer.MIN_VALUE;
                break label20;
             }
          }
    
          $continuation = new ContinuationImpl($completion) {
             // $FF: synthetic field
             Object result;
             int label;
    
             @Nullable
             public final Object invokeSuspend(@NotNull Object $result) {
                this.result = $result;
                this.label |= Integer.MIN_VALUE;
                return CoroutineTestKt.delayExample((Continuation)this);
             }
          };
       }
    
       Object $result = $continuation.result;
       Object var3 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
       switch ($continuation.label) {
          case 0:
             ResultKt.throwOnFailure($result);
             System.out.println("delay 이전 작업");
             $continuation.label = 1;
             if (DelayKt.delay(100L, $continuation) == var3) {
                return var3;
             }
             break;
          case 1:
             ResultKt.throwOnFailure($result);
             break;
          default:
             throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
       }
    
       System.out.println("delay 이후 작업");
       return Unit.INSTANCE;
    }
    ```

- Decompile 결과 코드 분석
  1. Continuation 객체 준비 ( CPS )

      ```kotlin
      Continuation $continuation;
      label20: {
         if ($completion instanceof <undefinedtype>) {
            $continuation = (<undefinedtype>)$completion;
            if (($continuation.label & Integer.MIN_VALUE) != 0) {
               $continuation.label -= Integer.MIN_VALUE;
               break label20;
            }
         }
      
         $continuation = new ContinuationImpl($completion) {
            // $FF: synthetic field
            Object result;
            int label;
      
            @Nullable
            public final Object invokeSuspend(@NotNull Object $result) {
               this.result = $result;
               this.label |= Integer.MIN_VALUE;
               return CoroutineTestKt.delayExample((Continuation)this);
            }
         };
      }
      ```

    - 코드 흐름
      - 함수의 인자로 전달된 `$completion`이 이미 suspend 함수용으로 생성된 ContinuationImpl인지 확인
      - 만약 그렇다면, 내부의 `label` 값에서 정지 표시(일반적으로 `Integer.MIN_VALUE` 비트, 즉 최상위 비트)를 제거하여 재개 상태로 전환
      - 그렇지 않으면, 새 ContinuationImpl 익명 클래스를 생성
        - 여기서, invokeSuspend 함수는 continuation 의 `resumeWith` 를 구현 한 것
    - Continuations와 label
      - 생성된 ContinuationImpl은 suspend 함수의 상태를 저장하는 필드로 `int label`을 사용
      - label 값은 여러 suspend 지점을 식별하기 위한 상태 값
      - 컴파일러는 suspend 함수가 중단되었다가 재개될 때, label을 변경하여 현재 진행할 위치를 추적
  2. 초기 상태와 suspend 지점 호출 ( State machine )

      ```kotlin
      Object $result = $continuation.result;
      Object var3 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
      switch ($continuation.label) {
         case 0:
            ResultKt.throwOnFailure($result);
            System.out.println("delay 이전 작업");
            $continuation.label = 1;
            if (DelayKt.delay(100L, $continuation) == var3) {
               return var3;
            }
            break;
         case 1:
            ResultKt.throwOnFailure($result);
            break;
         default:
            throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
      }
      ```

    - 코드 흐름
      - case 0
        - 이것은 suspend 함수가 처음 호출되었을 때의 상태
        - 먼저, `ResultKt.throwOnFailure($result)`를 호출하여 이전 실행 결과(혹은 예외 발생 여부)를 확인
        - "delay 이전 작업"을 출력
        - 그리고 `$continuation.label`을 1로 설정하여, 다음 재개 시점에서 바로 case 1로 진입하도록 함
        - 그 후, `DelayKt.delay(100L, $continuation)`를 호출
          - 이 delay 함수는 100ms 동안 실행을 중단(suspend)시키며, “delay 함수 구현체 내부적”으로는 비동기 작업이 완료될 때 Continuation의 `resumeWith()`를 호출
            - delay 함수 내부적으로, `resumeWith()` 를 부르는 곳은 찾을 수 없었음
            - 하지만 의사 코드만 작성 해 본다면, 아래의 형태 처럼 생각 해 볼 수 있음

                ```kotlin
                // 인터페이스 Delay의 메서드로 스케줄링을 담당하는 함수
                fun scheduleResumeAfterDelay(timeMillis: Long, continuation: CancellableContinuation<Unit>) {
                    // 예를 들어, Timer를 사용한다고 가정하면:
                    Timer().schedule(timeMillis) {
                        // 시간이 지난 후, Continuation의 resumeWith를 호출하여 코루틴을 재개함
                        continuation.resumeWith(Result.success(Unit))
                    }
                }
                ```

            - 이떄 `resumeWith()` 가 호출되면, 위에서 익명함수로 구현한 `ContinuationImpl` 의 `invokeSuspend` 가 호출되고, 이때 `delayExample` 함수가 재호출 됨
            - 따라서, 변경된 label 값을 가지고 함수가 재호출 되고, 다른 task 를 실행 하게 됨
          - delay가 완료되지 않았으면, 즉시 `COROUTINE_SUSPENDED` (var3)를 반환하고, 이 함수도 같은 값을 반환하여 suspend 상태로 종료
      - case 1
        - delay 함수 호출 후, 재개되어 label이 1인 상태에서 실행
        - 다시 ResultKt.throwOnFailure($result)를 통해 결과 또는 예외를 확인한 후, break하여 계속 실행
  3. 재개 후 최종 작업

      ```kotlin
      System.out.println("delay 이후 작업");
      return Unit.INSTANCE;
      ```

    - 코루틴이 재개되어 case 1로 들어오면, "delay 이후 작업"을 출력하고, 최종 결과(Unit.INSTANCE)를 반환
- 함수 Flow 정리
  1. `delayExample` 함수 호출
  2. Continuation 객체 있는지 확인 및 없다면 익명 클래스 객체 생성
  3. switch 문 실행 ( label = 0 )
  4. label 1로 변경
  5. delay 이전 작업 출력 및, delay 함수 실행
  6. delay 함수도 suspend 함수이므로, 해당 함수의 결과값을 받음
  7. 4에서 받은 결과값이 `COROUTINE_SUSPENDED` 값이라면, `COROUTINE_SUSPENDED` 리턴 하고 함수 실행 중단 ( suspend )
  8. delay 가 완료되어, delay 함수 내부에서 `resumeWith()` 가 호출 되면, `delayExample` 함수가 재호출
  9. label 이 1이기 때문에, switch 문에서 별 다른 작업 하지 않고 끝
  10. 이후 “delay 이후 작업” 프린트 후 `Unit.INSTANCE` 리턴하며 함수 종료 알림
- 결론
  - Continuation 객체의 역할
    - suspend 함수의 상태를 저장하고, 중단된 이후 이어서 실행할 수 있도록 하는 콜백 인터페이스
    - 상태값(label) 필드를 이용해 suspend 지점을 식별하며, 재개 시점에 맞춰 코드 흐름을 제어
  - CPS 변환
    - suspend 함수가 Continuation 파라미터를 추가해 CPS 형태로 변환됨으로써, 함수는 중간에 suspend된 후 나중에 resume(재개)될 수 있음
    - 위 코드에서는 delay() 호출로 인해 suspend되었다가, 100밀리초 후에 resume되어 최종 작업("delay 이후 작업")을 수행하고 결과를 반환
  - State machine
    - switch 문을 통해 label 값에 따른 상태 머신을 구현
    - delay 이전 / 이후를 label 값을 변화 시킴으로써 실행 순서 조절