---
title: Android - ViewModel 에 대해서   
date: 2025-04-14
categories:
 - Android
tags:
 - Android 
 - ViewModel
---

Android 에서 자주 쓰이는 ViewModel에 대해서 알아봅니다.

<!-- more -->

### 1. SafeIterableMap

<aside>
💡 순회 ( Iteration ) 중에, 항목이 추가되거나 삭제되더라도 예외가 발생 하지 않는 Map 자료 구조
</aside>

- 일반적인 map 구현체에선, ConcurrentModificationException 이 발생하지만, SafeIterableMap 에선 발생 하지 않음
- Android 아키텍처 컴포넌트에서 상태 관리나 이벤트 전달과 같은 작업에 사용
- LinkedList 기반이라, HashMap / TreeMap 에 비해 검색 속도가 느릴 수 있음

### 2. SavedStateProvider

<aside>
💡 SavedStateRegistry 에서 상태를 저장하고 복원하기 위해 사용하는 인터페이스
</aside>

- 컴포넌트가 종료되기 전에 상태를 저장하고, 이후 복원 할 수 있도록 설계 됨
- saveState() 메서드를 통해 컴포넌트의 상태를 Bundle로 반환
- SavedStateRegistry의 consumeRestoredStateForKey() 메서드를 통해 복원

```kotlin
fun interface SavedStateProvider {
    /**
     * Called to retrieve a state from a component before being killed
     * so later the state can be received from [consumeRestoredStateForKey]
     *
     * Returns `S` with your saved state.
     */
    fun saveState(): Bundle
}
```

### 3. SavedStateRegistry

<aside>
💡 Activity / Fragment 의 상태를 저장 / 복원 하기 위한 클래스
</aside>

- 데이터 저장
  - `SafeIterableMap<String, SavedStateProvider>()` 을 통해 저장
  - registerSavedStateProvider 를 통해 SavedStateProvider 를 등록
  - performSave 가 호출되면, SavedStateProvider의 saveState 메서드를 호출하여 데이터를 저장
- 데이터 복원
  - performRestore 메서드를 통해 이전에 저장된 상태를 복원합니다.
  - 복원된 데이터는 consumeRestoredStateForKey 메서드를 통해 개별적으로 소비할 수 있습니다.
  - 복원된 데이터는 한 번 소비되면 내부적으로 제거됩니다.
- 재생성
  - runOnNextRecreation 메서드를 사용하여 특정 클래스가 Activity나 Fragment 재생성 시 자동으로 초기화되도록 설정
  - 이 클래스는 AutoRecreated 인터페이스를 구현해야 하며, 기본 생성자를 가져야 합니다.
- performAttach
  - SavedStateRegistry를 Lifecycle에 연결
- performRestore
  - 저장된 상태를 복원
  - performAttach가 호출된 이후에만 복원이 가능하며, 중복 복원을 방지

### 4. SavedStateRegistryController

<aside>
💡 SavedStateRegistryOwner가 저장 상태를 관리하고 복원할 수 있도록 돕는 상태 관리 컨트롤러
</aside>

- SavedStateRegistry 객체를 생성하여 저장 상태 관리 기능을 제공
- performAttach()
  - SavedStateRegistry를 초기화하고 생명주기 관찰자를 추가
  - 반드시 생명주기 상태가 INITIALIZED일 때 호출되어야 함
- performRestore(savedState: Bundle?)
  - 저장된 상태를 복원
  - 생명주기 상태가 STARTED 이상일 경우 호출할 수 없음
    - performRestore는 Lifecycle이 초기화 단계(Lifecycle.State.INITIALIZED)에서 호출되어야 하며, 이미 시작된 상태(Lifecycle.State.STARTED 이상)에서는 호출되면 안 됨
    - 잘못된 시점에서 상태를 복원하면 SavedStateRegistry의 데이터가 손상되거나, 예상치 못한 동작이 발생 가능
- performSave(outBundle: Bundle)
  - 현재 상태를 Bundle에 저장
  - 등록된 모든 상태 제공자들의 데이터를 수집하고 병합
- create(owner: SavedStateRegistryOwner)
  - SavedStateRegistryController의 인스턴스를 생성
  - SavedStateRegistryOwner의 생성 시점에 호출

