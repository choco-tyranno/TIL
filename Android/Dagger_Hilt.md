# Dagger - Hilt



+ Hilt는 ServiceLocator패턴의 보일러플레이트가 제거됨.
+ @HiltAndroidApp 주석을 Application 클래스에 붙임.
  -> 앱 수명주기에 연결된 컨테이너 추가됨.
+ Hilt를 사용하는 Android클래스에 @AndroidEntryPoint 주석 붙임.
  -> 해당 Android클래스의 수명주기에 연결된 종속 항목 컨테이너가 생성됨. 
  `⚠️note :` Hilt는 현재 Android 유형 중 `Application`(`@HiltAndroidApp`주석 필요), `Activity`, `Fragment`, `View`, `Service`, `BroadcastReceiver`를 지원
+ 컨테이터 트리에서 상위 컨테이너가 제공하는 종속항목은 하위 컨테이너에서도 엑세스 가능.
+  @Inject 주석을 붙여 인스턴스를 `필드 삽입`.
+ `⚠️note : ` Injection된 필드는 private 불가.
+ @Inject주석을 Injection 하려는 클래스의 생성자에 붙임.
  -> Hilt에서 해당 클래스의 인스턴스 제공방법을 알게됨. `binding'`이라 부름.
+ 기본적으로 Hilt의 모든 binding은 범위가 지정되지 않음.
  -> 요청시 마다 새 인스턴스 생성해 제공됨. 단, `범위 주석`을 붙이면 싱글 인스턴스가 공유됨.
+ 인스턴스 범위를 주석에 따른 컨테이너로 지정.
  `⚠️note : ` 어디에서 사용되는지와 관계없이 해당 컨테이너에서 항상 같은 인스턴스를 제공.
  -> 아래 `구성요소 범위`  주석 참고.
+ Hilt는 수명주기가 서로 다른 여러 컨테이너를 생성할수있으므로 컨테이너 범위 지정 주석이 존재.
+ 인터페이스나 프로젝트에 포함되지 않은 클래스와 같은 생성자가 삽입될 수 없는 유형의 인스턴스 제공은 @Inject를 붙일 수 없고, Hilt 모듈을 사용.
+ @Module과 @InstallIn 주석이 붙은 것이 Hilt 모듈 클래스.
  -> @Module은 모듈임을 알리는, @InstallIn은 파라미터를 통해 Hilt에 어느 컨테이너에서 Hilt구성요소를 사용할 수 있는지 알려줌. 하단 `Android 클래스용으로 생성된 구성요소` 참고.
+ Kotlin에서 @Provides 함수만 포함하는 모듈은 object클래스 가능.
  -> 제공자가 최적화되고 생성된 코드에 대부분 인라인됨.
+  @Provides 주석을 Hilt모듈의 함수에 붙여 인스턴스 제공방법 알림.
   `⚠️note : ` 매개변수로 필요한 종속항목도 따로 @Provide함수로 제공방법을 알려놓으면 매치됨.
+ `⚠️note : ` Hilt모듈의 @InstallIn주석 매개변수로 필요되는 구성요소는 멤버 항목들의 스코프와 연결됨(예 : ActivityScoped의 경우 activity별로 각자 다른 activity 구성요소가 생성되고 그 각자의 구성요소와 함께 생명주기에 따른 종속항목 인스턴스가 관리됨).
  `⚠️note : ` 싱글 인스턴스 제공 여부는 @Provide함수가 붙은 제공함수 또는 해당 클래스 생성자에 따로 타입 범위 주석을 붙여야함.
+ Hilt컨테이너는 맞춤 결합에 종속 항목으로 삽입할 수 있는 일련의 기본 결합을 제공.
  -> @ApplicationContext 같은 주석 또는 하단 `구성요소 기본 결합` 참고.
+ @EntryPoint과 @InstallIn 주석을 달아 Hilt가 지원하지 않는 클래스에 필드 삽입.
  ->Hilt가 관리하는 객체의 그래프에 코드가 들어가는 지점이 됨.
  ->진입점을 설치할 구성요소를 지정.
+ EntryPointAccessors를 정적 메서드로 진입점에 엑세스. 매개변수로 구성요소 인스턴스 또는 구성요소 소유자 역할을 하는 @AndroidEntryPoint 객체 사용됨. 하단 `Hilt가 지원하지 않는 클래스에 종속 항목 삽입` 참고.
  ⚠️note : 매개변수로 전달하는 구성요소와 @EntryPoint 인터페이스의 @InstallIn 주석에 있는 Android 클래스와 일치해야함.
+ Fragment에 Injection하는 경우, Activity에도 @AndroidEntryPoint 붙여줘야함.
  Hilt가 호스팅하는 안드로이드 구성요소를 알아야 하므로.
+ @InstallIn의 결합 컨테이너 지정은 구성요소를 종속항목으로 포함하는지 여부로 결정.
+ 추상클래스 모듈로 인터페이스 제공.
  @Binds 주석을 추상 제공 함수에 붙임.
  구현은 고유한 매개변수를 추가해 지정.
+ @Binds 또는 @Provides가 특정유형 결합을 지정하면 구현 클래스에서 동일한 기능을 하는 주석은 삭제해도 됨.
+ Hilt에 동일한 타입의 다른 구현들(여러개의 결합)을 제공하려면 한정자(Qualifier) 사용.
+ annotation class 에 @Qualifier주석을 붙임. @Bind 또는 @Provides 함수에 붙임.





