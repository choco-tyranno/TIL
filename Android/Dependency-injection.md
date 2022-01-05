<h1> Dependency injection</h1>

> ### Intro :
>
> + Android 권장 앱 아키텍처는 관심사가 분리된 클래스로 코드를 다루길 권장.
>   계층 구조의  각 클래스는 단일책임을 갖고 클래스들은 사이의 종속성 관계가 발생할 수 있음.
> + DI를 사용하면 클래스를 쉽게 연결. 테스트를 위한 구현 교체 쉬움.

## 수동 종속성 삽입

+ 예 : 로그인 flow(앱 기능에 상응하는 화면 그룹으로 간주)에서의 종속 관계(가르키는 방향으로 종속), 
  LoginActivity ->LoginViewModel -> UserRepository 
  ->(UserLocalDataSource->`Room`) or (UserRemoteDataSource -> `Retrofit`)
+ 이러한 종속항목들을 필요되는 부분에서 항상 생성해 사용한다면, 객체 재사용이 어렵움.
+ 재사용하려면 싱글톤 패턴을 따르게되는데, 테스트 시 모든 테스트가 싱글톤 인스턴스를 공유하므로 테스트가 더 어려워짐.

### 컨테이너로 종속성 관리

+ 객체 재사용 문제 솔루션으로 자체 종속항목 컨테이너를 만드는 방법.
+ Application 클래스에 컨테이너 인스턴스를 배치하여 접근성을 만들 수 있음.
  Application에서 컨테이너 인스턴스를 가져와서 공유된 종속항목 인스턴스(e.g. AppContainer.userRepository) 획득 가능.
+ 여러 위치에서 종속항목이 필요한 경우(예: ViewModel) Factory패턴으로 인스턴스를 만드는 것이 좋다.

### 어플리케이션 흐름에서 종속성 관리

+ Flow 가 다양해질 수 있고, 앱 상태 손실을 방지하기 위해 최적화(해당 Flow에서 필요하지 않은 인스턴스 삭제)로 종속항목 객체가 Flow 범위에만 속하게 하는 것이 권장됨.
+  Flow에 따른 Container(e.g. LoginContainer)를 AppContainer의 필드 변수(프로퍼티)로 두어 해당 Container의 라이프사이클을 관리해야하는 곳(예:LoginActivity)에서 만들고 삭제.
  (LoginActivity의 onCreate, onDestroy 라이프사이클 메서드에 LoginContainer 객체를 만들고 삭제하는 코딩). `note: ` `Lifecycler-aware Component` 라이프사이클 관찰 사용이 권장됨.
+ 구성변경에도 유지되는 컨테이너가 필요하다면 [UI 상태 저장 가이드](https://developer.android.com/topic/libraries/architecture/saving-states?authuser=1) 참고. 프로세스 종료 처리 방식과 동일하게 처리해야함.



## Hilt 종속성 주입 라이브러리

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

> note : @ActivityRetainedScope will safe guard from configuration changes such screen orientation or language changes. However, @ActivityScoped` is negative.





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