### 5. ViewModelStore

<aside>
💡 ViewModel 인스턴스들을 저장, 검색 및 정리하며 수명주기에 따른 관리를 담당하는 클래스
</aside>

- ViewModelStore 클래스는 내부에 가변 맵을 유지하며, 키와 연결된 ViewModel 인스턴스들을 저장, 검색, 그리고 정리하는 기능을 제공
- put(key, viewModel)
  - **원리:** 전달받은 키에 해당하는 `ViewModel`을 저장하며, 이미 해당 키에 저장된 이전 인스턴스가 있으면 이를 clear 호출해 정리
- get(key)
  - **원리:** 저장된 맵에서 지정한 키의 `ViewModel`을 반환
- keys()
  - **원리:** 현재 저장된 모든 키의 복사본을 반환하여 외부에서 안전하게 조회 할 수 있도록 함
- clear()
  - **원리:** 맵에 저장된 모든 `ViewModel` 인스턴스에 대해 clear()를 호출한 후, 내부 저장소를 비움

### 6.  ViewModelProvider

<aside>
💡 ViewModel의 생성, 저장, 및 재사용을 관리하여 Android 애플리케이션의 상태를 효율적으로 유지하는 역할
</aside>

- expect 키워드를 사용하여 플랫폼별 구현을 기대하는 멀티플랫폼 구조를 따름
  - expect 키워드는 Kotlin Multiplatform에서 사용되며, 플랫폼별 구현이 필요함을 나타냄
- Get method

    ```kotlin
    @MainThread
    public operator fun <T : ViewModel> get(modelClass: KClass<T>): T
    
    @MainThread
    public operator fun <T : ViewModel> get(key: String, modelClass: KClass<T>): T
    ```

  - ViewModel을 클래스 타입(modelClass) 또는 키(key)를 기반으로 가져옴
  - ViewModel이 존재하지 않으면 새로 생성
- Factory interface

    ```kotlin
    public interface Factory {
        public open fun <T : ViewModel> create(
            modelClass: KClass<T>,
            extras: CreationExtras,
        ): T
    }
    ```

  - ViewModel 생성 로직을 캡슐화
  - Factory를 통해 ViewModel의 인스턴스를 생성
- OnRequeryFactory

    ```kotlin
    public open class OnRequeryFactory {
        public open fun onRequery(viewModel: ViewModel)
    }
    ```

  - ViewModel이 재요청될 때 호출되는 콜백을 정의
- companion object

    ```kotlin
    public companion object {
        public fun create(
            owner: ViewModelStoreOwner,
            factory: Factory = ViewModelProviders.getDefaultFactory(owner),
            extras: CreationExtras = ViewModelProviders.getDefaultCreationExtras(owner),
        ): ViewModelProvider
    
        public fun create(
            store: ViewModelStore,
            factory: Factory = DefaultViewModelProviderFactory,
            extras: CreationExtras = CreationExtras.Empty,
        ): ViewModelProvider
    
        public val VIEW_MODEL_KEY: CreationExtras.Key<String>
    }
    ```

  - ViewModelProvider를 생성하는 정적 메소드와 상수를 제공
  - VIEW_MODEL_KEY는 ViewModel의 키를 관리하는 데 사용

### 7. Lazy 인터페이스

<aside>
💡 객체를 지연 초기화 하는데 활용
</aside>

- 지연 초기화(lazy initialization)를 지원하기 위해 설계된 인터페이스
  - 이는 객체의 초기화를 실제로 필요할 때까지 미루는 방식으로, 성능 최적화와 메모리 절약에 유용
- 코드

    ```kotlin
    public interface Lazy<out T> {
        /**
         * Gets the lazily initialized value of the current `Lazy` instance.
         * Once the value was initialized it must not change during the rest of lifetime of this `Lazy` instance.
         *
         * @sample samples.lazy.LazySamples.lazyExplicitSample
         */
        public val value: T
    
        /**
         * Returns `true` if a value for this `Lazy` instance has been already initialized, and `false` otherwise.
         * Once this function has returned `true` it stays `true` for the rest of lifetime of this `Lazy` instance.
         */
        public fun isInitialized(): Boolean
    }
    ```

