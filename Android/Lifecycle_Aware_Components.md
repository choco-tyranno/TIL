# Lifecycle-Aware Components

### 라이프사이클 인식 구성요소 권장사항 : 

> + UI컨트롤러(액티비티, 프래그먼트)를 가볍게 유지. 자체 데이터 확보X. ViewModel을 사용해 데이터 확보. UI컨트롤러는 LiveData 객체를 관찰해 변경사항을 뷰에 반영.
> + UI컨트롤러의 책임은 데이터 변경에 따라 뷰를 업데이트하거나 사용자 작업을 다시 ViewModel에 알리는 것(데이터기반 UI).
> + ViewModel에 데이터 로직 배치. ViewModel은 UI컨트롤러와 앱의 나머지 부분 간의 커넥터 역할. 다른 컴포넌트를 호출해 데이터를 가져온 후 결과를 다시 UI 컨트롤러에 제공(네트워크로부터 데이터 가져오기는 ViewModel의 책임 X ).
> + Databinding을 사용해 뷰와 UI컨트롤러 간 인터페이스를 깔끔하게 유지(Java로 하려면 Butter Knife와 같은 라이브러리로 보일러플레이트를 피하고 더 나은 추상화를 구현해라).
> + UI가 복잡하다면 UI 수정을 처리할 수 있는 presenter 클래스를 만드는게 좋음.
>   -> UI 컴포넌트 테스트가 더 쉬워짐.
> + ViewModel에서 View 또는 Activity 컨텍스트 참조 X. ViewModel은 Activity 등과 다른 라이프사이클을 갖기 때문에 액티비티 Leak가 발생하고 이에 따라 가비지 컬렉터가 제대로 액티비티를 제대로 처리 못함.
> + Kotlin Coroutine을 사용해 장기 실행 작업 및 비동기적으로 실행될 수 있는 기타 작업을 관리.

### 사용 사례 :

> + 대략적인 위치와 세분화된 위치 업데이트 간 전환. 앱의 포그라운드 백그라운드 상태에 따라위치 상태 전환.  라이프사이클 인식 컴포넌트 LiveData를 사용해 UI 자동 업데이트.
> + 동영상 버퍼링 중지와 시작. 라이프사이클 인식 컴포넌트를 사용하면 동영상 버퍼링을 최대한 빨리 시작하지만, 앱이 완전히 시작될 때까지 재생을 연기. 앱이 제거될 때 버퍼링 종료 가능.
> + 네트워크 연결시작과 중지. 앱이 포그라운드에 있는 동안 네트워크 데이터를 실기간으로 업데이트(스트리밍) 가능. 백그라운드로 가면 실시간 업데이트 자동 일시중지 가능.
> + 애니메이션 드로어블 일시중지와 재개. 앱이 백그라운드에 있는 동안 애니메이션 드로어블 일시중지를 처리하고, 포그라운드시 드로어블 재개 가능. 

+ 라이프사이클 인식 컴포넌트는 액티비티, 프래그먼트와 같은 다른 컴포넌트의 라이프사이클 상태 변결에 따라 작업을 실행.
+ 라이프사이클 인식 컴포넌트를 사용하면 일반적인 패턴에서의 라이프사이클 메서드에 쓰여야 했던 의존성 컴포넌트들의 코드들을 컴포넌트 요소 자체로 옮길 수 있음.
+ 어플리케이션은 안드로이드 작동 방식의 특성상 메모리 누수, 어플리케이션 비정상 종료 등을 피하기 위해 라이프사이클을 고려해야 함.
+ 안드로이드 프레임워크에 정의된 대부분의 앱 컴포넌트들에는 라이프사이클이 연결되어있음.
+ 라이프사이클은 OS 또는 프로세스에서 실행중인 프레임워크 코드에서 관리됨.
+ `android.lifecycle` 패키지에서 라이프사이클 인식 컴포넌트를 빌드 할 수 있는 클래스 및 인터페이스 제공.



## `LifeCycle`

+ `Lifecycle`은 액티비티나 프래그먼트 같은 컴포넌트의 라이프사이클 상태 관련 정보를 포함. 다른 객체가 이 상태를 관찰할 수 있게하는 클래스.
+ Lifecycle은 두 가지 기본 열거를 사용해 연결된 컴포넌트의 라이프사이클 상태를 추적함(하단 '안드로이드 액티비티 라이프사이클을 구성하는 상태 및 이벤트' 참고).
+ `Event :` 프레임워크 및 Lifecycle 클래스에서 전달되는 라이프사이클 이벤트. 이러한 이벤트는 액티비티와 프래그먼트의 콜백 이벤트에 매핑됨.
+ `State : ` Lifecycle 객체가 추적한 컴포넌트의 현재 상태.  

