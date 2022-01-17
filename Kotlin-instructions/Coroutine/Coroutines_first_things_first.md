# Coroutines: first things first

## CoroutineScope : 

+ 코루틴스코프는 CoroutineScope의 extension function인 `launch` 또는 `async`를 사용해 생성한 코루틴을 추적함.
+ 진행중인 코루틴 작업은 scope.cancel() 호출로 언제나 취소할 수 있음.
+ 앱의 특정 레이어에서 코루틴의 라이프사이클을 시작 및 제어를 원할 때 마다 코루틴스코프를 만들어야 함.
  안드로이드에는 특정 라이프사이클 클래스와 함께 CoroutineScope를 제공하는 KTX라이브러리가 있음.
  (예 : viewModelScope와 lifecycleScope 등..)
+  코루틴스코프를 생성할 때 생성자 매개변수로 CoroutineContext를 받음.

```kotlin
// 새 스코프와 코루틴을 만드는 예시 :
// Job과 Dispatcher는 CoroutineContext로 결합됨. 
val scope = CoroutineScope(Job() + Dispatchers.Main)
val job = scope.launch {
    // new coroutine
}
```

## Job :

+ Job은 코루틴 핸들임.
+ launch 또는 async로 생성한 모든 코루틴은 고유하게 식별가능하고 라이프사이클을 인식하는 Job 인스턴스를 반환함.
+ Job을 코루틴스코프에 전달해 라이프사이클을 처리할 수 있음.

## CoroutineContext :

+  코루틴컨텍스트는 코루틴의 동작을 정의하는 요소들의 집합임.
  구성 :
  + Job - 코루틴 라이프사이클을 제어.
  + CoroutineDispatcher - 적절한 스레드로 작업을 보냄.
  + CoroutineName - 코루틴의 이름. 디버깅에 유용.
  + CoroutineExceptionHandler - 잡히지 않는 예외를 다룸.
+ 새 코루틴의 코루틴컨텍스트는 Job의 새 인스턴스가 생성되며 라이프사이클을 제어할 수 있게됨.
  나머지 요소들은 부모의 코루틴컨텍스트(다른 코루틴 또는 생성된 코루틴 스코프)로부터 상속됨.
+ 코루틴스코프는 코루틴들을 생성할 수 있고, 코루틴 안에 코루틴들을 더 생성할 수 있기 때문에 암시적 작업 계층이 형성됨.

```kotlin
// 코루틴스코프는 사용해 새로운 코루틴을 만들고, 코루틴 내부에 코루틴들을 더 생성하는 예시 :
val scope = CoroutineScope(Job() + Dispatchers.Main)
val job = scope.launch {
    // New coroutine that has CoroutineScope as a parent
    val result = async {
        // New coroutine that has the coroutine started by 
        // launch as a parent
    }.await()
}
```

+ 계층의 루트는 일반적으로 코루틴스코프임.
  시각화 : 
  <img src="https://miro.medium.com/max/700/0*cIlNrjUhl4aZjrkJ"/>

+ 코루틴들은 작업 계층에서 실행됨. 부모는 코루틴스코프 또는 다른 스코프.

## Job lifecycle

+ Job의 상태 : New, Active, Completing, Completed, Cancelling and Cancelled.
+ 상태 자체에 대한 접근은 불가하지만, isCancelled 및 isCompleted와 같은 Job 프로퍼티에 접근할 수 있음.

Job 라이프사이클 :

<img src="https://miro.medium.com/max/700/0*zGHzocA6-lCxk0aX"/>

+ 코루틴이 active 상태일 때, 코루틴의 실패 또는 job.cancel() 호출로 Job을 Cancelling 상태로 이동시킬 수 있음.
  (isActive = false, isCancelled = true)
+ Cancelled상태에서도 isCompleted = true임.

## 부모 코루틴컨텍스트 :

+ 작업 계층에서 각 코루틴에는 코루틴스코프 또는 다른 코루틴이 부모로 있음.
+ 코루틴컨텍스트는 다음 공식을 기반으로 계산되기 때문에 자식의 코루틴컨텍스트와 부모의 코루틴컨텍스트가 달라질 수 있음.

> Parent context = Defaults + inherited CoroutineContext + arguments
>
> + 요소 중 일부는 `default` 값을 갖음:  CoroutineDispatcher의 default는 `Dispatchers.Default`이고,
>   CoroutineName의 default는 coroutine임.
> + `상속된 코루틴컨텍스트`는 코루틴스코프 또는 해당 코루틴을 생성한 코루틴의 코루틴컨텍스트임.
> + `인수`는 코루틴빌더에 보낸 것으로, 상속된 컨텍스트의 요소들 보다 우선됨.

+ Note : 코루틴컨텍스트는 `'+'` 연산자를 사용해 결합될 수 있음.
  코루틴컨텍스트는 요소 집합이므로 + 연산자 오른쪽 요소가 왼쪽에 있는 요소를 재정의하여 새 코루틴컨텍스트가 생성됨.
  (예 : (Dispatchers.Main, "이름") + (Dispatchers.IO) = (Dispatchers.IO, "이름") )

코루틴 스코프에 의해 시작된 모든 코루틴은 코루틴컨텍스트에 적어도 아래와 같은 요소들을 갖음.(아래 예시에서 name : "coroutine"는 default 값으로 들어갔음) : 

<img src="https://miro.medium.com/max/700/0*gjkoETUbx4mPhYEF"/>

> New coroutine context = parent CoroutineContext + Job()

```kotlin
//위에서 정의한 코루틴스코프를 가지고 새 코루틴을 만든 예시 :
val job = scope.launch(Dispatchers.IO) {
    // new coroutine
}
```

+ 코루틴컨텍스트 및 부모 컨텍스트의 Job은 새 코루틴과 동일한 인스턴스일 수 없음. 새 코루틴은 항상 Job의 새 인스턴스를 얻음 :

<img src="https://miro.medium.com/max/700/0*LPnIOfVGQqrqPZ__"/>



