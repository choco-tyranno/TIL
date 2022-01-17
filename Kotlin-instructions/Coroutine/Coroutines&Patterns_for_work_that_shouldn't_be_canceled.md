# Coroutines & Patterns for work that shouldn't be canceled

+ Android Jetpack에서 제공하는, 각 특정 lifecycle에 자동으로 연결되어 관리되는, CoroutineScope인 viewModelScope와 lifecycleScope를 사용하지 않고
  고유한 CoroutineScope를 생성해 사용한다면 Job에 연결하고 필요할 때 취소를 호출하도록 해야함.
+  데이터베이스 쓰는 작업 또는 서버로 특정 네트워크 요청을 만드는 등 사용자가 해당 앱을 잠시 뒤로하고 다른 앱 등으로 이동해 떠난 경우에도 완료되어야 할 작업들의 특별 관리가 필요함.

## Coroutine vs WorkManager

+ 코루틴은 앱 프로세스가 살아있는 동안 실행됨.
+ 프로세스 보다 오래 지속되어야 하는 작업을 실행해야 하는 경우, 안드로이드에선 WorkManager를 사용할 것을 권장.
  WorkManager는 미래 어느 시점에서 실행될 것이라고 예상되는 중요한 작업에 사용하는 라이브러리.
+ 코루틴은 현재 프로세스에서 유효한, 사용자가 앱 종료시 취소될 수 있는 작업에 사용하는 것을 권장.
  (예 : 캐싱을 위한 네트워크 요청 등)

## 코루틴의 좋은 사례 :

1. 새로운 코루틴을 만들거나 withContext를 호출할 때, Dispatcher를 하드코딩 피하기.
   -> 테스트 시 쉽게 교체 가능한 이점.
2. ViewModel/Presenter 레이어에서 코루틴을 만들기.
   -> UI계층이 비즈니스 로직을 직접 트리거하지 않도록해야 함. ViewModel/Presenter레이어가 대신하도록 맡김.
3. VM/Presenter 레이어에서 suspend 함수와 Flow들을 노출.
   -> 호출자(일반적으로 VM 레이어)는 해당 계층에서 발생하는 작업의 실행 및 수명주기를 제어할 수 있으며, 필요시 취소할 수 있음.

## 코루틴에서 취소되면 안되는 작업

+ Application 클래스에서 고유 scope를 생성하고 이 scope에서 작업을 호출하면 됨. 이 scope는 필요로하는 클래스에 주입.
+ 고유 CoroutineScope의 이점(GlobalScope아 같은 다른 솔루션을 사용했을 때와 비교해서)은 원하는 대로 Config할 수 있다는 점( 예 : CoroutineExceptionHandler가 필요한 경우/ Dispatcher로 사용되는 자체 Thread pool이 있는 경우 등).
  -> `applicationScope`라고 부를 수 있고, SupervisorJob()을 포함해야 코루틴에서의 실패가 계층구조로 전파되지 않음.

```kotlin
class MyApplication : Application() {
  // No need to cancel this scope as it'll be torn down with the process
  val applicationScope = CoroutineScope(SupervisorJob() + otherConfig)
}
// applicationScope는 앱 프로세스의 끝남과 함께 취소됨.
```

+ Repository 레이어에서 주로 사용.

## 코루틴 빌더 고르기

+ 작업에 따라 launch 또는 async를 사용해 새로운 코루틴을 만들어야 하는 경우도 있음.
  결과값을 반환해야하면 async와 await를, 그렇지 않으면 launch와 join(작업이 끝나는 것을 기다리는 연산자)을 사용.

```kotlin
//launch를 사용해 코루틴을 트리거한 예시 :
class Repository(
  private val externalScope: CoroutineScope,
  private val ioDispatcher: CoroutineDispatcher
) {
  suspend fun doWork() {
    withContext(ioDispatcher) {
      doSomeOtherWork()
      externalScope.launch {
        // 예외를 던질 수 있는 경우, try/catch롤 감싸거나
        // externalScope의 코루틴스코프에 내장된 CoroutineExceptionHandler에 의존.
        veryImportantOperation()
      }.join()
    }
  }
}
```

```kotlin
// async를 사용한 사례 :
class Repository(
  private val externalScope: CoroutineScope,
  private val ioDispatcher: CoroutineDispatcher
) {
  suspend fun doWork(): Any { // 구체적인 결과 타입 설정필요.
    withContext(ioDispatcher) {
      doSomeOtherWork()
      return externalScope.async {
        // await가 호출되면 이 예시에서 dowork를 호출한 호출자 코루틴에 예외가 전파됨.
        // 단, 호출 컨텍스트가 취소되면 무시될 수 있음에 유의할 것.
        veryImportantOperation()
      }.await()
    }
  }
}
```

+ 위 예에서 ViewModelScope가 사라지더라도 externalScope를 사용한 작업은 계속 진행됨.
+ veryImportantOperation()이 완료될 때까지 기다림.

### 조금 더 간단하게 : scope.launch 대신 withContext(scope.coroutineContext) 사용