- 장점
  - Lazy는 내부적으로 초기화 상태를 추적하며, 필요할 때만 초기화 로직을 실행
    - 이를 통해 불필요한 연산을 방지하고, 프로그램의 성능을 향상시킬 수 있음
  - 객체 생성이나 연산이 비용이 크거나, 프로그램 실행 중 항상 필요한 것이 아닐 때 사용
    - 예: 데이터베이스 연결, 파일 읽기, 복잡한 계산 등
  - 초기화가 필요하지 않은 경우, 초기화 로직을 건너뛰어 메모리와 CPU 자원을 절약
  - LazyThreadSafetyMode를 통해 초기화의 스레드 안전성을 제어할 수 있음
    - `SYNCHRONIZED`: 기본값으로, 스레드 안전한 초기화를 보장
    - `PUBLICATION`: 여러 스레드에서 초기화가 중복될 수 있지만, 최종적으로 하나의 값만 사용
    - `NONE`: 스레드 안전성을 보장하지 않으며, 단일 스레드 환경에서 사용
- by lazy
  - by lazy는 내부적으로 Lazy 인터페이스를 구현한 객체를 생성
  - by lazy를 사용하면 초기화 로직(람다 함수)이 Lazy 객체에 저장
  - lazy 함수는 Lazy 객체를 반환하며, value 프로퍼티에 접근할 때 초기화 로직이 실행
  - lazy 함수는 기본적으로 `LazyThreadSafetyMode.SYNCHRONIZED` 모드로 동작하여, 멀티스레드 환경에서도 안전하게 초기화가 이루어지도록 보장

### 8. ViewModelLazy

<aside>
💡 ViewModel을 지연 초기화(Lazy Initialization) 방식으로 제공하기 위해 설계
</aside>

- ViewModel 객체를 필요 할 때 까지 생성하지 않음
- ViewModel 은 ViewModelStore 를 통해서 관리 됨
- 내부 코드

    ```kotlin
    public class ViewModelLazy<VM : ViewModel> @JvmOverloads constructor(
    		// 생성할 ViewModel의 클래스 타입.
        private val viewModelClass: KClass<VM>, 
        // ViewModelStore를 제공하는 람다.
        private val storeProducer: () -> ViewModelStore, 
        // ViewModelProvider.Factory를 제공하는 람다.
        private val factoryProducer: () -> ViewModelProvider.Factory,
        // CreationExtras를 제공하는 람다 (기본값은 CreationExtras.Empty).
        private val extrasProducer: () -> CreationExtras = { CreationExtras.Empty }
    ) : Lazy<VM> {
        private var cached: VM? = null
    
        override val value: VM
            get() {
                val viewModel = cached
                return if (viewModel == null) {
                    val store = storeProducer()
                    val factory = factoryProducer()
                    val extras = extrasProducer()
                    ViewModelProvider.create(store, factory, extras)
                        .get(viewModelClass)
                        .also { cached = it }
                } else {
                    viewModel
                }
            }
    
        override fun isInitialized(): Boolean = cached != null
    }
    ```

  - cached
    - 초기화된 ViewModel 객체를 저장하는 변수로, 이후 호출 시 동일한 인스턴스를 반환합니다.
  - value
    - Lazy 인터페이스의 필수 구현으로, ViewModel 객체를 반환
    - cached가 null이면 storeProducer, factoryProducer, extrasProducer를 호출하여 ViewModel을 생성하고 캐싱
  - isInitialized
    - Lazy 인터페이스의 또 다른 구현으로, cached가 초기화되었는지 여부를 반환

### 9. by viewModels() 확장함수

<aside>
💡 ViewModelProvider와 ViewModelStore를 활용하여 ViewModel 객체를 간단하게 만들 수 있는 확장 함수
</aside>

