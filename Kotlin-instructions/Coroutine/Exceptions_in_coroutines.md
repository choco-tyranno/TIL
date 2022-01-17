# Exceptions in coroutines

+ 작업이 예상대로 진행되지 않을 때에도 적절한 사용자 경험을 제공하는 것이 중요.
+ 작업이 성공하지 못했을 땐, 사용자에게 올바른 메시지를 표시해야 함.
+ 코루틴이 예외와 함께 실패하면, 예외를 부모에게 전파함.
  부모는 1)나머지 자식들을 취소하고, 2) 스스로를 취소, 3) 부모의 부모로 예외를 전파함.
+ 예외는 계층 구조의 루트에 도달하고, 코루틴스코프가 시작한 모든 코루틴들도 취소됨.

코루틴 예외의 전파와 스코프 작업의 전체 취소 :

<img src="https://miro.medium.com/max/700/0*UcEpsF2X-ihztU2Z"/>

+ UI-관련 코루틴스코프와 같이 위에서 살펴본 예외 전파가 적절하지 않은 경우도 있음.
  예 : UI관련 코루틴 스코프의 자식 코루틴에서 예외를 발생시키면->UI스코프는 취소되고->취소된 스코프에서 더이상 코루틴이 시작될 수 없는 상태가 되기 때문에 전체 UI 컴포넌트가 먹통이 됨.
  -> 대안으로 SupervisorJob을 사용해볼 수 있음.

## SupervisorJob

+ 자식의 실패가 다른 자식들에게 영향을 미치지 않음.
+ 스스로를 취소하지도 않고, 다른 자식들을 취소하지 않음.
+ 예외를 전파하지도 않음.

```kotlin
val uiScope = CoroutineScope(SupervisorJob())
```

<img src="https://miro.medium.com/max/700/0*Mrf17HLbWQPfTt1I"/>

+ 예외가 처리되지 않고, 코루틴컨텍스트가 CoroutineExceptionHandler를 갖지 않는 경우,
  예외는 default 스레드의 ExceptionHandler에 도달함.
  JVM에선 예외가 콘솔의 로그되고, 안드로이드에선 Dispatcher와 무관하게 앱 충돌이 발생함.
+ 잡히지 않은 예외는 사용하는 Job의 종류와 무관하게 던져짐.
+  스코프빌더 coroutineScope 및 supervisorScope에도 동일하게 적용됨.
+ ⚠️Caution : SupervisorJob은 스코프의 일부인 경우에만 작동함 :
  `supervisorScope` 사용 또는 `CoroutineScope(SupervisorJob())`.
+ SupervisorJob은 하나의 코루틴 실패가 부모 또는 형제의 작업을 취소하지 않길 바라는 경우에 적합.

```kotlin
// Scope handling coroutines for a particular layer of my app
// 특정 레이어에 대한 코루틴을 처리하는 스코프.
val scope = CoroutineScope(SupervisorJob())
scope.launch {
    // Child 1
}
scope.launch {
    // Child 2
}
//child1이 취소되어도 scope와 child2 모두 취소되지 않음.
```

```kotlin
// Scope handling coroutines for a particular layer of my app
val scope = CoroutineScope(Job())
scope.launch {
    supervisorScope {
        launch {
            // Child 1
        }
        launch {
            // Child 2
        }
    }
}
//위와 마찬가지로 child1의 취소가 scope의 취소 및 child2의 취소로 이어지지 않음.
```

### Watch out quiz!

: What is the Job kind of child1's parent?

```kotlin
val scope = CoroutineScope(Job())
scope.launch(SupervisorJob()) {
    // new coroutine -> can suspend
   launch {
        // Child 1
    }
    launch {
        // Child 2
    }
}
```

+ 정답 : Job
  SupervisorJob은 scope.launch로 생성된 코루틴의 부모일 뿐임.
  -> note : SupervisorJob은 scope의 일부인 경우에만 작동.
  SupervisorJob을 코루틴 빌더의 매개변수로 전달하면 원하는 대로 동작X.

<img src="https://miro.medium.com/max/700/0*CB9c0_BAhlSJpC7w"/>

## SupervisorJob의 내부 동작 :

