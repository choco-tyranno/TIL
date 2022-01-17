# Cancellation in coroutines

+ 필요 이상의 더 많은 작업 실행으로 메모리와 에너지를 낭비하는 것을 막기 위해 코루틴에선 라이프사이클을 제어하고, 코루틴이 더 이상 필요하지 않을 때 취소가 필요함. 구조화된 동시성(structured concurrency)이 나타내려고 하는 것이 이런 것.

## Cancel 호출하기

+ 스코프를 취소하면 해당 스코프에서 실행되고 있는 자식 코루틴들이 모두 취소됨.

```kotlin
// assume we have a scope defined for this layer of the app
val job1 = scope.launch { … }
val job2 = scope.launch { … }
scope.cancel()
```

+ 특정 코루틴만 취소하려면 `job.cancel()` 호출(형제 코루틴에 영향 없음).

```kotlin
// assume we have a scope defined for this layer of the app
val job1 = scope.launch { … }
val job2 = scope.launch { … }
// First coroutine will be cancelled and the other one won’t be affected
job1.cancel()
```

+ 코루틴은 특별한 예외인 CancellationException을 발생시켜 취소를 처리.

+ 취소 사유에 대한 자세한 내용을 제공하려면 .cancel을 호출할 때 다음과 같은 전체 메서드 시그니쳐로 CancellationException 인스턴스를 제공할 수 있음.

  > fun cancel(cause: CancellationException? = null)

+ 자체 CancellationException 인스턴스를 제공하지 않는다면, 기본 `CancellationException`이 생성됨.

  >public override fun cancel(cause: CancellationException?) {
  >    cancelInternal(cause ?: defaultCancellationException())
  >}

+ CancellationException이 던져지기 때문에, 이러한 메커니즘을 사용해서 코루틴 취소를 처리할 수 있음.

+ 내부적으로 자식 Job은 예외를 통해 취소를 부모에 알림.

+ 부모는 취소의 원인을 사용해서 예외 처리가 필요한지를 결정할 수 있음.

+ CancellationException으로 인해 자식이 취소된 경우, 부모에서 별다른 조치가 필요하지 않음.

+ Scope를 한번 취소하면, 그 취소된 스코프에서 새로운 코루틴을 실행할 수 없음.

+ 자체 스코프를 사용하지 않고 Android KTX 라이브러리를 사용하는 경우 취소 처리를 따로할 필요가 없음.

+ ViewModel 스코프에서 작업하는 경우 `viewModelScope`를, 라이프사이클 스코프에 연결된 코루틴을 시작하려면 `lifecycleScope`를 사용하면 됨.

+ viewModelScope와 lifecycleScope는 관련된 적절한 시기에 취소되는 CoroutineScope 객체임.
  예 : ViewModel이 cleared되면 viewModelScope에서 launch된 코루틴들이 취소됨.

## 코루틴 취소와 작업 정지

+ 취소를 호출한다고 코루틴 작업이 바로 중지되는 것음 아님.

```kotlin
// 코루틴을 1000ms동안 실행하고 job.cancel()을 호출하지만 작업은 계속 진행되고 작업이 완료된 후에 코루틴이 Canceling 상태가 됨.
import kotlinx.coroutines.*
 
fun main(args: Array<String>) = runBlocking<Unit> {
   val startTime = System.currentTimeMillis()
    val job = launch (Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) {
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("Hello ${i++}")
                nextPrintTime += 500L
            }
        }
    }
    delay(1000L)
    println("Cancel!")
    job.cancel()
    println("Done!")
}
```

+ 주기적으로 코루틴이 활성상태인지 체크하는 코드가 필요함.

## 코루틴 작업 취소가능하게 만들기 :

+ 취소에 협력적인 코루틴 작업을 만들기위해선, 주기적으로 취소를 확인하거나 작업 전에 취소를 확인해주는 것이 필요.

```kotlin
//CPU 집약적인 작업을 진행하기 전에 취소를 확인.
val job = launch {
    for(file in files) {
        // TODO check for cancellation
        readFile(file)
    }
}
```

+ kotlinx.coroutines의 suspend 함수(예 : withContext, delay 등..)는 모두 취소 가능함.
  -> 따로 취소를 체크하거나, 작업 실행을 중지하거나, CancellationException을 던질 필요가 없음.
+ 코루틴 라이브러리에서 제공하는 suspend함수를 사용하지 않는 경우, 코루틴 코드를 취소에 협력적으로 하기 위한 두 가지 방안이 있음.
  1. `job.isActive` 또는 `ensureActive`를 체크.
  2. `yield()`를 사용해 다른 작업이 수행될 수 있게 하기.

## Job의 활성상태 체크하기 :

```kotlin
//launch 블럭 안에서 job.isActive에 접근가능.
while (i < 5 && isActive)
```

+ while 블락 밖에서 !isActive를 체크함으로써 작업이 취소된 경우에 필요한(예 : logging) 동작을 추가할 수도 있음.

```kotlin
// 코루틴 라이브러리에서 제공되는 메서드 : ensureActive()
fun Job.ensureActive(): Unit {
    if (!isActive) {
         throw getCancellationException()
    }
}
```

