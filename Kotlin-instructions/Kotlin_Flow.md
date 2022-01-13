# Kotlin - Flow

[관련문서 : Flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/)

+ 코루틴에서 Flow는 여러 값을 순차적으로 내보낼(emit) 수 있는 타입.
  예 : 데이터베이스로부터의 업데이트를 수신하는데 flow 사용 가능.
+ 코루틴을 기반으로 빌드됨.
+ 비동기식으로 계산될 수 있는 스트림 데이터 개념.
+ 내보내는 값들은 모두 동일 유형이어야 함.
+ 값 시퀀스를 생성하는 Iterator와 비슷하지만, flow는 suspend 함수를 사용해 값을 비동기적으로 생성하고 사용함.
  -> 네트워크 요청을 main-safe하도록 만들 수 있음.

<img src="https://developer.android.com/images/kotlin/flow/flow-entities.png"/>

+ Producer(생산자)는 스트림에 추가되는 데이터를 생산.
  flow 코루틴으로 비동기적 데이터 생산 가능.
+  (Optional) Intermediary(중개자)는 스트림에서 내보내는 각 값이나 스트림 자체를 수정할 수 있음.
+ Consumer(소비자)는 스트림의 값을 사용.

> 안드로이드에선..
>
> + 보통 Repository가 UI데이터 생산자에 해당.
> + UI는 최종적으로 데이터를 표시하는 소비자에 해당.
> + UI레이어가 사용자 입력 이벤트의 생산자이고, 계층 구조의 다른 레이어가 이 이벤트의 사용자이기도 함.
> + 생산자와 소비자 사이의 중개자는 보통 다음 레이어의 요구사항에 맞게 조정하는 위해 데이터 스트림을 수정하는 역할을 함.

### Flow 만들기

+ flow builder API를 사용해 flow 만듬.
+ 빌더는 emit 함수를 사용해 수동으로 새 값을 데이터 스트림에 내보낼 수 있는 새 flow를 만듬.
+  flow 빌더는 코루틴 내에서 실행됨. 
  -> 비동기 API 이점 활용 가능.
+ ⚠️제한사항 : 
  1. flow들은 순차적임. suspend 함수 호출 후 반환될 때까지 producer는 정지 상태로 유지됨. 
  2. flow 빌더에서는 Producer가 다른 CoroutineContext의 값을 emit할 수 없음.
     -> 새 코루틴을 만들거나 withContext블록을 사용해 다른 CoroutineContext에서 emit 호출하지 말것.
     (이런 경우 callbackFlow같은 다른 flow 빌더 사용 가능)

### 스트림 수정

+ 중개자는 `중간 연산자`를 사용해 값을 소비하지 않고도 데이터 스트림 수정 가능.
+ 중간 연산자는 데이터 스트림에 적용되는 경우 작업 체인을 설정하는 함수.
  값이 향후 사용될 때까지 실행되지 않음(lazy operations). 

```kotlin
//중개연산자 map을 사용해 데이터를 변형한 예시 :
class NewsRepository(
    private val newsRemoteDataSource: NewsRemoteDataSource,
    private val userData: UserData
) {
    /**
     * Returns the favorite latest news applying transformations on the flow.
     * These operations are lazy and don't trigger the flow. They just transform
     * the current value emitted by the flow at that point in time.
     */
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            // Intermediate operation to filter the list of favorite topics
            .map { news -> news.filter { userData.isFavoriteTopic(it) } }
            // Intermediate operation to save the latest news in the cache
            .onEach { news -> saveInCache(news) }
}
```



### Flow에서 Collecting

+ `터미널 연산자`를 사용해 값 수신 대기를 시작하는 flow를 트리거.
+ 내보낼때 스트림의 모든 값을 가져오려면 `collect`를 사용.
  collect는 suspend 함수. 모든 새 값에서 호출되는 매개변수로 람다 사용.

