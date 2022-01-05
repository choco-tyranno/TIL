# Saved State module for ViewModel

+ 시스템-초기화 프로세스 중단을 처리해야하는 경우 onSaveInstanceState()를 백업으로 사용.
+ 일반적으로 UI상태는 ViewModel에 저장 및 참조되기 때문에 onSaveInstanceState()를 사용에는 `Saved State Module` 이 요구됨.
+ ViewModel의 생성자에 SavedStateHandle 객체가 필요.
+ SavedStateHandle은 key-value map. 개체를 쓰거나 받는 것을 지원.
+ SavedStateHandle에 저장된 value 객체는 시스템-초기화 프로세스 중단 후에도 동일한 객체로 계속 사용할 수 있음.
+ `⚠️Caution : ` 저장되는 State는 단순하고 가벼워야함.

````kotlin
//추가 구성 없이 생성자에 추가해주면 됨. ViewModel팩토리가 적절한 SavedStateHandle 제공.
class SavedStateViewModel(private val state: SavedStateHandle) : ViewModel() { ... }
//커스텀 ViewModelProvider.Factory 인스턴스 제공의 경우 AbstractSavedStateViewModelFactory를 extends해서 설정 가능.
//낮은 버전의 프래그먼트 라이브러리 사용경우, 종속항목 추가 후 SavedStateViewModelFactory 사용.
````

## SavedStateHandle을 사용한 작업

+ SavedStateHandle은 set()/get() 메서드로 저장된 상태에 데이터 작성 및 저장된 상태에서 데이터를 검색할 수 있게 하는 키-값 맵.
+ getLiveData()를 사용해 LiveData에 wrapping된 SavedStateHandle에서 값을 검색할 수 있음.
+ 키의 값이 업데이트되면 LiveData가 새 값을 수신. 보통 데이터 목록을 필터링하는 쿼리를 입력하는 등 사용자 인터렉션으로 인해 값이 설정됨.
  이러한 업데이트된 값은 [transform LiveData](https://developer.android.com/topic/libraries/architecture/livedata#transform_livedata)으로 사용될 수 있음.

```kotlin
class SavedStateViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
    val filteredData: LiveData<List<String>> =
        savedStateHandle.getLiveData<String>("query").switchMap { query ->
        repository.getFilteredData(query)
    }

    fun setQuery(query: String) {
        savedStateHandle["query"] = query
    }
}
```

+ SavedStateHandle을 사용하면 프로세스 중단 시나리오에 UI컨트롤러에서 값을 수동으로 저장 및 복원해 ViewModel에 다시 전달할 필요가 없어짐. 

> 구현된 메소드 :
>
> + contains(key : String)
> + remove(key : String)
> + keys()

+ SavedStateHandle 내에 저장된 데이터는 UI 컨트롤러의 나머지 savedInstanceState와 함께 Bundle로 저장되고 복원됨.
+ 기본적으로 다음과 같은 Bundle 타입과 동일한 데이터 타입에 대해 SavedStateHandle의 set()/get()를 호출할 수 있음.

|                           |                   |
| ------------------------- | ----------------- |
| **Type/Class support**    | **Array support** |
| `double`                  | `double[]`        |
| `int`                     | `int[]`           |
| `long`                    | `long[]`          |
| `String`                  | `String[]`        |
| `byte`                    | `byte[]`          |
| `char`                    | `char[]`          |
| `CharSequence`            | `CharSequence[]`  |
| `float`                   | `float[]`         |
| `Parcelable`              | `Parcelable[]`    |
| `Serializable`            | `Serializable[]`  |
| `short`                   | `short[]`         |
| `SparseArray`             |                   |
| `Binder`                  |                   |
| `Bundle`                  |                   |
| `ArrayList`               |                   |
| `Size (only in API 21+)`  |                   |
| `SizeF (only in API 21+)` |                   |

+ 만일 저장하고자 하는 클래스가 위 목록에 있는 클래스를 확장하지 않는 경우, `@Parcelize` Kotlin 주석을 추가하거나 `Parcelable`을 직접 구현할 수 있음.

## Non-parcelable 클래스 저장

+ 위 구현이 없는 경우에도 인스턴스를 SavedStateHandle에 직접 저장 가능.
+ Lifecycle 2.3.0-alpha03부터 SavedStateHandle은 setSavedStateProvider() 메서드로 객체를 Bundle로 저장하는 자체 로직을 제공.
+ SavedStateRegistry.SavedStateProvider는 저장할 상태가 포함된 Bundle을 반환하는 단일 `saveState()`메서드를 정의하는 인터페이스.
+ SavedStateHandle은 상태를 저장할 준비가 되면 saveState()를 호출해 SavedStateProvider에서 Bundle을 검색해 연결된 key용 Bundle을 저장함.

```kotlin
//임시 파일을 만드는 로직을 캡슐화한 사례 :
private fun File.saveTempFile() = bundleOf("path", absolutePath)

private fun Bundle.restoreTempFile() = if (containsKey("path")) {
    File(getString("path"))
} else {
    null
}

class TempFileViewModel(savedStateHandle: SavedStateHandle) : ViewModel() {
    private var tempFile: File? = null
    init {
        val tempFileBundle = savedStateHandle.get<Bundle>("temp_file")
        if (tempFileBundle != null) {
            tempFile = tempFileBundle.restoreTempFile()
        }
        savedStateHandle.setSavedStateProvider("temp_file") { // saveState()
            if (tempFile != null) {
                tempFile.saveTempFile()
            } else {
                Bundle()
            }
        }
    }

    fun createOrGetTempFile(): File {
      return tempFile ?: File.createTempFile("temp", null).also {
          tempFile = it
      }
    }
}
```