<img src="https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg?hl=ko"/>

### - 안드로이드 액티비티 라이프사이클을 구성하는 상태 및 이벤트

+ `DefaultLifecycleObserver`를 구현해서 라이프사이클 메서드를 재정의해 컴포넌트의 라이프사이클 상태를 모니터링할 수 있음.

  ```ko
  class MyObserver : DefaultLifecycleObserver {
      override fun onResume(owner: LifecycleOwner) {
          connect()
      }
  
      override fun onPause(owner: LifecycleOwner) {
          disconnect()
      }
  }
  //만든 라이프사이클 옵저버를 전달.
  myLifecycleOwner.getLifecycle().addObserver(MyObserver())
  //myLifecycleOwner 객체는 LifecycleOwner인터페이스의 구현임.
  ```

## `LifecycleOwner`

+ LifecycleOwner는 클래스에 Lifecycle이 있음을 나타내는 단일 메서드: getLifecycle()를 갖는 인터페이스.
+ `Note :` 전체 어플리케이션 프로세스의 라이프사이클을 관리하려는 경우 `ProcessLifecycleOwner` 참고.
+ 구현으로서 Lifecycle 소유권을 추출하고 함께 작동하는 컴포넌트를 작성할 수 있게 만듬. 

+ Custom LifecyclerOwner 구현은 LifecycleRegistry를 사용해 이벤트 전달해야함.

  ```kotlin
  //커스텀 라이프사이클오너 구현 예 :
  class MyActivity : Activity(), LifecycleOwner {
  
      private lateinit var lifecycleRegistry: LifecycleRegistry
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
  
          lifecycleRegistry = LifecycleRegistry(this)
          lifecycleRegistry.markState(Lifecycle.State.CREATED)
      }
  
      public override fun onStart() {
          super.onStart()
          lifecycleRegistry.markState(Lifecycle.State.STARTED)
      }
  
      override fun getLifecycle(): Lifecycle {
          return lifecycleRegistry
      }
  }
  ```

  



### 중지 이벤트 처리

+ Lifecycle이 AppCompatActivity 또는 Fragment에 속하면 Lifecycle 상태가 CREATE로 변경되고, AppCompatActivity 또는 Fragment의 onSaveInstanceState()가 호출되면 ON_STOP 이벤트가 전달됨.
+ onSaveInstanceState()를 통해 상태가 저장되면 ON_START가 호출될 떄까지 UI는 변경할 수 없는 것으로 간주됨.
+ LiveData는 옵저버의 관련 Lifecycle이 적어도 STARTED 상태가 아니라면 옵저버를 호춯하지 않게해 저장된 상태와의 불일치되는 엣지 케이스를 방지함.
+ AppCompatActivity의 onStop() 메서드는 onSaveInstanceState() 이후에 호출되어 UI 상태변경이 허용되지 않지만 Lifecycle이 아직 CREATED 상태로 전환되지 않는다는 차이가 생김.
  이 문제를 방지하기 위해 `beta2` 버전 이하의 Lifecycle 클래스는 이벤트를 전달하지 않아도 상태를 CREATE로 표시 -> 시스템에서 onStop()을 호출할 때까지 이벤트가 전달되지 않더라도 현재 상태를 확인하는 모든 코드가 실제 값을 받도록 함.
  `⚠️문제 :` API 레벨 23 이하에서 Android 시스템은 다른 액티비티로 인해  액티비티가 일부 가려진 경우에도 액티비티 상태를 저장함( : onSaveInstanceState()호출하지만 onStop()은 호출 X).
  -> UI상태 수정 불가능한 경우에도 옵저버는 라이프사이클이 계속 활성상태라고 생각하는 경우가 발생할 수 있다.
+ LiveData 클래스에 유사한 동작을 노출하려는 모든 클래스는 위 beta2 이하에서 제공되는 해결방법을 구현해야함.
  `note : ` 1.0..0-rc1 버전부터 Lifecycle 객체는 CREATED로 표시되고 onSaveInstanceState()가 호출되면 onStop() 메서드 호출을 기다리지 않고 ON_STOP이 전달됨. ⚠️주의 : API 레벨 26 이하에서 Activity 클래스의 호출 순서와 일치하지 않으므로 주의 필요.