```kotlin
//터미널 연산자 collect를 사용해 repository 레이어로부터의 데이터를 ViewModel에서 소비하는 예시 :
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    init {
        viewModelScope.launch {
            // Trigger the flow and consume its elements using collect
            newsRepository.favoriteLatestNews.collect { favoriteNews ->
                // Update View with the latest favorite news
            }
        }
    }
}
//여기서 flow를 collect함으로써 일정 주기로 네트워크 요청한 결과를 내보내고 최신 뉴스를 새로고침하는 producer를 트리거함.
//producer는 while(true) loop로 인해 항상 활성화 상태에 있고 ViewModelScope가 취소에 의해 close될 수 있음.
```

+ Flow collection은 다음 사유로 정지될 수 있음.
  1. 수집하는 코루틴이 취소됨. -> 아래 있는 producer도 정지됨.
  2. producer의 item emit을 마침. -> 위 예시에선 data 스트림이 닫히고 collect를 호출한 코루틴이 재개됨.
+ Flow 는 cold / lazy함.
  -> 터미널 연산자가 호출되지 않는한 producer 코드가 실행되지 않음.
  따라서 flow collecting이 여러 개가 있으면, 각 호출에 따라 데이터 소스를 서로 다른 주기로 최신 뉴스를 여러 번 가져옴.
  여러 Consumer들이 동시에 collect하는 경우 flow를 공유하고 최적화하려면 `shareIn` 연산자를 사용할 것. 

### 예상치못한 예외 포착하기

+ Producer 구현은 타사 라이브러리로부터 제공될 수 있기 때문에 언제든 예상치못한 예외가 발생할 수 있음.
+ `catch` 중간 연산자로 예외를 다룰 수 있음.

```kotlin
//catch 중간연산자로 예외 처리한 예시 :
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    init {
        viewModelScope.launch {
            newsRepository.favoriteLatestNews
                // Intermediate catch operator. If an exception is thrown,
                // catch and update the UI
                .catch { exception -> notifyError(exception) }
                .collect { favoriteNews ->
                    // Update View with the latest favorite news
                }
        }
    }
}
//예외발생시 collect람다가 호출되지 않음.
```

```kotlin
//catch에서도 flow에 item emit 가능. -> repository레이어에 캐시된 값을 대신 내보내는 예시 :
class NewsRepository(...) {
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            .map { news -> news.filter { userData.isFavoriteTopic(it) } }
            .onEach { news -> saveInCache(news) }
            // If an error happens, emit the last cached values
            .catch { exception -> emit(lastCachedNews()) }
}
//이 예의 경우 예외발생에도 emit처리하기 때문에 collect가 호출됨.
```

### 다른 CoroutineContext에서 실행

+ 기본적으로 flow빌더의 producer는 코루틴의 코루틴컨텍스트에서 실행되고,
  다른 코루틴컨텍스트에서 값을 내보낼 수 없음.
+ 하지만 repository레이어가 Dispatcher.Main에서 동작해선 안되는 것과 같은 바람직하지 않은 경우도 있음.
+ `flowOn` 중간연산자를 사용해  upstream flow의 코루틴컨텍스트를 바꿀 수 있음.
+ flowOn 연산자가 적용되는 upstream flow는 flowOn 앞에 붙는 중간연산자 및 producer에 해당.
+ flowOn 뒤에 붙는 중간연산자나 consumer는 downStream flow에 해당. flowOn에 영향을 받지 않음.
+ flowOn 연산자가 여러 개 있는 경우 현재 위치에서 각자가 upstream을 변경함.

```kotlin
//flowOn으로 upstream의 CoroutineContext 변경 예시 :
class NewsRepository(
    private val newsRemoteDataSource: NewsRemoteDataSource,
    private val userData: UserData,
    private val defaultDispatcher: CoroutineDispatcher
) {
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            .map { news -> // Executes on the default dispatcher
                news.filter { userData.isFavoriteTopic(it) }
            }
            .onEach { news -> // Executes on the default dispatcher
                saveInCache(news)
            }
            // flowOn affects the upstream flow ↑
            .flowOn(defaultDispatcher)
            // the downstream flow ↓ is not affected
            .catch { exception -> // Executes in the consumer's context
                emit(lastCachedNews())
            }
}
```

