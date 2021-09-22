<h1>LiveData</h1>

+ The observable data holder class



<h3>Get liveData</h3>

`````ko

@Dao
interface TodoDao {
    @Query("SELECT * FROM Todo")
    fun getAll(): LiveData<List<Todo>>
...
}
`````

 



<h3>Observe data & action</h3>

`````ko
db.todoDao.getAll().observe(this, Observer{
	todos->result_text.text = todos.toString()
})

`````

