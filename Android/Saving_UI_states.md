# Saving UI states

+ 액티비티의 UI상태를 적시에 보존하고 복원하는 것은 UX의 중요한 부분.
  UI 컨트롤러가 보유하는 데이터는 시스템에 의해 파괴될 위험성이 항상 존재함. 
+  사용자의 기대와 시스템 동작 간 간극을 메워 어플 및 액티비티 간 전환에서 UI상태를 유지하기 위해, 일시적 상태보존 솔루션 (ViewModel과 onSaveInstance() )과 영구적 상태보존 솔루션(로컬 저장소)의 장단점에 따른 적절한 조합을 이용해야 함.
  `note : ` 조합은 위해선 UI데이터의 복잡성, 앱 사용 케이스, 검색 속도와 메모리 사용 고려 사항 등에 따라 결정.

### <사용자에 의한 UI 상태 해제>와  <시스템에 의한 UI 상태 해제>

### 사용자에 의한 UI 상태 해제 : 

+ 사용자가 액티비티를 완전히 닫을 수 있는 방법 :
  + 뒤로가기 버튼 누름
  + 오버뷰(최근) 화면에서 액티비티를 스와이프로 제거
  + 액티비티에서 위로 탐색(네비게이트 업)
  + 설정에서 직접 앱 종료
  + Activity.finish()가 호출되는 작업 시
+ 액티비티가 완전히 닫히는 경우, 액티비티 인스턴스와 이와 함께 저장된 모든 상태 및 saved instance state record가 메모리에서 제거됨.

### 시스템에 의한 UI상태 해제 : 

+ 스크린 회전 또는 다중 창 모드 전환 등 `구성(configuration)변경` 발생 시.
+ 앱이 백그라운드 상태가 되는 경우 시스템은 앱 프로세스를 파괴할 수 있는 가능성이 생김.

### UI상태 유지를 위한 옵션

|                                                             | ViewModel                                                | Saved instance state                                 | Persistent storage                                           |
| :---------------------------------------------------------- | :------------------------------------------------------- | :--------------------------------------------------- | ------------------------------------------------------------ |
| 저장 위치                                                   | in memory                                                | 디스크에 직렬화                                      | 디스크 또는 네트워크                                         |
| configuration 변경에도 남아 있는지                          | ㅇ                                                       | ㅇ                                                   | ㅇ                                                           |
| 시스템 초기화 프로세스 죽음에도 남아있는지                  | ㄴ                                                       | ㅇ                                                   | ㅇ                                                           |
| 사용자의 액티비티 해제에도 남아 있는지 dismissal/onFinish() | ㄴ                                                       | ㄴ                                                   | ㅇ                                                           |
| 데이터 제한                                                 | 복잡한 개체 Ok. 공간은 사용 가능한 메모리에 의해 제한됨. | primitive type 및 String과 같은 단순하고 작은 객체만 | 디스크 공간 또는 네트워크 리소스에서 검색하는 비용/시간에 의해서 제한됨. |
| 읽기/쓰기 시간                                              | 빠름(메모리 액세스)                                      | 느림(직렬화/역직렬화 및 디스크 액세스 필요 때문)     | 느림(디스크 액세스 또는 네트워크 트랜잭션 필요 때문)         |



## ViewModel로 Configuration 변경 처리

+ ViewModel에 캐시된 데이터로 Config 변경에 따른 네트워크 및 디스크로부터의 데이터 로드를 다시 수행하는 것을 방지 가능.
+ saved instance state와 달리 ViewModel은 lifecycle owner가 제거되면 지워짐.

## onSaveInstanceState 사용 (시스템 초기화 프로세스의 제거에 대응하기 위한 백업 등)

+ `onSaveInstanceState()` 콜백에 저장되는 데이터는 시스템이 UI컨트롤러를 파괴하고 다시 재생성하는 경우 UI컨트롤러의 상태를 reload하는 데 필요.
+ Saved instance state의 bundle은 디스크에 직렬화되기 때문에 config 변경과 프로세스 죽음에도 유지됨. 스토리지와 속도에 의해 제한됨.
  직렬화는 개체가 복잡하면 메모리를 많이 소모 -> 이 프로세스는 config 변경 중에 기본 스레드에서 발생하기 때문에 장기실행 직렬화로 인한 프레임 손실 및 시각적 끊김 발생 가능.