[JobSupport.kt file - ](https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/common/src/JobSupport.kt#L645) childCancelled 함수와 notifyCancelling 함수 참고.

SupervisorJob 구현을 살펴보면 childCancelled 메서드에서 false를 반환함( : 취소를 전파하지 않음 & 예외를 처리하지 않음).

##  예외 처리하기 :

+ 코틀린에선 예외를 처리하기 위해 다음 문법을 사용 : `try/catch` 또는 `runCatching`(try/catch에서 내부적으로 사용하는 빌트인 helper 함수)

+ 잡히지 않은 예외는 항상 발생할 수 있지만, 다른 코루틴 빌더는 예외를 다른 방식으로 처리하기도 함 :

  + Launch : launch와 함께 예외가 즉시 발생함. try/catch 내부에서 예외를 throw할 수 있는 코드를 감쌀 수 있음.

    ```kotlin
    scope.launch {
        try {
            codeThatCanThrowExceptions()
        } catch(e: Exception) {
            // Handle exception
        }
    }
    ```

  + Async : async가 루트 코루틴으로 사용된 경우(CoroutineScope 인스턴스 또는 supervisorScope의 직접 자식으로서),
    예외는 자동으로 throw되지 않고 `.await()`를 호출함으로써 throw됨.
    루트 코루틴일 때, async로 throw된 예외를 처리하려면, .await() 호출을 try/catch 내부에 래필할 수 있음.

    ```kotlin
    supervisorScope {
        val deferred = async {
            codeThatCanThrowExceptions()
        }
        try {
            deferred.await()
        } catch(e: Exception) {
            // Handle exception thrown in async
        }
    }
    // await하지 않는 경우에는 예외가 throw되지 않으므로 try/catch로 wrapping할 필요 없음.
    //SupervisorJob은 코루틴에서 예외를 처리토록 함(전파X). 상위 계층으로 예외를 전파함으로써 catch블록이 호출되지 않는 Job과 대조됨.
    ```

    ```kotlin
    coroutineScope {
        try {
            val deferred = async {
                codeThatCanThrowExceptions()
            }
            deferred.await()
        } catch(e: Exception) {
            // Exception thrown in async WILL NOT be caught here 
            // but propagated up to the scope
        }
    }
    ```

    + 다른 코루틴에 의해 생성된 코루틴에서 발생한 예외는 코루틴빌더와 무관하게 항상 전파됨. 
      예 :
    
    ```kotlin
    val scope = CoroutineScope(Job())
    scope.launch {
        async {
            // async가 예외를 throw하면, .await() 호출 없이도 launch가 예외를 throw함.
        }
    }
    // 스코프의 직계 자식 코루틴이 launch이기 때문에, async에서 예외를 throw하면 즉시 예외가 발생함. 
    ```
    
    + ⚠️코루틴스코프 빌더 또는 다른 코루틴에 의해 생성된 코루틴에서 던져진 예외는 try/catch에 잡히지 않음.
    
    ## CoroutineExceptionHandler
    
    + 코루틴컨텍스트의 옵션 항목. 잡히지 않은 예외를 다루는데 사용됨.
    + 예외가 잡힐 때 마다, 예외가 발생한 코루틴컨텍스트와 예외 자체에 대한 정보를 가짐.
    
    ```kotlin
    // CoroutineExceptionHandler 정의하는 방법 : 
    val handler = CoroutineExceptionHandler {
        context, exception -> println("Caught $exception")
    }
    ```
    
    + 다음 요구사항과 함께 예외가 잡힘.
    
      + When : 예외를 자동으로 던지는 코루틴에 의해 예외가 던져짐(launch와 함께 작동O, async는 작동X).
      + Where : 코루틴스코프의 코루틴콘텍스트 또는 루트 코루틴 내부에 있음(코루틴스코프 또는 supervisorScope의 직접 자식).
    
      ```kotlin
      // handler에 의해 예외가 잡힘.
      val scope = CoroutineScope(Job())
      scope.launch(handler) {
          launch {
              throw Exception("Failed coroutine")
          }
      }
      ```
    
      내부 코루틴에 설치된 핸들러의 다음 경우에는 잡히지 않음 :
    
      ```kotlin
      val scope = CoroutineScope(Job())
      scope.launch {
          launch(handler) {
              throw Exception("Failed coroutine")
          }
      }
      // 올바른 코루틴콘텍스트에 핸들러가 설치된 것이 아니기 때문에 예외가 안잡힘.
      // 내부 launch는 예외를 부모로 전파하고, 부모는 handler에 대해 모르는 상황이기 때문에 결국 예외는 핸들링되지 못하고 던져짐.
      ```



> 결론 :
>
> 앱에서 우아한 예외 처리는 좋은 UX를 만드는 데 중요함.
>
> 취소가 전파된는 것을 피하고 싶다면 Job 대신 `SupervisorJob`을 사용하자.
>
> 전파되는 잡히지 않은 예외들을 잡아내서 훌륭한 UX를 만들어내자.







