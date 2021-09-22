<h1>Room persitence library</h1>

+ The object-mapping library



코드 라인에서 Room을 입력한뒤 add dependency를 통해 간단히 dependency를 추가할 수 있다.



<h3>환경 설정</h3>

```kotlin

    def room_version = "2.3.0"
	implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor "androidx.room:room-compiler:$room_version"

// 위 코드가 추가되는데, Kotlin를 사용하기 위해서는 
    kapt("androidx.room:room-compiler:$room_version")
// 이 코드가 추가되어야 한다(Kotlin annotation processing tool.kapt).


plugins{
 	id 'kotlin-kapt'   
}

//위 플러그인이 추가되어야 함.
```





<h3>Entity</h3>

`````ko
@Entity
data class Todo(var title : String){
    @PrimaryKey(autoGenerate = true) var id : Int = 0
}
`````





<h3>Dao</h3>

`````ko
@Dao
interface TodoDao {
    @Query("SELECT * FROM Todo")
    fun getAll(): LiveData<List<Todo>>

    @Insert
    fun insert(todo: Todo)

    @Update
    fun update(todo: Todo)

    @Delete
    fun delete(todo: Todo)
}
`````





<h3>Database</h3>

`````ko
@Database(entities = [Todo::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun todoDao(): TodoDao
}
`````



<h3>Get Db instance</h3>

`````ko
val db = Room.databaseBuilder(
        applicationContext,
        AppDatabase::class.java, "todo-db"
    ).allowMainThreadQueries().build()
    
    
//{.allowMainThreadQueries()}는 학습을 위해 임시로 넣어 놓은것.
//이를 MainThread가 아닌 쓰레드에서 비동기 처리해야 한다.

`````