+ UI를 이전 상태로 복원하는데 필요한 데이터를 불어오는데 필요한 id와 같은 최소한의 데이터 저장에 사용하는 것을 권장.
+ 시스템 초기화 프로세스의 죽음에 대처하도록 onSaveInstanceState() 구현을 권장.
+ Intent로 액티비티가 전환되는 경우 시스템이 액티비티를 복원할 때 bundle of extras가 액티비티에 전달되기 때문에 onSaveInstanceState가 필요없는 경우도 있음.
+ [SavedStateModule for ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel-savedstate#savedstatehandle)을 사용해 ViewModel에 저장된 상태에 대한 액세스 제공 가능.
  -> UI컨트롤러의 onSaveInstanceStated로부터 다시 ViewModel로 저장된 상태를 전달할 필요가 없어짐. 

## SavedStateRegistry를 사용해 저장된 상태로 연결

+ UI컨트롤러(Activity 1.0.0, Fragment 1.1.0 부터)는 SavedStateRegistryOwner가 구현되어있으며, UI컨트롤러에 바인드된 SavedStateRegistry를 제공함.
+ SavedStateRegistry는 컴포넌트가 UI컨트롤러의 저장된 상태에 연결되는 것 및 기여될 수 있도록 해줌.
  (예 : `Saved State module for ViewModel`은 SavedStateRegistry를 사용해 SavedStateHandle을 생성하고 ViewModel 객체에 제공함. UI컨트롤러에서 getSavedStateRegisty() 호출을 통해 SavedStateRegistry를 얻을 수 있음).
+  저장된 상태에 기여하고자 하는 컴포넌트는 `SavedStateRegistry.SavedStateProvider` (단일 saveState()메서드를 정의하는)를 구현해야함.
+ saveState() 메서드는 컴포넌트가 저장할 상태를 담은 `Bundle`을 반환토록 함.
+ SavedStateRegistry는 UI컨트롤러 라이프사이클의 상태 저장 단계에서 이 메서드를 호출함.

```kotlin
//SavedStateRegistry.SavedStateProvider 구현체 예시.
class SearchManager : SavedStateRegistry.SavedStateProvider {
    companion object {
        private const val QUERY = "query"
    }

    private val query: String? = null

    ...

    override fun saveState(): Bundle {
        return bundleOf(QUERY to query)
    }
}
```

+ SavedStateProvider를 등록하려면 SavedStateRegistry의 registerSavedStateProvider()를 호출해 Provider자체와 그 데이터를 연결할 키를 함께 전달해야 함.
+  Provider의 저장된 데이터는 SavedStateRegistry의 consumeRestoredStateForKey(key : String) 호출로 검색 가능.
+ UI컨트롤러에서 super.onCreate()를 호출한 뒤에 onCreate() 블록에서 SavedStateProvider를 등록할 수 있음.
  note : 또는 SavedStateRegistryOwner에 LifecycleObserver 설정으로 LifecycleOwner를 구현하고 ON_CREATE 이벤트 발생 시 SavedStateProvider를 등록할 수도 있음.
  LifecycleObserver를 사용하면 SavedStateRegistryOwner로부터 이전에 저장된 상태의 등록 및 검색을 분리할 수 있음. 아래 예시 참고.

```kotlin
//LifecycleObserver설정 및 LifecycleOwner 구현으로 SavedStateProvider 구현 사례 :
class SearchManager(registryOwner: SavedStateRegistryOwner) : SavedStateRegistry.SavedStateProvider {
    companion object {
        private const val PROVIDER = "search_manager"
        private const val QUERY = "query"
    }

    private val query: String? = null

    init {
        // Register a LifecycleObserver for when the Lifecycle hits ON_CREATE
        registryOwner.lifecycle.addObserver(LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_CREATE) {
                val registry = registryOwner.savedStateRegistry

                // Register this object for future calls to saveState()
                registry.registerSavedStateProvider(PROVIDER, this)

                // Get the previously saved state and restore it
                val state = registry.consumeRestoredStateForKey(PROVIDER)

                // Apply the previously saved state
                query = state?.getString(QUERY)
            }
        }
    }

    override fun saveState(): Bundle {
        return bundleOf(QUERY to query)
    }

    ...
}

class SearchFragment : Fragment() {
    private var searchManager = SearchManager(this)
    ...
}
```

note : SavedStateRegistry와 onSaveInstanceState는 동일한 Bundle에 데이터를 저장함.

## 복잡하거나 큰 데이터는 Local Persistent 사용 권장 

+ 데이터베이스 또는 SharedPreference과 같은 영구 로컬 저장소는 사용자가 앱의 데이터를 삭제하지 않는 한 앱이 사용자 기기에 설치되어 있는 동안 유지됨.
+ 로컬 저장소를 사용하는 경우 데이터는 처음부터 메모리에 있지 않고 메모리로 읽어와야 하는 작업이 필요하기 때문에 검색하는 데 비용이 많이 들 수 있음.
+ ViewModel이나 SavedInstanceState는 장기 저장소 솔루션이 아님. 이러한 단기 저장 솔루션은 일시적인 UI상태를 임시 저장하는 데에만 사용할 것 권장.

## UI상태 관리하기 : 

+ Local Persistent : 액티비티의 라이프사이클과 무관한 영구 저장할 데이터 및 메타데이터가 포함된 객체 컬렉션 저장.
+ ViewModel : UI컨트롤러 표시에 필요한 모든 데이터 저장.
+ onSavedInstanceState : 시스템에서 UI컨트롤러 중지 후 다시 생성될 때 쉽게 상태를 다시 로드하는데 필요한 소량의 데이터(대게 id, 최근 검색어) 저장.
+ onSaveInstanceState() 번들에 저장된 액티비티 쿼리를 ViewModel에 전달. ViewModel은 캐시된 검색결과의 유무를 확인하고 없다면 검색하여 결과 로드.
  note : 빈 쿼리 전달로 로드할 데이터 없음을 ViewModel에 알릴 수 있음. 













