-> ensureActive 메서드는 Job이 활성상태가 아닌 경우 즉시 throw함.

```kotlin
while (i < 5) {
    ensureActive()
    …
}
```

+ isActive를 확인하는 if문을 직접구현하지 않아도 되므로 상용구 코드가 줄어들지만, 로깅과 같은 다른 작업이 필요한 경우의 유연성을 잃게 됨.

## yield()를 사용해 다른 작업이 진행되도록 하기 :

+ 작업이 1)CPU 과중되고 , 2) 스레드 풀 소진 가능성이 있고, 3) 스레드 풀에 스레드를 추가하지 않고도 스레드가 다른 작업을 할 수 있도록 하려는 경우 yield()를 사용할 수 있음.
+ Job이 이미 완료된 경우에, yield에 의해 수행되는 첫번째 작업은 완료를 체크하고 CancellationException를 던짐으로써 코루틴을 빠져나감.
+ yield는 ensureActive()와 같이 주기적 체크에서 첫번째로 호출되는 함수가 될 수 있음.

## Job.join vs Deferred.await의 취소

+ 코루틴으로부터의 결과를 기다리는 방법으로, 1) launch로부터 반환되는 Job의 join 호출, async로부터 반환되는 Deferred(Job 타입 중 하나)의 await() 호출이 있음.
+ job.cancel 이후 Job.join이 호출되면 작업이 완료될 때까지 코루틴이 suspend됨.
  job.join 이후의 job.cancel 호출은 효과가 없음(join 호출로 인해 해당 job 완료시까지 job.cancel에 닿지않기 때문).
+ 취소된 deferred에서 await를 호출하면 JobCancellationException이 발생함.

```ko
val deferred = async { … }
deferred.cancel()
val result = deferred.await() // throws JobCancellationException!
```

## 취소 사이드-효과 처리하기 : 

+ 코루틴이 취소됐을 때 특정 동작이 실행되어야 할 수 있음는 경우.
  (예 : 사용한 리소스를 닫거나, 취소를 기록하거나, 실행하려는 정리 코드를 기록하려는 경우)

### !isActive로 체크

+ isActive로 주기적인 체크를 하고 있는 경우, while 루프 밖에서 리소스들을 정리할 수 있음.

```kotlin
while (i < 5 && isActive) {
    // print a message twice a second
    if (…) {
        println(“Hello ${i++}”)
        nextPrintTime += 500L
    }
}
// the coroutine work is completed so we can cleanup
println(“Clean up!”)
```

### Try catch finally

+ 코루틴이 취소되면 CancellationException을 던지기 때문에 suspending 작업을 try/catch로 감싸고 finally 블럭으로 정리 작업을 구현할 수 있음.

```kotlin
val job = launch {
   try {
      work()
   } catch (e: CancellationException){
      println(“Work cancelled!”)
    } finally {
      println(“Clean up!”)
    }
}
delay(1000L)
println(“Cancel!”)
job.cancel()
println(“Done!”)
```

+ 취소가 발생했을 때 실행해야하는 정리 작업이 suspend라면 동작하지 않음.
  코루틴이 Cancelling 상태에 있으면 더이상 suspend는 불가.
+ 코루틴이 취소되었을 때, suspend함수를 호출해 정리작업을 하려면, NonCancellable CoroutineContext로 정리 작업을 전환해야 함.
  -> 작업이 완료될 때까지 Cancelling 상태에서 suspend 코드와 코루틴이 유지됨. 

```kotlin
//suspend 정리 작업을 위해 NonCancellable 코루틴컨텍스트에서 작업토록 함.
val job = launch {
   try {
      work()
   } catch (e: CancellationException){
      println(“Work cancelled!”)
    } finally {
      withContext(NonCancellable){
         delay(1000L) // or some other suspend fun 
         println(“Cleanup done!”)
      }
    }
}
delay(1000L)
println(“Cancel!”)
job.cancel()
println(“Done!”)
```

## suspendCancellableCoroutine과 invokeOnCancellation

+ suspendCoroutine 메서드를 사용해 콜백을 코루틴으로 변환하는 경우, 대신 suspendCancellableCoroutine을 사용하는 것이 좋음.
+ 취소 시 수행할 작업은 continuation.invokeOnCancellation을 사용해 구현할 수 있음.

```kotlin
suspend fun work() {
   return suspendCancellableCoroutine { continuation ->
       continuation.invokeOnCancellation { 
          // do cleanup
       }
   // rest of the implementation
}
```





> 결론 : 
>
> 구조화된 동시성의 이점을 실현하고 불필요한 작업을 수행하지 않도록 하려면 코드를 취소할 수 있도록 만들어야 함.
>
> Jetpack에 정의된 CoroutineScope들을 사용할 것을 권장.
>
> 고유한 CoroutineScope를 사용하는 경우 Job에 연결하고 적절한 시점에서 취소를 호출하도록 해야 함.
>
> 코루틴의 취소는 협조적이어야 하므로, 취소가 지연되는지 확인하고 필요 이상으로 더 많은 작업이 수행되지 않도록 해야 함.

