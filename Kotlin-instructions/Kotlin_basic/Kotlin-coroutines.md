# Kotlin coroutines

> + 동시성 프로그래밍.
> + 백그라운드 스레드 관리에 권장되는 방법이며, 콜백들을 줄여줌.
> + 코루틴은 코틀린 기능으로, 데이터베이스나 네트워크 엑세스와 같은 장기 작업에 대한 비동기 콜백들을 순차적인 코드로 변환한다.

+ `launch` 와 `runBlocking`으로 코드 실행 컨트롤.
+ suspend 함수로 비동기 코드를 순차적으로.
+ `suspendCoroutine`을 사용해 기존 API들을 코루틴으로 변환 가능.
+ AAC를 사용한 코루틴.
+ 테스팅 코루틴

```kotlin
dependencies {
  ...
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:x.x.x"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:x.x.x"
}
```

+ `note:`@UiThread 어노테이션을 함수에 붙이면 리턴이 메인스레드로부터 직접 진행됨.
  -> 이는 블락킹 메서드를 호출할 수 없음을 의미.
+ `suspend(유예)` 키워드는 코루틴을 가능하게 함. suspend가 마킹된 함수는 해당 함수의 결과가 준비될 때까지 다음으로의 진행을 유예하고, 결과와 함께 재개됨.
+  유예된 결과를 기다리는 동안, 호출 스레드를 블락하지 않음.
  -> 호출 스레드는 다른 함수들과 다른 코루틴을 진행할 수 있음.
+ suspend 키워드는 실행되는 스레드를 지정하지 않고, suspend 기능은 백그라운드 스레드나 메인 스레드에서 실행될 수 있음.
+ 함수에의 @WorkerThread 어노테이션은 에러가 발생하는 경우를 제외하고는 메인스레드에서 호출될 수 없음을 의미.
+ `suspend`키워드는 다른 언어에서의 async와 비슷하지만, 코틀린에선 suspend 함수가 호출되면 await()가 무조건 사용됨.
+ 코틀린에서도 async 빌더로 시작된 코루틴에서 결과를 기다리는 기능을 하는 Defferd.await() 메서드가 있음.

## CoroutineScope

+ 코틀린에서 모든 코루틴은 코루틴스코프 안에서 실행됨.
+ 스코프는 포함되는 코루틴 Job들의 라이프타임을 컨트롤함.
+ 스코프에 포함된 Job이 취소되면 해당 스코프에서 시작된 모든 코루틴들이 취소됨.
  안드로이드에선 스코프를 실행하고 있는 모든 코루틴들을 취소하기위해 사용됨.
+  일반적으로 코루틴의 진입점을 UI에 두기 위해 코루틴의 시작을 `Dispatchers.Main`으로 하는 게 알맞음.
+ Dispatchers.Main에서 시작된 코루틴은 유예된 동안 메인 스레드를 블락하면 안됨. 
+ ViewModel 코루틴은 거의 항상 메인 스레드의 UI를 업데이트하므로 메인스레드에서 코루틴을 시작하면 추가적인 스레드 스위칭 낭비를 방지할 수 있음.
+ 메인스레드에서 시작된 코루틴은 Dispatchers를 언제든 스위칭할 수 있음.
  (예 : 다른 dispatcher에서 큰 JSON을 파싱하고 결과를 메인스레드로 넘겨줌)

### 코루틴의 메인-세이프티 제공

+ 언제든 스레드 스위칭이 가능하므로 UI관련 코루틴을 메인스레드에서 시작하는 것이 권장됨.
+ Room과 Retrofit은 코루틴을 사용하면 메인-세이프티를 라이브러리에서 자체적으로 제공함.
  -> 따로 스레드 관리가 필요없음.
+ 리스트를 정렬하거나 파일을 읽는 등의 블락킹 코드는 반드시 메인-세이프티 처리해줘야함.
  -> 코루틴를 서포하지않은 네트워킹/데이터베이스 라이브러리의 경우에도 마찬가지.

### ViewModelScope

```kotlin
dependencies {
  ...
  implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:x.x.x"
}
```

+ 라이브러리 : UI관련 코루틴을 시작하기 위한 설정이 된 CoroutineScope를 ViewModel에 추가한 것.
+ ViewModel 클래스의 viewModelScope 확장 함수가 포함됨.
  `Dispatchers.Main`에 바인드된 스코프.
  ViewModel의 라이프사이클 상태가 cleared 상태가 되면 자동으로 포함된 모든 코루틴 취소됨.
+ delay() 함수는 suspend 함수. 스레드 블락킹하지 않음.
  대신 dispatcher가 delay함수에 인자로 입력된 값의 시간 후에 다시 재개하도록 예약함.

### Testing coroutines

+ ⚠️kotlinx-coroutines-test 라이브러리는 실험용으로, 릴리즈 전 주요 변경사항이 있을 수 있음.

