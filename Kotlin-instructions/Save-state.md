<h1>Save state</h1>

+ 예기치 못한 시스템 종료에 대비한 상태 저장방법들.
+ 아래 방법들은 안드로이드 시스템에 데이터를 저장해 둠으로써 다시 불러올 수 있는 것이기 때문에 간단하고 가벼운 데이터만 저장해야 한다.



<h3> 1. onSaveInstanceState() & onRestoreInstanceState()</h3>

`````ko
class MainActivity : AppCompatActivity(){
...
	override fun onSaveInstanceState(outState : Bundle){
		super.onSaveInstanceState(outState)
		outState.putInt("count", viewModel.count)
	}
	
	override fun onRestoreInstanceState(savedInstanceState: Bundle){
		super.onRestoreInstanceState(savedInstanceState)
		viewModel.count = savedInstanceState.getInt("count")
	}
}

//onSaveInstanceState()에서 인수로 전달되는 outState 번들에 키와 함께 value를 저장.
//onRestoreInstanceState()를 통해 저장된 것으로 데이터를 복구.
`````



<h3>2. SaveStateHandle</h3>

+ Saved State Module for ViewModel

`````ko
class MainViewModel(private val handle : SavedStateHandle) : ViewModel(){
	private var count = handle.get<Int>("count") ? : 0
		set(value){
			field = value
			//value 값 확정시킴.
			countLiveData.value = value
			handle.set("count", value)
			//setter를 override하면서 동시에 수행될 코드들 삽입.
		}
    val countLiveData : MutableLiveData<Int> = MutableLiveData<Int>()
    ...
}

//ViewModel의 경우 생성자 인자로 (private val handle : SavedStateHandle)만 널어주기만 하면 된다.
`````





<h3>