- 내부 코드

    ```kotlin
    @MainThread
    public inline fun <reified VM : ViewModel> ComponentActivity.viewModels(
        noinline extrasProducer: (() -> CreationExtras)? = null,
        noinline factoryProducer: (() -> Factory)? = null
    ): Lazy<VM> {
        val factoryPromise = factoryProducer ?: {
            defaultViewModelProviderFactory
        }
    
        return ViewModelLazy(
            VM::class,
            { viewModelStore },
            factoryPromise,
            { extrasProducer?.invoke() ?: this.defaultViewModelCreationExtras }
        )
    }
    ```

  - reified 키워드 → 제네릭 타입 정보를 런타임에 사용할 수 있도록 함
    - 이를 통해 VM::class를 사용하여 ViewModel의 클래스 타입을 런타임에 참조할 수 있음
  - producer

      ```kotlin
      noinline extrasProducer: (() -> CreationExtras)? = null,
      noinline factoryProducer: (() -> Factory)? = null
      ```

    - 추가적인 설정을 제공, 없으면 null
  - Lazy 반환

      ```kotlin
      return ViewModelLazy(
              // 생성할 ViewModel의 클래스 타입.
          VM::class,
          // ViewModel의 저장소를 제공. ComponentActivity의 viewModelStore를 사용
          { viewModelStore },
          // ViewModel 생성에 사용할 팩토리. 기본적으로 defaultViewModelProviderFactory 사용
          factoryPromise,
          //  CreationExtras를 제공. extrasProducer가 없으면 기본값을 사용
          { extrasProducer?.invoke() ?: this.defaultViewModelCreationExtras }
      )
      ```


### 10. ComponentActivity

<aside>
💡 ViewModel 상태 복구와 관련된 코드는 ViewModelStore와 SavedStateRegistry를 활용하여 상태를 저장하고 복구하는 방식으로 구현
</aside>

- ComponentActivity 특이사항
  1. SavedStateRegistry 초기화 / 연결 ( in init 블록 )

      ```kotlin
      savedStateRegistryController.performAttach()
      enableSavedStateHandles()
      ```

    - performAttach()는 SavedStateRegistry를 ComponentActivity에 연결
    - enableSavedStateHandles()는 ViewModel에서 SavedStateHandle을 사용할 수 있도록 설정
  2. SavedStateRegistry 복구 ( in onCreate )

      ```kotlin
      savedStateRegistryController.performRestore(savedInstanceState)
      ```

    - SavedRegistry의 상태 복구
    - performRestore()로 Bundle에서 저장된 상태를 읽어와 SavedStateRegistry에 복구
  3. ViewModelStore 초기화

      ```kotlin
      override val viewModelStore: ViewModelStore
          /**
           * Returns the [ViewModelStore] associated with this activity
           *
           * Overriding this method is no longer supported and this method will be made
           * `final` in a future version of ComponentActivity.
           *
           * @return a [ViewModelStore]
           * @throws IllegalStateException if called before the Activity is attached to the
           * Application instance i.e., before onCreate()
           */
          get() {
              checkNotNull(application) {
                  ("Your activity is not yet attached to the " +
                      "Application instance. You can't request ViewModel before onCreate call.")
              }
              ensureViewModelStore()
              return _viewModelStore!!
          }
      
      private fun ensureViewModelStore() {
          if (_viewModelStore == null) {
              val nc = lastNonConfigurationInstance as NonConfigurationInstances?
              if (nc != null) {
                  // Restore the ViewModelStore from NonConfigurationInstances
                  _viewModelStore = nc.viewModelStore
              }
              if (_viewModelStore == null) {
                  _viewModelStore = ViewModelStore()
              }
          }
      }
      ```

    - ensureViewModelStore() 함수를 통해 ViewModelStore 초기화
      - 캐싱된 lastNonConfigurationInstance 에서 ViewModelStore 를 가져오거나, 없으면 새로 생성
    - ViewModelStore는 ViewModel 인스턴스를 저장하고 관리
    - ComponentActivity는 ViewModelStore를 초기화하거나 이전 상태를 복구
    - lastNonConfigurationInstance에서 ViewModelStore를 복구
  4. onDestroy 에서 ViewModelStore 정리

      ```kotlin
      if (!isChangingConfigurations) {
          viewModelStore.clear()
      }
      ```

    - isChangingConfigurations가 false일 때만 ViewModelStore를 정리
    - 이는 구성 변경 시 ViewModel을 유지하기 위함