```kotlin
//./test/MainViewModelTest.kt
class MainViewModelTest {
   @get:Rule
   val coroutineScope =  MainCoroutineScopeRule()
   @get:Rule
   val instantTaskExecutorRule = InstantTaskExecutorRule()

   lateinit var subject: MainViewModel

   @Before
   fun setup() {
       subject = MainViewModel(
           TitleRepository(
                   MainNetworkFake("OK"),
                   TitleDaoFake("initial")
           ))
   }
}
```

+ Rule은 JUnit에서 테스트 실행 전후에 코드를 실행하는 방법. 두가지 Rule은 오프 디바이스 테스트에서 MainViewModel를 테스트할 수 있음. 
+ InstantTaskExecutorRule은 각 작업을 동기적으로 실행하도록 LiveData를 config하는 JUnit 규칙.
+ MainCoroutineScopeRule은 kotlinx-coroutines-test의 TestCoroutineDispatcher를 사용하도록 Dispatchers.Main을 Config하는 이 코드베이스의 custom rule.
+  setup()메서드에서 MainViewModel의 새로운 인스턴스는 테스팅을 위한 fake를 사용해 생성.
+ 실제 네트워크나 데이터베이스를 사용하지않고 스타터 코드에 제공되는 가짜 구현.

```kotlin
@Test
fun whenMainClicked_updatesTaps() {
   subject.onMainViewClicked()
   Truth.assertThat(subject.taps.getValueForTest()).isEqualTo("0 taps")
   coroutineScope.advanceTimeBy(1000)
   Truth.assertThat(subject.taps.getValueForTest()).isEqualTo("1 taps")
}
```

+ onMainViewClicked()가 호출되면서 만든 코루틴이 시작됨.
+ 테스트에선 가상-시간을 사용해서 onMainViewClicked로 시작된 코루틴의 실행을 제어.
+ MainCoroutineScopeRule을 사용하면 Disapatchers.Main에서 실행되는 코루틴의 실행을 resume, pause 또는 control할 수 있음.
+ AdvanceTimeBy(1_000)을 호출로 메인 디스패쳐가 1초 후 다시 시작하도록 예약하는 코루틴을 즉시 실행하도록 함.
+ 이 테스트는 완전 결정적(항상 같은 방식으로 동작함)이고, 메인 디스패쳐에서 시작된 코루틴의 실행을 완전 제어할 수 있기 때문에 값이 설정되기까지 1초를 기다릴 필요가 없음.

### Use coroutine :

+ 감싸는 코루틴이 없는 상태에서 첫 코루틴을 만들땐 launch로 시작하는 것을 권장.
  -> uncaught exception 발생시 자동으로 에러가 uncaught exception handler에 전달됨(기본적으로 앱 크러쉬 발생).
  async로 시작된 코루틴은 await를 호출할 때까지 호출자에게 예외를 throw하지 않음. await 함수는 suspend 함수이기 때문에 코루틴 내부에서만 호출가능.
+ 코루틴 내부에서 launch 또는 async로 자식 코루틴을 시작할 수 있음.
  리턴값이 필요없을 땐 launch를 사용, 필요할땐 async 사용.
+ 코루틴은 suspend함수를 실행하면서 일시정지한 것처럼 동작. suspend함수의 완료와 함께 재개됨.
  -> 일반 블로킹 함수의 호출 형태처럼 보이지만 메인 스레드를 블락하지 않고 suspend 함수의 완료를 자동으로 대기함.
+ suspend함수에서의 예외는 일반함수 오류처럼 작동. 호출자로 에러가 전달됨. try/catch 블록으로 처리 가능.
  기본언어로 제공되는 에러 핸들링이 가능하기 때문에 모든 콜백에 대한 오류 처리를 대신할 수 있다는 이점이 생김.
+ 코루틴에서 예외가 발생하면 기본적으로 부모 코루틴까지 취소됨.
  ->여러 관련 작업을 함께 취소하기 용이.
+ try/catch의 finally 블록에서 spinner 중단과 같은 작업이 용이.

### Uncaught exceptions

+ 코루틴에서의 uncaught exception은 논코루틴 코드에서의 예외와 비슷. 기본적으로 코루틴 Job을 취소하고 부모 코루틴에 스스로 취소해야함을 알림.
  코루틴이 에외를 처리하지 않으면 CoroutineScope의 uncaught exception handler로 전달됨.
+ 기본적으로 uncaught exception은 JVM에서 스레드의 uncaught exception handler로 전달됨. CoroutineExceptionHandler 제공으로 이 동작을 커스텀 가능.

### Dispatcher 간 스위칭

+ 코루틴 내 withContext() 호출로 디스패쳐를 스위칭할 수 있음. 람다에 대해서만 새롭게 호출되는 디스패쳐로 전환된 다음, 람다의 결과는 withContext를 호출한 기존 디스패쳐로 반환됨.
  -> withContext의 result 또는 error를 핸들링하면 됨.
+ 기본적으로 Kotlin 코루틴은 `Main`,`IO`,`Default` 세 가지 Dispatcher를 제공.
  `IO 디스패쳐` : 네트워크 또는 디스크 읽기와 같은 IO 작업에 최적화
  `Default 디스패쳐` : CPU intensive 작업에 최적화