```kotlin
//I/O작업에 최적화된 dispatcher 사용.
class NewsRemoteDataSource(
    ...,
    private val ioDispatcher: CoroutineDispatcher
) {
    val latestNews: Flow<List<ArticleHeadline>> = flow {
        // Executes on the IO dispatcher
        ...
    }
        .flowOn(ioDispatcher)
}
```

### Jetpack 라이브러리에서의 Flow

+ Flow는 많은 Jetpack 라이브러리들과 통합되지만, 특히 live data 업데이트와 데이터 무제한 스트림에 잘맞음.
+ Room과 함께 Flow를 사용하면 database의 변경에 대한 알림을 받을 수 있음.
  DAO 정의에서 반환타입을 Flow타입으로만 정의하면 됨.

````kotlin
// Example 테이블에서 변화가 생길 때 마다 데이터베이스의 새로운 아이템들과 함께 새로운 리스트가 emit됨. 
@Dao
abstract class ExampleDao {
    @Query("SELECT * FROM Example")
    abstract fun getExamples(): Flow<List<Example>>
}
````

### 콜백기반 API를 Flow로 변환

```kotlin
//콜백기반API 중 하나인 Firebase Firestore와 flow를 사용한 예시:
class FirestoreUserEventsDataSource(
    private val firestore: FirebaseFirestore
) {
    // Method to get user events from the Firestore database
    fun getUserEvents(): Flow<UserEvents> = callbackFlow {

        // Reference to use in Firestore
        var eventsCollection: CollectionReference? = null
        try {
            eventsCollection = FirebaseFirestore.getInstance()
                .collection("collection")
                .document("app")
        } catch (e: Throwable) {
            // If Firebase cannot be initialized, close the stream of data
            // flow consumers will stop collecting and the coroutine will resume
            close(e)
        }

        // Registers callback to firestore, which will be called on new events
        val subscription = eventsCollection?.addSnapshotListener { snapshot, _ ->
            if (snapshot == null) { return@addSnapshotListener }
            // Sends events to the flow! Consumers will get the new events
            try {
                offer(snapshot.getEvents())
            } catch (e: Throwable) {
                // Event couldn't be sent to the flow
            }
        }

        // The callback inside awaitClose will be executed when the flow is
        // either closed or cancelled.
        // In this case, remove the callback from Firestore
        awaitClose { subscription?.remove() }
    }
}
```

+ flow빌더와 달리 callbackFlow는 `send` 함수를 사용한 다른 CoroutineContext에서의 emit,
   또는 `offer` 함수를 사용한 코루틴 외부에서의 emit을 가능하게 함.
+ callbackFlow는 내부적으로 blocking queue와 개념이 매우 유사한 `channel`을 사용함.
+ Channel은 버퍼링될 수 있는 요소들의 최대 수 용량으로 config됨.
  callbackFlow에서 생성된 channel의 기본 용량은 64개.
+ channel의 공간이 가득찼을 때 새 element를 추가하려는 경우, send는 새 요소를 위한 공간이 생길 때까지 producer를 일시중단 시키지만, offer는 channel에 element를 추가하지 않고 즉시 false를 반환함.

### StateFlow

+ StateFlow는 현재 및 새 상태 업데이트를 collector들로 내보내는 state-holder observalbe flow.
+ 현재 상태 값은 value 속성으로 읽을 수도 있음.
+ 상태를 업데이트하고 flow로 보내려면 MutableStateFlow 클래스의 value 프로퍼티에 새 값을 할당하면 됨.
+ 안드로이드에서 StateFlow는 observable하고 mutable한 상태를 유지해야하는 클래스에 적합.

