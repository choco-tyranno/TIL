<h1>Coroutine scope</h1>

+ The built-in coroutine scopes.
+ It contained in each KTX extentions(e.g. For ViewModelScope, use ...viewModel-ktx dependency).

<h3>Usage</h3>

`````ko
    class MainViewModel(...):AnroidViewModel(...){
    ...
    	fun insert(todo : String) {
        	viewModelScope.launch(Dispatchers.IO){
            	db.todoDao().insert(Todo(todo))
        	}
    	}
    }
    
//여기서 dao의 insert()는 suspend 되어있어 비동기 처리되도록 함.
`````