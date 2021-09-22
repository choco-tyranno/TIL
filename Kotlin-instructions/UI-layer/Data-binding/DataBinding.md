<h1>DataBinding</h1>

+ Binding UI components in layouts to data sources.



<h3>Build environment</h3>

`````ko
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
`````



<h3>Usage</h3>

`````ko
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewmodel"
            type="com.choco_tyranno.masklocation.MainViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
...


`````

//xml의 최상위 레이아웃을 <layout> 태그로 감싸준다.
//xml name space부분(xmlns:...)가 layout 태그가 아닌 그 안의 레이아웃 어딘가에 붙어있다면 디자인 팔레트로 뷰를 만지는 경우 xmlns가 왔다갔다하고 중복되는 오류가 있을 수 있으니 이부분을 layout 태그 안으로 옮겨 놓는다.

`````ko

//instead of <code>setContentView(R.layout.activity_main)</code> : 
val binding : ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)

//binding객체는 해당xml에 대한 정보를 모두 가지고있는 객체.

binding.lifecycleOwner = this

binding.viewmodel = viewModel
`````





<h3>Two-way databinding</h3>

`````ko

        <EditText
            android:id="@+id/todo_edit"
            android:text="@={viewmodel.newTodo}"
            ...
            
//"@={}" syntax는 양방향 데이터바인딩 문법으로, programmatical하게 viewmodel의 newTodo 데이터가 바뀌든, 사용자로 인해 EditText의 text가 변경되든 두 방향 모두에서 서로 영향을 받는다.
`````

