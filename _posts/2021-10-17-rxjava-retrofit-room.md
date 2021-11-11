---
title: RxJava 강의 13 - RxJava with Retrofit / Room, 기존의 Callback을 RxStream으로 
date: 2021-10-17
categories:
 - Android
tags:
 - Reactive Programming
 - RxJava
---

Retrofit / Room을 RxJava와 함께 사용한 예시 코드를 살펴보고, RxJava를 지원하지 않고 Callback 형태만 지원하는 라이브러리를 Create operator를 활용하여 RxStream으로 변환 해 봅니다. 사용된 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/tree/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/exercise)에 모두 있으니 Github를 통해 전체 코드를 보는 것이 흐름 파악에 더 좋습니다. 

<!-- more -->

# 📚 TL; DR

## 📖 Retrofit

- Retrofit 함수의 결과를 RxStream의 형태로 변환해주는 라이브러리를 통해 Callback의 형태를 RxStream의 형태로 변환 할 수 있음

## 📖 Room

- Room의 query 함수의 결과를 RxStream의 형태로 변환해주는 라이브러리를 통해 query 함수의 결과 형태를 RxStream의 형태로 변환 할 수 있음

# 📚 Retrofit / Room

## 📖 개요

 Network 통신을 할 때 주로 사용되는 Library인 Retrofit과 Room을 RxJava와 함께 사용하여 활용 해 봅니다. 

 이번 포스팅에서는 Retrofit / Room library의 기초에 초점을 두어 설명하기 보다, RxJava와 Retrofit / Room을 함께 사용하는 형태를 주로 다룹니다. 

 Retrofit에 대한 상세 내용은 [Docs](https://square.github.io/retrofit/)를, Room에 대한 상세 내용은 [Docs](https://developer.android.com/training/data-storage/room?hl=ko)를 참조 해 주세요.

## 📖 Retrofit 설치

- Retrofit을 설치하고자 하는 모듈의 gradle 파일에 Retrofit Dependency를 추가 해 줍니다.
    
    ```kotlin
    // retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.7.2'
    ```
    
- Retrofit을 활용한 네트워크 통신 결과 형태를 RxJava2의 Stream 형태로 변형해주는 adapter dependency를 추가 해 줍니다.
    
    ```kotlin
    // retrofit-rxJava
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.7.2'
    ```
    
- 필요하다면 Retrofit 통신에 사용되는 유틸 라이브러리의 dependency들도 설치 해 줍니다.
    
    ```kotlin
    // gson converter
    implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
    // OkHttp
    implementation 'com.squareup.okhttp3:okhttp:3.14.7'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.4.2'
    ```
    
- 위에서 추가한 라이브러리의 버전은 필요에 따라 변경하시면 됩니다.

## 📖 기본적인 Retrofit - RxJava Stream 변환 방식

 기본적인 사용 방법은 Retrofit과 동일하나, 기존에는 API interface를 만들 때 `Call<T>`객체를 결과 값으로 리턴 해 주지만, RxStream, 즉 Observable / Single / Completable 등의 형태로 결과 형태를 정해 줄 수 있습니다. 

 즉, 아래의 예시 코드에서 첫번째 형태 ( `loginRetrofitCall` )는 Retrofit의 Callback 인터페이스를 활용하여 통신의 결과를 수신 할 수 있고 두번째 형태 ( `loginRxStream` )는 Single stream을 구독하는 곳에서 통신의 결과를 수신 할 수 있습니다. 

```kotlin
interface LoginApi {
    @POST("api/token/")
    fun loginRetrofitCall(@Body loginInfo: LoginInfo): Call<LoginItem>

    @POST("api/token/")
    fun loginRxStream(@Body loginInfo: LoginInfo): Single<LoginItem>
}
```

 `enqueue` operator를 활용 했던 것 처럼, 위의 `loginRxStream` 을 구독함으로써 네트워크 통신이 진행되며 결과 값을 받아 볼 수 있습니다. 

 네트워크 통신의 성공 여부만 필요하다면 `Completable` 을 활용 할 수 있으며, 각 필요에 따라 `Maybe` 등을 활용 할 수 있습니다. 

## 📖 Room 설치

- Room을 설치하고자 하는 모듈의 gradle 파일에 Room Dependency를 추가 해 줍니다.
    
    ```kotlin
    room_version = "2.3.0"
    // Room
    implementation "androidx.room:room-runtime:$room_version"
    ```
    
- Room 함수의 결과값을 RxJava의 Stream 형태로 변형해주는 라이브러리의 dependency를 추가 해 줍니다. ( [Docs](https://developer.android.com/training/data-storage/room/accessing-data#query-rxjava) )
    
    ```kotlin
    // optional - Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:$room_version"
    // optional - RxJava support for Room
    implementation "androidx.room:room-rxjava2:$room_version"
    ```
    
- `AppDatabase_Impl does not exist` 에러가 발생 하는 경우 `kotlin-kapt` plugin과 아래의 dependency를 추가 해 줍니다. ( ref : [https://stackoverflow.com/a/49323956](https://stackoverflow.com/a/49323956) )
    
    ```kotlin
    // https://stackoverflow.com/a/49323956
    kapt "androidx.room:room-compiler:$room_version"
    ```
    
- 위에서 추가한 라이브러리의 버전은 필요에 따라 변경하시면 됩니다.

## 📖 기본적인 Room - RxJava Stream 변환 방식

 기본적인 사용 방법은 Room과 동일하나, 기존에는 Dao interface를 만들 때 받고싶은 List / 객체 / Unit 값으로 리턴 해 주지만, RxStream, 즉 Observable / Single / Completable 등의 형태로 결과 형태를 정해 줄 수 있습니다. 

 즉, 아래의 예시 코드에서 `NotRx`키워드가 포함된 함수 ( 위 3개 함수 )들은 기존의 Room 활용 방식과 동일하고, `NotRx` 키워드가 포함되지 않은 함수 ( 아래의 3개 함수 )와 같이 선언 해 둔다면 Room을 활용한 db query의 결과 값을 Stream의 형태로 변형하여 수신 할 수 있습니다. 

```kotlin
@Dao
interface LocalTokenDao {
    @Query(value = "SELECT * FROM token LIMIT 1")
    fun getTokenNotRx(): LocalTokenItem

    @Insert
    fun saveTokenNotRx(tokenItem: LocalTokenItem)

    @Query(value = "DELETE FROM token")
    fun deleteAllCachedTokenNotRx()

    @Query(value = "SELECT * FROM token LIMIT 1")
    fun getToken(): Single<LocalTokenItem>

    @Insert
    fun saveToken(tokenItem: LocalTokenItem): Completable

    @Query(value = "DELETE FROM token")
    fun deleteAllCachedToken(): Completable
}
```

 query의 성공 여부만 필요하다면 `Completable` 을 활용 할 수 있으며, 각 필요에 따라 `Maybe`/ `Single` 등을 활용 할 수 있습니다. 

 위의 상세 내용은 ( [Docs](https://developer.android.com/training/data-storage/room/accessing-data#query-rxjava) )를 참조 해 주세요.

## 📖 Callback을 Create operator를 활용하여 Stream으로 변환하기

 Retrofit / Room을 활용하여 데이터 작업을 진행하다 보면, Rx를 지원하지 않는 라이브러리들이 존재합니다. 

 이럴 때, 라이브러리에서 제공해주는 콜백을 create operator를 활용하여 stream으로 변환하여 사용 할 수 있습니다. 

 예를들어, Firebase의 FCM을 활용 할 때, deviceToken을 Firebase에서 가져오는 작업을 진행 할 때 사용되는 callback 함수는 아래와 같습니다. ( ref : [https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko#retrieve-the-current-registration-token](https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko#retrieve-the-current-registration-token) )

```kotlin
FirebaseMessaging.getInstance().token.addOnCompleteListener(OnCompleteListener { task ->
    if (!task.isSuccessful) {
        Log.w(TAG, "Fetching FCM registration token failed", task.exception)
        return@OnCompleteListener
    }

    // Get new FCM registration token
    val token = task.result

    // Log and toast
    val msg = getString(R.string.msg_token_fmt, token)
    Log.d(TAG, msg)
    Toast.makeText(baseContext, msg, Toast.LENGTH_SHORT).show()
})
```

 위의 콜백을 아래의 형태처럼 Single stream으로 변환 해 준다면, 데이터를 처리 할 때 위의  `token.addOnCompleteListener` 를 활용 하면서도 stream을 이어 나갈 수 있습니다. 

```kotlin
private val getFirebaseTokenSingle: Single<String> by lazy {
    Single.create { emitter ->
        FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
            if (!task.isSuccessful) {
                Log.d("FCM", "fail to get token = ${task.exception?.localizedMessage}")
                emitter.onError(task.exception!!)
            }
            Log.d("FCM", "success to get token = ${task.result}")
            emitter.onSuccess(task.result!!)
        }
    }
}
```

```kotlin
// LoginRepositoryImpl.kt

override fun loginRxStream(id: String, password: String): Completable {
    return loginDataSource
        .loginRxStream(id, password)
        .flatMapCompletable {
            localTokenDataSource
                .deleteAllCachedToken()
                .andThen(localTokenDataSource.saveToken(mappingRemoteDataToLocal(it)))
                .andThen(getFirebaseTokenSingle.ignoreElement())
        }
}
```

 아래 Project 설명 부분에서 다시 설명하겠지만, 위에서 사용된 코드들은 [Github](https://github.com/kangraemin/RxJavaStudy/tree/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/exercise)에 모두 있으니 Github를 통해 전체 코드를 보는 것이 흐름 파악에 더 좋습니다. 

## 📖 RxJava와 Retrofit / Room을 혼용할 시 장점

### Callback을 만들지 않아도 됩니다.

- retrofit에서 Call 객체를 가지고 이벤트 처리를 진행하는 경우, 네트워크 통신 ( A )이 성공 했을 때 다른 네트워크 통신 ( B )을 하고, 시작한 네트워크 통신 ( B )가 성공했을 때 또다른 네트워크 통신 ( C )를 진행한다고 할 때, `enqueue` 를 여러번 중첩하여 쓰게 됩니다.
- 위와 같이 콜백이 여러번 중첩되는 경우, 코드 읽는것이 힘들어 질 뿐더러 flow를 추적하는것이 쉽지 않습니다.
- RxJava에서는 여러개의 Stream을 이어 놓는다면 구독하는 곳에서, 즉 subscribe 부분에서 모든 이벤트를 한번에 처리 할 수 있으므로 콜백을 여러개 만들지 않아도 됩니다.

### Error 처리 하기 용이합니다.

- `retryWhen` operator를 활용하여 특정 작업이 실패 했을 때, 다른 작업을 진행하고 실패한 작업을 재시도 등의 로직을 짜는것이 정말 간단해집니다.
- 예를들어, accessToken이 필요한 api A가 있다고 가정합시다.
- A작업을 할 때 header 값에 accessToken 값을 실어 보냅니다.
- 이때 accessToken 값이 만료되면, refreshToken을 가지고 accessToken을 재발급 받은 뒤 A작업을 재시도하는 로직을 짜려고 합니다.
- retryWhen을 활용하여, error stream을 refreshToken을 가지고 accessToken을 발행하는 stream으로 변경한다면 위 로직 구현이 끝이납니다.

### 여러 네트워크 처리를 한번에 처리하기 좋습니다.

- `zip` / `merge` / `concat` 등의 operator를 활용하여 여러개의 네트워크 통신 작업의 결과값을 정말 간단하게 처리 할 수 있습니다.
- 예를들어 여러개의 네트워크 통신 ( A, B, C, D )을 동시에 시작하고, A / B / C / D의 순서대로 결과값을 받고자 할 때 `concatEager` operator, 또는 `concat`과 `replaySubject`를 활용하여 매우 간단히 구현 할 수 있습니다.

### 즉, RxJava에 이미 만들어진 Operator들을 활용 할 수 있는것이 최고의 장점입니다.

- RxJava에 이미 만들어진 operator들을 활용하여, 네트워크 통신 flow를 간략히 할 수 있습니다.
- 예를들어, `filter` operator를 활용하여 네트워크 상태가 연결 되어 있을 때 만 네트워크 통신을 진행하도록 할 수 있고, `switchMap` operator를 활용하여 중복된 네트워크 통신이 진행 되어야 할 때 이전의 네트워크 작업을 취소시켜 줄 수 있습니다.
- `subscribeOn` operator / `observeOn` 를 활용하여 multi threading을 구현하기도 쉽습니다.
- 위에 적힌 예시들 이외에도 활용 방안은 정말 무궁무진 합니다.

# 📚 예시 프로젝트 설명 ( Login 기능 )

## 📖 프로젝트 기능 설명

![ezgif-3-c6e605f4a800.gif](/assets/images/posts/2021-10-17-rxjava-retrofit-room/ezgif-3-c6e605f4a800.gif)

- 네트워크 통신을 통해 로그인을 진행합니다.
- id / password가 모두 한글자 이상 입력되면 login 버튼이 활성화 되고, id / password가 하나라도 비었을 시 login 버튼을 비활성화 합니다.
- 로그인의 성공 여부를 버튼들 아래의 텍스트뷰에 표기합니다.
- 로그인을 성공하면, refreshToken / accessToken을 서버에서 제공 해 줍니다.
- 로그인을 성공하면, 저장된 token 데이터를 삭제하고 새로 받은 token 데이터를 로컬 DB에 저장합니다.
- `login not rx` 버튼을 클릭하면, RxJava를 활용하지 않은 Retrofit / Room을 활용하여 token을 가져오고, 저장합니다.
- `login` 버튼을 클릭하면, RxJava를 활용한 Retrofit / Room을 활용하여 token을 가져오고, 저장합니다.
- 전체적인 코드는 [Github](https://github.com/kangraemin/RxJavaStudy/tree/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/exercise)에서 확인 하실 수 있습니다.
- 아래에 코드에 대한 설명을 적어 놓았으나, 전체적인 맥락을 파악하기 위해 전체적인 코드를 [Github](https://github.com/kangraemin/RxJavaStudy/tree/main/AndroidProject/app/src/main/java/com/example/rxjavalecture/exercise)에서 확인해보는것을 강력하게 권해드립니다.

## 📖 Project code style

- 최대한 RxJava의 활용에만 초첨을 두어 설명하기 위해 의존성 주입 library / data binding의 적극적인 활용 등은 진행하지 않았습니다.
- MVVM 아키텍처 패턴을 활용하였으며, data binding을 view binding 용으로 활용하였습니다.
- AAC 뷰모델, 의존성 주입 라이브러리는 사용하지 않았습니다.
- Repository Pattern을 활용하였습니다.
- Room / Retrofit / RxJava / RxBinidng을 활용하였습니다.

## 📖 Login Button 활성화 / 비활성화 코드

- RxBinding과 combineLatest을 활용하여 구현 할 수 있습니다.
    
    ```kotlin
    // in LoginExampleActivity
    
    compositeDisposable
        .add(
            Observable
                .combineLatest(
                    binding.etId
                        .textChanges(),
                    binding.etPassword
                        .textChanges(),
                    BiFunction { id: CharSequence, password: CharSequence -> return@BiFunction id.isNotEmpty() && password.isNotEmpty() }
                )
                .subscribe({
                    binding.btnLoginNotRx.isEnabled = it
                    binding.btnLogin.isEnabled = it
                }, { it.printStackTrace() })
        )
    ```
    
- `textChanges()` operator를 활용하여 id / password의 text 변화를 stream으로 변환합니다.
- 변환한 stream을 `combineLatest` 를 활용하여 하나로 합치고, `BiFunction` 을 통해 다른 값으로 변환 해 줍니다.

## 📖 Login UI

- RxBinding의 `clicks()`를 통해 클릭 이벤트를 받고, viewModel의 login 함수를 실행시켜 줍니다.
    
    ```kotlin
    // in LoginExampleActivity
    
    package com.example.rxjavalecture.exercise.login
    
    import android.os.Bundle
    import androidx.appcompat.app.AppCompatActivity
    import androidx.databinding.DataBindingUtil
    import com.example.rxjavalecture.R
    import com.example.rxjavalecture.databinding.ActivityLoginExampleBinding
    import com.example.rxjavalecture.exercise.data.local.DatabaseClient
    import com.example.rxjavalecture.exercise.data.local.token.LocalTokenDataSourceImpl
    import com.example.rxjavalecture.exercise.data.remote.RetrofitClient
    import com.example.rxjavalecture.exercise.data.remote.login.LoginDataSourceImpl
    import com.example.rxjavalecture.exercise.data.repository.login.LoginRepositoryImpl
    import com.jakewharton.rxbinding3.view.clicks
    import com.jakewharton.rxbinding3.widget.textChanges
    import io.reactivex.Observable
    import io.reactivex.disposables.CompositeDisposable
    import io.reactivex.functions.BiFunction
    
    class LoginExampleActivity : AppCompatActivity() {
    
        private lateinit var binding: ActivityLoginExampleBinding
    
        private val compositeDisposable = CompositeDisposable()
    
        private val viewModel: LoginViewModel by lazy {
            LoginViewModel(
                loginRepository = LoginRepositoryImpl(
                    localTokenDataSource = LocalTokenDataSourceImpl(
                        localTokenDao = DatabaseClient.tokenDao()
                    ),
                    loginDataSource = LoginDataSourceImpl(
                        loginApi = RetrofitClient.loginAPI
                    )
                )
            )
        }
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            binding = DataBindingUtil.setContentView(this, R.layout.activity_login_example)
    
            compositeDisposable
                .add(
                    Observable
                        .combineLatest(
                            binding.etId
                                .textChanges(),
                            binding.etPassword
                                .textChanges(),
                            BiFunction { id: CharSequence, password: CharSequence -> return@BiFunction id.isNotEmpty() && password.isNotEmpty() }
                        )
                        .subscribe({
                            binding.btnLoginNotRx.isEnabled = it
                            binding.btnLogin.isEnabled = it
                        }, { it.printStackTrace() })
                )
    
    				// Button click시 login 실행
            compositeDisposable
                .add(
                    binding
                        .btnLogin
                        .clicks()
                        .subscribe({
                            viewModel.loginRxStream(
                                binding.etId.text.toString(),
                                binding.etPassword.text.toString()
                            )
                        }, { it.printStackTrace() })
                )
    
    				// Button click시 login 실행
            compositeDisposable
                .add(
                    binding
                        .btnLoginNotRx
                        .clicks()
                        .subscribe({
                            viewModel.loginRetrofitCall(
                                binding.etId.text.toString(),
                                binding.etPassword.text.toString()
                            )
                        }, { it.printStackTrace() })
                )
    
    				// ViewModel의 loginSuccessSubject를 구독하는 작업
    				// loginSuccessSubject가 true로 변경되면, 로그인이 성공한 뷰 / false면 로그인이 실패한 뷰를 보여 줌 
            compositeDisposable
                .add(
                    viewModel
                        .loginSuccessSubject
                        .subscribe({ successToLogin ->
                            if (successToLogin) {
                                binding.tvLoginResult.text =
                                    String.format(getString(R.string.login_result_format, "로그인 성공"))
                            } else {
                                binding.tvLoginResult.text =
                                    String.format(getString(R.string.login_result_format, "로그인 실패"))
                            }
                        }, { it.printStackTrace() })
                )
    
            binding.tvLoginResult.text = String.format(getString(R.string.login_result_format, "로그인 시도 전"))
        }
    
    		// Activity가 destroy 되면, viewModel에 있는 stream을 dispose 해주기 위함
        override fun onDestroy() {
            super.onDestroy()
            viewModel.onViewCleared()
        }
    }
    ```
    
- LoginViewModel
    
    ```kotlin
    package com.example.rxjavalecture.exercise.login
    
    import com.example.rxjavalecture.exercise.data.remote.base.BaseCallback
    import com.example.rxjavalecture.exercise.data.remote.base.transformCompletableToSingleDefault
    import com.example.rxjavalecture.exercise.data.repository.login.LoginRepository
    import io.reactivex.disposables.CompositeDisposable
    import io.reactivex.subjects.PublishSubject
    
    class LoginViewModel(
        private val loginRepository: LoginRepository
    ) {
    
        val loginSuccessSubject = PublishSubject.create<Boolean>()
    
        private val loginSubject = PublishSubject.create<Pair<String, String>>()
        private val compositeDisposable = CompositeDisposable()
    
        init {
            compositeDisposable
                .add(
                    loginSubject
                        .flatMapSingle {
                            loginRepository
                                .loginRxStream(it.first, it.second)
                                .transformCompletableToSingleDefault()
                        }
                        .subscribe({
                            loginSuccessSubject.onNext(it.throwable == null)
                        }, { it.printStackTrace() })
                )
        }
    
        fun loginRetrofitCall(id: String, password: String) {
            loginRepository.loginRetrofitCall(id, password, object : BaseCallback<Unit> {
                override fun onSuccess(data: Unit) {
                    loginSuccessSubject.onNext(true)
                }
    
                override fun onFail(throwable: Throwable) {
                    loginSuccessSubject.onNext(false)
                }
            })
        }
    
        fun loginRxStream(id: String, password: String) {
            loginSubject.onNext(Pair(id, password))
        }
    
        fun onViewCleared() {
            compositeDisposable.dispose()
        }
    }
    ```
    
    - ViewModel의 loginRxStream 함수가 실행되면, Subject에 onNext를 통해 id / password를 감싼 Pair 객체를 흘려 줍니다.
    - LoiginRepository를 주입해서 사용받고 있으며, 이를 활용하여 login을 진행 합니다.
    - `transformCompletableToSingleDefault()` 메소드를 활용하여 login에서 에러가 발생 하였을 때, subject가 종료되지 않도록 합니다.

## 📖 Repository Component

- Repository
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.repository.login
    
    import com.example.rxjavalecture.exercise.data.remote.base.BaseCallback
    import io.reactivex.Completable
    
    interface LoginRepository {
        fun loginRetrofitCall(id: String, password: String, callback: BaseCallback<Unit>)
        fun loginRxStream(id: String, password: String): Completable
    }
    ```
    
    - login 작업을 서술해놓은 interface 입니다.
    - retrofit을 활용한 함수와, RxStream을 활용한 함수를 서술 해 놓았습니다.
    - 로그인의 결과값인 Token 값을 View에서 사용하고 있지 않기 때문에, `Single`이 아닌 `Completable`로 진행 하였습니다.
- RepositoryImpl
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.repository.login
    
    import android.util.Log
    import com.example.rxjavalecture.exercise.data.local.token.LocalTokenDataSource
    import com.example.rxjavalecture.exercise.data.local.token.LocalTokenMapper.mappingRemoteDataToLocal
    import com.example.rxjavalecture.exercise.data.remote.base.BaseCallback
    import com.example.rxjavalecture.exercise.data.remote.login.LoginDataSource
    import com.example.rxjavalecture.exercise.data.remote.login.LoginItem
    import com.google.firebase.messaging.FirebaseMessaging
    import io.reactivex.Completable
    import io.reactivex.Single
    import retrofit2.Call
    import retrofit2.Callback
    import retrofit2.Response
    
    class LoginRepositoryImpl(
        private val localTokenDataSource: LocalTokenDataSource,
        private val loginDataSource: LoginDataSource
    ) : LoginRepository {
    
        private val getFirebaseTokenSingle: Single<String> by lazy {
            Single.create { emitter ->
                FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
                    if (!task.isSuccessful) {
                        Log.d("FCM", "fail to get token = ${task.exception?.localizedMessage}")
                        emitter.onError(task.exception!!)
                    }
                    Log.d("FCM", "success to get token = ${task.result}")
                    emitter.onSuccess(task.result!!)
                }
            }
        }
    
        override fun loginRetrofitCall(id: String, password: String, callback: BaseCallback<Unit>) {
            loginDataSource
                .loginRetrofitCall(id, password, object : Callback<LoginItem> {
                    override fun onResponse(call: Call<LoginItem>, response: Response<LoginItem>) {
                        if (response.isSuccessful) {
                            try {
                                Thread {
                                    localTokenDataSource
                                        .deleteAllCachedTokenNotRx()
                                    localTokenDataSource
                                        .saveTokenNotRx(mappingRemoteDataToLocal(response.body()!!))
                                }.start()
                            } catch (e: Throwable) {
                                e.printStackTrace()
                            }
                            callback.onSuccess(Unit)
                        } else {
                            callback.onFail(Exception())
                        }
                    }
    
                    override fun onFailure(call: Call<LoginItem>, t: Throwable) {
                        callback.onFail(t)
                    }
                })
        }
    
        override fun loginRxStream(id: String, password: String): Completable {
            return loginDataSource
                .loginRxStream(id, password)
                .flatMapCompletable {
                    localTokenDataSource
                        .deleteAllCachedToken()
                        .andThen(localTokenDataSource.saveToken(mappingRemoteDataToLocal(it)))
                        .andThen(getFirebaseTokenSingle.ignoreElement())
                }
        }
    }
    ```
    
    - `loginDataSource` 를 활용하여 network 통신을 통해 login 작업을 진행하고, login 이후 서버에서 가져온 token 값을 `localTokenDataSource` 를 통해 처리합니다.
    - RxStream을 활용한 버전에서는 `flatMapCompletable` 을 활용하여 stream을 변환하고, login 작업이 끝났을 때 `localTokenDataSource` 를 활용하여 저장되어 있던 token 값을 지우고, 지우는걸 성공 했을 때 받아온 token 값을 저장합니다.
    - RxStream을 활용하지 않은 버전에서는, callback을 통해 로그인 api 통신 성공 및 response가 isSuccessful 일 때 로그인에 성공했다고 가정하고 로컬 토큰을 조작하는 작업을 진행합니다.
    - firebase를 활용하여 deviceToken 값을 가져 오는 작업을 진행 할 때, `getFirebaseTokenSingle` 을 활용하여 `andThen` stream으로 연결 해 준것을 확인 할 수 있습니다.

## 📖 Remote Component

- BaseAPICallResult
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.base
    
    data class BaseAPICallResult<T>(
        val result: T? = null,
        val throwable: Throwable? = null
    )
    ```
    
- BaseAPICallTransformer
    
    ```kotlin
    // BaseAPICallTransformer 
    
    package com.example.rxjavalecture.exercise.data.remote.base
    
    import io.reactivex.Completable
    import io.reactivex.Single
    import io.reactivex.SingleTransformer
    
    fun Completable.transformCompletableToSingleDefault(): Single<BaseAPICallResult<Unit>> =
        toSingleDefault(Unit)
            .compose(wrappingSingleAPIResultData())
    
    fun <T> Single<T>.wrappingAPICallResult(): Single<BaseAPICallResult<T>>
        = compose(wrappingSingleAPIResultData())
    
    fun <T> wrappingSingleAPIResultData() = SingleTransformer<T, BaseAPICallResult<T>> { single ->
        single
            .map { data -> BaseAPICallResult(result = data) }
            .onErrorReturn { BaseAPICallResult(throwable = it) }
    }
    ```
    
    - ViewModel 에서 subject를 활용하여 stream을 이어가는 경우에, 네트워크 통신 도중 에러가 발생 했을 때에도 subject가 죽지 않도록 합니다.
    - BaseAPICallResult 형태로 Network Call의 결과 값을 리턴 해 줍니다.
- Login API
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.login
    
    import io.reactivex.Single
    import retrofit2.Call
    import retrofit2.http.Body
    import retrofit2.http.POST
    
    interface LoginApi {
        @POST("api/token/")
        fun loginRetrofitCall(@Body loginInfo: LoginInfo): Call<LoginItem>
    
        @POST("api/token/")
        fun loginRxStream(@Body loginInfo: LoginInfo): Single<LoginItem>
    }
    ```
    
- LoginDataSource
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.login
    
    import io.reactivex.Single
    import retrofit2.Callback
    
    interface LoginDataSource {
        fun loginRetrofitCall(id: String, password: String, callback: Callback<LoginItem>)
        fun loginRxStream(id: String, password: String): Single<LoginItem>
    }
    ```
    
- LoginDataSourceImpl
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.login
    
    import io.reactivex.Single
    import io.reactivex.schedulers.Schedulers
    import retrofit2.Callback
    
    class LoginDataSourceImpl(
        private val loginApi: LoginApi
    ): LoginDataSource {
        override fun loginRetrofitCall(id: String, password: String, callback: Callback<LoginItem>) {
            loginApi
                .loginRetrofitCall(LoginInfo(id, password))
                .enqueue(callback)
        }
    
        override fun loginRxStream(id: String, password: String): Single<LoginItem> {
            return loginApi
                .loginRxStream(LoginInfo(id, password))
                .subscribeOn(Schedulers.io())
        }
    }
    ```
    
    - Retrofit의 loginApi를 통해 서버와 통신합니다.
    - RxStream을 활용한 함수에서는 여기서 구독을 진행하지 않지만, RxStream을 활용하지 않은 함수에서는 여기서 `enqueue` 작업을 통해 콜백을 등록하고 통신을 시작합니다.
- LoginInfo
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.login
    
    data class LoginInfo(
        val username: String,
        val password: String
    )
    ```
    
    - 로그인 작업을 위해 필요한 인자의 형태가 서술된 data class 입니다.
- LoginItem
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.remote.login
    
    data class LoginItem(
        val access: String,
        val refresh: String
    )
    ```
    
    - 서버를 통해 login 작업을 진행 한 결과의 형태가 서술된 data class 입니다.

## 📖 Local Component

- LocalTokenDao
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.local.token
    
    import androidx.room.Dao
    import androidx.room.Insert
    import androidx.room.Query
    import io.reactivex.Completable
    import io.reactivex.Single
    
    @Dao
    interface LocalTokenDao {
        @Query(value = "SELECT * FROM token LIMIT 1")
        fun getToken(): Single<LocalTokenItem>
    
        @Insert
        fun saveToken(tokenItem: LocalTokenItem): Completable
    
        @Query(value = "DELETE FROM token")
        fun deleteAllCachedToken(): Completable
    
        @Query(value = "SELECT * FROM token LIMIT 1")
        fun getTokenNotRx(): LocalTokenItem
    
        @Insert
        fun saveTokenNotRx(tokenItem: LocalTokenItem)
    
        @Query(value = "DELETE FROM token")
        fun deleteAllCachedTokenNotRx()
    }
    ```
    
    - 함수에 `NotRx` 가 포함되지 않은 함수는 RxStream을 활용한 쿼리 함수이며, `NotRx` 가 포함된 함수는 RxStream을 활용하지 않은 쿼리 함수입니다.
- LocalDataSource
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.local.token
    
    import io.reactivex.Completable
    import io.reactivex.Single
    
    interface LocalTokenDataSource {
        fun getToken(): Single<LocalTokenItem>
        fun saveToken(tokenItem: LocalTokenItem): Completable
        fun deleteAllCachedToken(): Completable
        fun getTokenNotRx(): LocalTokenItem
        fun saveTokenNotRx(tokenItem: LocalTokenItem)
        fun deleteAllCachedTokenNotRx()
    }
    ```
    
- LocalDataSourceImpl
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.local.token
    
    import io.reactivex.Completable
    import io.reactivex.Single
    import io.reactivex.schedulers.Schedulers
    
    class LocalTokenDataSourceImpl(
        private val localTokenDao: LocalTokenDao
    ) : LocalTokenDataSource {
        override fun getToken(): Single<LocalTokenItem> {
            return localTokenDao
                .getToken()
                .subscribeOn(Schedulers.io())
        }
    
        override fun saveToken(tokenItem: LocalTokenItem): Completable {
            return localTokenDao
                .saveToken(tokenItem)
                .subscribeOn(Schedulers.io())
        }
    
        override fun deleteAllCachedToken(): Completable {
            return localTokenDao
                .deleteAllCachedToken()
                .subscribeOn(Schedulers.io())
        }
    
        override fun getTokenNotRx(): LocalTokenItem {
            return localTokenDao
                .getTokenNotRx()
        }
    
        override fun saveTokenNotRx(tokenItem: LocalTokenItem) {
            return localTokenDao
                .saveTokenNotRx(tokenItem)
        }
    
        override fun deleteAllCachedTokenNotRx() {
            return localTokenDao
                .deleteAllCachedTokenNotRx()
        }
    }
    ```
    
    - `subscribeOn` 을 통해, 백그라운드 쓰레드에서 작업하도록 설정 한 것을 확인 할 수 있습니다.
- LocalTokenItem
    
    ```kotlin
    package com.example.rxjavalecture.exercise.data.local.token
    
    import androidx.room.ColumnInfo
    import androidx.room.Entity
    import androidx.room.PrimaryKey
    
    @Entity(tableName = "token")
    data class LocalTokenItem(
        @PrimaryKey(autoGenerate = true) val idx: Int = 0,
        @ColumnInfo(name = "accessToken") val accessToken: String,
        @ColumnInfo(name = "refreshToken") val refreshToken: String
    )
    ```
    
    - Room의 token 테이블 형태를 서술한 data class 입니다.