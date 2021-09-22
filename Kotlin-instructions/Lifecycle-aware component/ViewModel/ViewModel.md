<h1>ViewModel</h1>

+ The viewModel class is designed to store and manage UI-related data in a lifecycle conscious way

+ ViewModel을 사용하면 환경설정(e.g. 화면 회전에 따른 기존 화면파괴)이 변해도 Data가 파괴되는 것을 막는다.

+ 비즈니스 로직을 ViewModel로 옮겨 UI와 비즈니스 로직을 분리한다.



<h3>Add dependency</h3> 

`````kotlin
val lifecycle_version = "2.4.0-beta01"

// ViewModel
implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version")
`````



<h3>Get Instance</h3>

````k
//1.
val viewModel = ViewModelProvider(this).get(MainViewModel::class.java)


//Additional dependency(lifecycle extention) needed.
//-> implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'

//2
val viewModel : MainViewModel by viewModels()

//Additional dependency(activity-ktx) needed.
//-> implementation("androidx.activity:activity-ktx:$activity_version")
//Check source compatibility & target compatibility (1.8 or over).
//File > Project structure > Modules에서 설정가능.
//-> 설정을 바꾸면 build.gradle에서 아래와 같이 설정된다.
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
//추가적으로 compileOptions과 같은 레벨에 아래 코드를 추가한다.
    kotlinOptions {
        jvmTarget = '1.8'
    }
````