```
buildscript {
    ...
    ext.hilt_version = '2.28-alpha'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

```
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}
...
dependencies {
    ...
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```



## Android클래스 Injection이 수행되는 lifecycle callback : 

| 생성된 구성요소             | 생성 위치                | 제거 위치                 |
| :-------------------------- | :----------------------- | :------------------------ |
| `ApplicationComponent`      | `Application#onCreate()` | `Application#onDestroy()` |
| `ActivityRetainedComponent` | `Activity#onCreate()`    | `Activity#onDestroy()`    |
| `ActivityComponent`         | `Activity#onCreate()`    | `Activity#onDestroy()`    |
| `FragmentComponent`         | `Fragment#onAttach()`    | `Fragment#onDestroy()`    |
| `ViewComponent`             | `View#super()`           | 제거된 뷰                 |
| `ViewWithFragmentComponent` | `View#super()`           | 제거된 뷰                 |
| `ServiceComponent`          | `Service#onCreate()`     | `Service#onDestroy()`     |



## 구성요소 범위 :

| Android 클래스                               | 생성된 구성요소             | 범위                     |
| :------------------------------------------- | :-------------------------- | :----------------------- |
| `Application`                                | `ApplicationComponent`      | `@Singleton`             |
| `View Model`                                 | `ActivityRetainedComponent` | `@ActivityRetainedScope` |
| `Activity`                                   | `ActivityComponent`         | `@ActivityScoped`        |
| `Fragment`                                   | `FragmentComponent`         | `@FragmentScoped`        |
| `View`                                       | `ViewComponent`             | `@ViewScoped`            |
| `@WithFragmentBindings` 주석이 지정된 `View` | `ViewWithFragmentComponent` | `@ViewScoped`            |
| `Service`                                    | `ServiceComponent`          | `@ServiceScoped`         |



## Android 클래스용으로 생성된 구성요소 : 

| Hilt 구성요소               | 인젝터 대상                                  |
| :-------------------------- | :------------------------------------------- |
| `ApplicationComponent`      | `Application`                                |
| `ActivityRetainedComponent` | `ViewModel`                                  |
| `ActivityComponent`         | `Activity`                                   |
| `FragmentComponent`         | `Fragment`                                   |
| `ViewComponent`             | `View`                                       |
| `ViewWithFragmentComponent` | `@WithFragmentBindings` 주석이 지정된 `View` |
| `ServiceComponent`          | `Service`                                    |



## 구성요소 기본 결합 :

| Android 구성요소            | 기본 결합                                     |
| :-------------------------- | :-------------------------------------------- |
| `ApplicationComponent`      | `Application`                                 |
| `ActivityRetainedComponent` | `Application`                                 |
| `ActivityComponent`         | `Application`, `Activity`                     |
| `FragmentComponent`         | `Application`, `Activity`, `Fragment`         |
| `ViewComponent`             | `Application`, `Activity`, `View`             |
| `ViewWithFragmentComponent` | `Application`, `Activity`, `Fragment`, `View` |
| `ServiceComponent`          | `Application`, `Service`                      |

```kotlin
//구성요소 기본 결합을 종속항목에 사용한 예시.
//1. 주석을 사용한 방법과 2. 기본결합을 사용한 방법.

class AnalyticsAdapter @Inject constructor(
  @ActivityContext context: Context
) { ... }

// The Activity binding is available without qualifiers.
class AnalyticsAdapter @Inject constructor(
  activity: FragmentActivity
) { ... }
```



## Hilt가 지원하지 않는 클래스에 종속 항목 삽입 예시 :

```kotlin
class ExampleContentProvider : ContentProvider() {

  @EntryPoint
  @InstallIn(ApplicationComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun analyticsService(): AnalyticsService
  }

  ...
}
```

```kotlin
class ExampleContentProvider: ContentProvider() {
    ...

  override fun query(...): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint =
      EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)

    val analyticsService = hiltEntryPoint.analyticsService()
    ...
  }
}
```





## Hilt가 생성하는 구성요소의 계층 구조 :

<img src="https://developer.android.com/images/training/dependency-injection/hilt-hierarchy.svg?hl=ko"/>



## 추상 모듈클래스로 인터페이스 제공 : 

```kotlin
@InstallIn(ActivityComponent::class)
@Module
abstract class NavigationModule {

    @Binds
    abstract fun bindNavigator(impl: AppNavigatorImpl): AppNavigator
}
```

```kotlin
class AppNavigatorImpl @Inject constructor(
    private val activity: FragmentActivity
) : AppNavigator {
    ...
}
```

### 사용 예 :

```kotlin
//Activity

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject lateinit var navigator: AppNavigator

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            navigator.navigateTo(Screens.BUTTONS)
        }
    }

    ...
}
```

```kotlin
//Fragment
@AndroidEntryPoint
class ButtonsFragment : Fragment() {

    @Inject lateinit var logger: LoggerLocalDataSource
    @Inject lateinit var navigator: AppNavigator

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_buttons, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        ...
    }
}
```

-> 제공함수 bindNavigator에 타입 범위가 지정되지 않았기때문에 Activity와 Fragment에서 각각 다른 인스턴스를 갖게됨. 
`⚠️note : ` 타입범위 지정시, 구성요소 계층구조에 따른 인스턴스 스코프를 갖음.
(예 : ActivityComponent의 경우 activity 하위의 계층에서 접근가능. activity별로 서로다른 activity 구성요소가 만들어지고 각 activity 내에서 싱글 인스턴스로 관리됨.)