```kotlin
//stateFlow 예시 :
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    // Backing property to avoid state updates from other classes
    private val _uiState = MutableStateFlow(LatestNewsUiState.Success(emptyList()))
    // The UI collects from this StateFlow to get its state updates
    val uiState: StateFlow<LatestNewsUiState> = _uiState

    init {
        viewModelScope.launch {
            newsRepository.favoriteLatestNews
                // Update View with the latest favorite news
                // Writes to the value property of MutableStateFlow,
                // adding a new element to the flow and updating all
                // of its collectors
                .collect { favoriteNews ->
                    _uiState.value = LatestNewsUiState.Success(favoriteNews)
                }
        }
    }
}

// Represents different states for the LatestNews screen
sealed class LatestNewsUiState {
    data class Success(news: List<ArticleHeadline>): LatestNewsUiState()
    data class Error(exception: Throwable): LatestNewsUiState()
}
```

+ flow builder를 사용해 빌드된 cold flow와 달리 stateFlow는 hot flow.
  -> flow를 collect하면 어떠한 producer 코드도 trigger하지 않음. 
+ StateFlow는 항상 활성상태에 있고, 메모리에 있으며, 가비지 콜렉션 루트에서 이것에 대한 그 어떤 참조도 없는 경우에만 가비지 콜렉션 대상이 됨. 
+ 새로운 Consumer가 Flow를 Collect하기 시작하면 스트림의 마지막 상태와 모든 후속 상태들을 받음.
  (LiveData와 같은 Observable 클래스들에서 이러한 동작을 찾을 수 있음)

```kotlin
//View가 StateFlow를 수신대기함(다른 flow들 처럼).
//액티비티의 라이프사이클 상태에 따라 collecting을 제어.
class LatestNewsActivity : AppCompatActivity() {
    private val latestNewsViewModel = // getViewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        // Start a coroutine in the lifecycle scope
        lifecycleScope.launch {
            // repeatOnLifecycle launches the block in a new coroutine every time the
            // lifecycle is in the STARTED state (or above) and cancels it when it's STOPPED.
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                // Trigger the flow and start listening for values.
                // Note that this happens when lifecycle is STARTED and stops
                // collecting when the lifecycle is STOPPED
                latestNewsViewModel.uiState.collect { uiState ->
                    // New value received
                    when (uiState) {
                        is LatestNewsUiState.Success -> showFavoriteNews(uiState.news)
                        is LatestNewsUiState.Error -> showError(uiState.exception)
                    }
                }
            }
        }
    }
}
```

+ ⚠️Caution : UI에서 launch나 launchIn extension function을 통해 직접적으로 Flow를 collect하지 말것.
  ->View가 보이지 않는 상황에서도 이벤트를 처리하게됨. -> App Crash.
  -> 방지를 위해, 위 사례에서와 같이 repeatOnLifecycle [API](`androidx.lifecycle:lifecycle-runtime-ktx:2.4.0-alpha01` library and higher)를 사용할 것.

+ 어떠한 Flow를 StateFlow로 변환하려면, `stateIn` 중간연산자를 사용.

### StateFlow와 Flow 그리고 LiveData

+ StateFlow는 LiveData와 같이 Observable한 data holder 클래스이며, 앱 아키텍처에서 사용될 때 비슷한 패턴을 따름.
+ StateFlow와 LiveData의 차이점 :
  + StateFlow는 LiveData와 달리 초기 상태가 생성자에 전달되어야 함.
  + LiveData.observe() 함수는 View가 STOPPED 상태가 되면 자동으로 Consumer를 unregister하지만, 
    StateFlow 또는 다른 Flow들에서 Collecting하는 것들은 자동으로 중지되지 않음.
    -> Lifecycle.repeatOnLifecycle 블록에서 Flow를 Collect하면 됨.

### `shareIn`을 사용해 Cold Flow를 Hot Flow로 변환

