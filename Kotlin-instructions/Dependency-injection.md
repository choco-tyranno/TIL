<h1> Dependency injection</h1>

> 의존관계에 있는 객체를 외부에서 생성해서 인자로 받아서 사용하고자 하는 것. 
>
> 테스트 용이. 
>
> 모듈 단위로 객체 관리.



안드로이드 프레임워크 특성상 Activity 등 특정 시점에 자동으로 생성되는 것들이 있다.

이러한 객체들은 생성자 파라미터로 의존 객체를 받을 수 없으므로 이것을 가능하게 해주는 라이브러리들이 Dagger / Hilt.



<h3>-Dagger</h3>

> 단점 : 
>
> Application 객체 안에 컴포넌트를 만들어야 하는 등 사전에 해야할 작업이 다소 많음.
>
> Jetpack 라이브러리와 같이 사용하려다 보면 코드가 복잡해진다. 



<h3>-Hilt</h3>

> Dagger를 쉽게 사용할 수 있도록 개선된 것이 Hilt.

`````ko
@HiltAndroidApp
class App : Application()

//Application 준비
`````

`````ko
@Singleton
class SampleRepository @Inject constructor()

//주입할 객체
`````

`````ko
@AndroidEntryPoint
class MainActivity : AppCompatActivity(){
	@Inject
	lateInit var repository : SampleRepositiry

...
}

//주입 객체를 사용하는 곳. 
`````





<h3> 1. Adding dependencies</h3>

`````ko
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.38.1'
    }
}

//in root build.gradle file.
`````

`````ko
plugins {
    kotlin("kapt")
    id("dagger.hilt.android.plugin")
}

android {
    ...
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.38.1")
    kapt("com.google.dagger:hilt-android-compiler:2.38.1")
}

// Allow references to generated code
kapt {
 correctErrorTypes true
}

//in app/build.gradle file.
`````

<h2>Steps</h2>

<h3>Make App.class</h3>

`````ko
@HiltAndroidApp
class ExampleApplication : Application() { ... }
`````

`````ko
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.choco_tyranno.studyhilt">

    <application
        android:name=".App"
        
        ...
//add name property (android:name=".App") in manifests file.
`````



<h3>Inject dependencies into Android classes</h3>

`````ko
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { ... }
	@Inject lateinit var repository : MyRepository
	
//add (@AndroidEntryPoint) annotation in receive side Android class.
//define supported Object variable by lateinit var with (@Inject) annotation.
`````



<h3>Component scopes</h3>

| Android class                                 | Generated component         | Scope                     |
| :-------------------------------------------- | :-------------------------- | :------------------------ |
| `Application`                                 | `SingletonComponent`        | `@Singleton`              |
| `Activity`                                    | `ActivityRetainedComponent` | `@ActivityRetainedScoped` |
| `ViewModel`                                   | `ViewModelComponent`        | `@ViewModelScoped`        |
| `Activity`                                    | `ActivityComponent`         | `@ActivityScoped`         |
| `Fragment`                                    | `FragmentComponent`         | `@FragmentScoped`         |
| `View`                                        | `ViewComponent`             | `@ViewScoped`             |
| `View` annotated with `@WithFragmentBindings` | `ViewWithFragmentComponent` | `@ViewScoped`             |
| `Service`                                     | `ServiceComponent`          | `@ServiceScoped`          |

`````ko
@Singleton
class MyRepository @Inject constructor(){

}
`````



<h2>Hilt module</h2>

>Hilt support object creation management by module.
>
>Hilt module is a class that is annotated with (@Module).
>
>It informs Hilt how to provide instance of certain types(Like Dagger).
>
>Annotate with (@InstallIn) in Hilt modules to tell Hilt which Android class each module will be used or installed in(Unlike Dagger).

`````ko
@Module
@InstallIn(SingletonComponent::class)
object ApplicationModule {

    @Provides
    fun provideHash() = hashCode().toString()

}
`````

<h3>Generated components for Android classes</h3>

| Hilt component              | Injector for                                  |
| :-------------------------- | :-------------------------------------------- |
| `SingletonComponent`        | `Application`                                 |
| `ActivityRetainedComponent` | N/A                                           |
| `ViewModelComponent`        | `ViewModel`                                   |
| `ActivityComponent`         | `Activity`                                    |
| `FragmentComponent`         | `Fragment`                                    |
| `ViewComponent`             | `View`                                        |
| `ViewWithFragmentComponent` | `View` annotated with `@WithFragmentBindings` |
| `ServiceComponent`          | `Service`                                     |

<h3>Inject dependencies into Android classes</h3>

`````ko

@AndroidEntryPoint
class MainFragment : Fragment(R.layout.fragment_main) {
...

    @Inject
    lateinit var applicationHash : String

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Log.d("MainFragment", "appHash: $applicationHash")
...
    }
}
`````





<h3> Provide type & Qualifier</h3>

`````ko
@Module
@InstallIn(SingletonComponent::class)
object ApplicationModule {
    @Provides
    fun provideHash() = hashCode().toString()

    @Provides
    fun provideTestString() = "test"

}

//Same type providing occur the below error.
//error: [Dagger/DuplicateBindings] java.lang.String is bound multiple times:
`````

<h3>Make Qualifier</h3>

`````ko
@Module
@InstallIn(SingletonComponent::class)
object ApplicationModule {
    @AppHash
    @Provides
    fun provideHash() = hashCode().toString()
    
    @TestString
    @Provides
    fun provideTestString() = "test"
}

//Make qualifier and provide it.
//Write custom annotation (e.g. @AppHash) and (alt+enter) for create annotation AppHash class file.
`````

`````ko
@Retention(AnnotationRetention.RUNTIME)
@Qualifier
annotation class AppHash

//Attach (@Retention, @Qualifier) annotation.
//(@Qualifier) for Hilt.
`````





<h3> Inject ViewModel objects with Hilt</h3>

`````ko
@HiltViewModel
class MainViewModel @Inject constructor(
    private val repository: MyRepository
) : ViewModel(){
    fun getRepositoryHash() = repository.hashCode().toString()
}
`````

`````ko
@Singleton
class MyRepository @Inject constructor()
`````