```kotlin
class Repository(
  private val externalScope: CoroutineScope,
  private val ioDispatcher: CoroutineDispatcher
) {
  suspend fun doWork() {
    withContext(ioDispatcher) {
      doSomeOtherWork()
      withContext(externalScope.coroutineContext) {
        veryImportantOperation()
      }
    }
  }
}
```

+ ⚠️주의 사항 :
  + veryImportantOperation()이 실행된 동안 doWork를 호출한 코루틴이 취소되면,
     veryImportantOperation()의 끝난 뒤가 아닌 다음 취소 포인트까지 실행이 유지됨. 
  + `CoroutineExceptionHandler`가 예상한대로 동작하지 않음. withContext에 context가 사용되면 예외가 다시 던져지기 때문.

## 대안 :

### GlobalScope와 주의사항 :  

다이렉트로 사용하지 않을 것을 권장

+ 하드 코딩 값을 조장함. GlobalScope를 곧바로 사용하는 경우 Dispatchers를 하드코딩하도록 부추길 수 있음. 안좋은 습관.
+ 이는 테스팅을 매우 어렵게 만듬. 코드가 제어 불가한 scope에서 실행될 것이기 때문에 작업 실행을 관리할 수 없음.
+ applicationScope에서 했던 것처럼 scope에 포함되는 모든 코루틴들을 위한 공통 코루틴컨텍스트를 갖을 수 없음.
  대신, 모든 코루틴에 공통 CoroutineContext를 전달해야 함.

### ProcessLifecycleOwner scope와 주의사항 :

다이렉트로 사용하지 않을 것을 권장

+ `androidx.lifecycle:lifecycle-process` 라이브러리에 applicationScope가 있음.
  `ProcessLifecycleOwner,get().lifecycleScope` 로 접근가능.
+ CoroutineScope 대신 LifecycleOwner를 inject함.
+ 프로덕션에선 ProcessLifecycleOwner.get()을 전달하고 unii test에선 LifecycleRegistry를 사용해 가짜 LifecycleOwner를 만들 수 있음.
+ 이 스코프의 기본 코루틴컨텍스트는 Dispatchers.Main.immediate를 사용함(백그라운드 작업에 알맞지 않을 수 있음).
+ GlobalScope와 마찬가지로 GlobalScope에서 시작한 모든 코루틴에 공통 CoroutineContext를 전달해야 함.
+ 위 같은 이유들로 Application 클래스에서 고유한 CoroutineScope를 만드는 것보다 많은 작업이 수반됨.
+ ViewModel/Presenter 아래의 레이어에 Android 라이프사이클이 관련된 클래스를 두지 않는 것을 선호.
  이러한 레이어는 플랫폼에 구애받지 않아야 하기 때문.

+ Disclaimer(포기자?)
  applicationScope의 CoroutineContext가 GlobalScope 또는 ProcessLifecycleOwner.get().lifecycleScope의 것과 같다고 판명된다면,
  다음과 같이 할당 가능.

  ```kotlin
  class MyApplication : Application() {
    val applicationScope = GlobalScope
  }
  //이렇게 하면 위에서 언급한 이점들을 모두 얻을 수 있고, 미래에 필요한 경우 쉽게 변경할 수 있음.
  ```

### NonCancellable 과 주의사항 : 

suspending cleanup code에만 사용할 것을 권장.

+ withContext(NonCancellable)를 사용하면 취소된 코루틴에서 suspend함수를 호출할 수 있음.
+ 코드를 정리하는 동작을 하는 데에 이것을 사용할 것을 제안함. 남용 하지말고.
+ 코루틴 실행에 대한 통제력을 상실하기 때문에 매우 위험함.
+ 더 간결하고 읽기 쉬운 코드를 만들 수 있긴하지만, 미래에 발생할 문제를 예측할 수 없음.

```kotlin
//NonCancellable 예시 :
class Repository(
  private val ioDispatcher: CoroutineDispatcher
) {
  suspend fun doWork() {
    withContext(ioDispatcher) {
      doSomeOtherWork()
      withContext(NonCancellable) {
        veryImportantOperation()
      }
    }
  }
}
```

+ 🤔 간결해보여서 좋지만, veryImportantOperation()에 외부 라이브러리가 있을지, 구현이 인터페이스로 추상화되어있을지 알 수 없기 때문에
  무슨 문제가 발생할 수 있을 지 알 수 없음.

  문제점들 :

  1. 테스트에서 이러한 작업을 중지할 수 없음
  2. delay를 사용하는 무한 루프는 더 이상 취소할 수 없음.
  3. 이 내부에서 Flow를 Collect하면 외부에서 Flow를 취소할 수 없음.

  이러한 문제는 미묘하고 디버그하기 매우 어려운 버그로 이어질 수 있음.



> 결론 : 
>
> 현재 스코프를 넘어서는 작업이 필요한 경우 Application 클래스 안에 커스텀 스코프를 만들어서 사용할 것을 권장.
>
> 이러한 작업에선 GlobalScope, ProcessLifecycleOwner scope, NonCancellable을 피하는 것이 좋음.