+ `shareIn`연산자를 통해 Cold Flow를 Hot Flow로 바꿀 수 있음.
+ StateFlow는 Hot Flow임(Flow가 collect되는 동안, 또는 가비지콜렉션 루트에 이에 대한 참조가 남아있는 동안 메모리에 남아있음).
+ shareIn 연산자를 사용하면 각 collector에서 새로운 Flow를 만드는 것을 대신하여, 받은 데이터를 각 Collector들에게 공유할 수 있음.
  조건 : 
  + Flow를 공유하기 위해서 사용되는 것은 CoroutineScope임.
    -> 이 Scope가 어떠한 Consumer보다 더 길게 수명을 지속해야 함. 
  + replay에 할당되는 값은 각 Collector에 재전달되는 아이템의 수량임.
  + started에 할당되는 값은 시작시 동작하는 방식(정책)임.

````kotlin
//shareIn 중간연산자 예시 :
class NewsRemoteDataSource(...,
    private val externalScope: CoroutineScope,
) {
    val latestNews: Flow<List<ArticleHeadline>> = flow {
        ...
    }.shareIn(
        externalScope,
        replay = 1,
        started = SharingStarted.WhileSubscribed()
    )
}
//latestNews Flow는 마지막으로 내보낸 1개의 항목을 새로운 Collector에 다시 내보내고(replay),
//externalScope가 살아있는 동안, 그리고 collector들이 활성상태에 있는 동안 활성상태를 유지함.
//SharingStarted.WhileSubscribed() 시작 정책은 Subscriber들이 활성화 상태에 있는 동안 Upstream Producer를 활성상태로 유지함.
//SharingStarted.Eagerly는 Producer를 즉시 시작.
//ShareingStarted.Lazily는 첫 Subscriber가 나타난 후 공유를 시작하고 활성상태를 영원히 유지함.
````

### SharedFlow 

+ SharedFlow 클래스를 직접만들 수 있음.
  StateFlow와 마찬가지로 MutableSharedFlow 타입의 프로퍼티를 사용해 아이템을 Flow에 보낼 수 있음.
+ `shareIn` 함수는 SharedFlow를 반환함. Hot Flow로, 이를 collect하는 모든 Consumer에게 값을 내보냄.
+ SharedFlow는 StateFlow를 high-config 가능한 일반화임.

```kotlin
// SharedFlow를 사용하여 앱의 나머지 부분으로 tick을 보내 모든 콘텐츠가 동시에 주기적으로 리프레시되도록 하는 예시 :
// TickHandler는 SharedFlow를 노출하여 다른 클래스가 콘텐츠를 새로고침할 타이밍을 알 수 있도록 함.
// Class that centralizes when the content of the app needs to be refreshed
class TickHandler(
    private val externalScope: CoroutineScope,
    private val tickIntervalMs: Long = 5000
) {
    // Backing property to avoid flow emissions from other classes
    private val _tickFlow = MutableSharedFlow<Unit>(replay = 0)
    val tickFlow: SharedFlow<Event<String>> = _tickFlow

    init {
        externalScope.launch {
            while(true) {
                _tickFlow.emit(Unit)
                delay(tickIntervalMs)
            }
        }
    }
}

class NewsRepository(
    ...,
    private val tickHandler: TickHandler,
    private val externalScope: CoroutineScope
) {
    init {
        externalScope.launch {
            // Listen for tick updates
            tickHandler.tickFlow.collect {
                refreshLatestNews()
            }
        }
    }

    suspend fun refreshLatestNews() { ... }
    ...
}
```

+ SharedFlow의 동작을 커스텀할 수 있음 : 
  + replay는 그 값의 수만큼 이전에 emit한 값들을 다시 내보낼 수 있음.
  + onBufferOverflow를 사용하면 버퍼가 꽉찬 경우의 정책을 지정할 수 있음.
    기본값은 BufferOverflow.SUSPEND(호출자 일시중단)임.
    다른 옵션으로는 `DROP_LATEST `또는 `DROP_OLDEST`가 있음.
  + MutableSharedFlow의 프로퍼티 subscriptionCount는 활성 collector 수를 지정해 비즈니스 로직 최적화도 가능.
  + resetReplayCache는 Flow에 전송된 최신 정보를 다시 내보내지 않을 때 사용.