```kotlin
//디스패쳐 스위칭 예 : (블록킹 호출 최적화 전)
suspend fun refreshTitle() {
   // interact with *blocking* network and IO calls from a coroutine
   withContext(Dispatchers.IO) {
       val result = try {
           // Make network request using a blocking call
           network.fetchNextTitle().execute()
       } catch (cause: Throwable) {
           // If the network throws an exception, inform the caller
           throw TitleRefreshError("Unable to refresh title", cause)
       }
      
       if (result.isSuccessful) {
           // Save it to database
           titleDao.insertTitle(Title(result.body()!!))
       } else {
           // If it's not successful, inform the callback of the error
           throw TitleRefreshError("Unable to refresh title", null)
       }
   }
}
```

+ withContext(Dispatcher.IO){} 블록 안에서 호출된 execute/insertTitle 함수들은 블록킹 함수이기 때문에 Dispatcher.IO의 스레드 중 하나를 차단.

### Advanced tip :

> 위 코드에선 코루틴 취소를 지원하지 않지만, 취소를 위해 kotlinx-coroutines에서 함수 호출 시 마다 발생하는 취소를 명시적으로 코드에서 확인해야 함.
>
> yield 호출로 다른 코루틴 취소를 확인하고 실행할 수 있음.
> 위 코드에선 네트워크 요청과 데이터베이스 쿼리 사이에 yield를 위한 호출 추가.
>
> 저수준 코루틴 인터페이스를 만들 때 수행해야 하는 취소를 명시적으로 확인할 수 있음.

+ Room과 Retrofit은 suspend 기능을 지원하므로 간단히 suspend modifier를 붙여 main-safe하게 만들 수 있음.
  Room은 쿼리를 자동으로 백그라운드 스레드에서 동작하도록 함.
  Retrofit도 suspend 수정자를 바꾸는것 만드로 main-safe하게 만들 수 있음. 백그라운드 스레드에서 요청 및 실행. 또한 리턴 타입을 Call<String> 타입에서 Call wrapper를 지운 것으로 사용할 수도 있음.
+  Room과 Retrofit은 커스텀 dispatcher를 사용(Dispatcher.IO 사용X).
  Room : 기본 query와 transaction Executor를 사용해 코루틴을 실행.
  Retrofit : 요청을 비동기로 보내기 위해 새로운 Call 객체를 만들고 enqueue를 호출.
+ 이처럼 편의상, app에서 작성하는 suspend 함수를 기본적으로 main-safe를 보장해놓으면 어느 디스패쳐에서 호출하든 안전하기 때문에 권장됨.
+ scope를 적절한 곳에 만들어서 사용하는 경우 취소 가능하게 만드는 것을 잊지말자.
  리사이클러뷰 어답터에서 DiffUtil연산을 사용하는 경우 등.

### 고차함수로 보일러플레이트 줄이기

중복되는 show & hide spinner / display error 코드를 고차함수를 사용해 줄이는 예제 :

```kotlin
private fun launchDataLoad(block: suspend () -> Unit): Job {
   return viewModelScope.launch {
       try {
           _spinner.value = true
           block()
       } catch (error: TitleRefreshError) {
           _snackBar.value = error.message
       } finally {
           _spinner.value = false
       }
   }
}

fun refreshTitle() {
   launchDataLoad {
       repository.refreshTitle()
   }
}
```

+ 람다로 반복되는 로직을 캡슐화하는 이러한 추상화 작성이 편리해짐.

### WorkManager와 코루틴 사용

+ WorkManager는 연기 가능한 백그라운드 작업에 유연하고 호환가능한 심플 라이브러리. 권장됨.
  ->WorkManager는 `가능한 한 빠른 백그라운드 작업 수행`, `사용자의 앱 이탈 등의 다양한 상황에서의 작업 보장의 조합이 필요한 백그라운드 작업`을 위한 아키텍처 컴포넌트.
+ 반드시 완료되어야 하는 작업에 적합. 
  예시 :
  1. 로그 업로딩
  2. 이미지 필터를 적용하고 이미지 저장
  3. 주기적으로 이뤄지는 로컬 데이터와 네트워크 동기화
+ 가장 심플한 Worker 클래스를 통해 WorkManager에서 일부 동기 작업을 실행할 수 있음.
+ CoroutineWorker 클래스를 사용하면 doWork() 함수를 suspend함수로 정의하면서 WorkManager를 사용할 수 있음.

```kotlin
// RefreshMainDataWork.kt (extends CoroutineWorker) 예시 : 
override suspend fun doWork(): Result {
   val database = getDatabase(applicationContext)
   val repository = TitleRepository(network, database.titleDao)

   return try {
       repository.refreshTitle()
       Result.success()
   } catch (error: TitleRefreshError) {
       Result.failure()
   }
}
```

+ note : CoroutineWorker.doWork() 함수는 suspend 함수로, Woker 클래스와 달리 WorkManager config에 지정된 Executor에서 실행되지 않음. 대신 Dispatchers.Default를 사용함. 물론 withContext()를 통해 dispatcher switching 가능.



