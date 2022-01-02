# AndroidTest with Hilt

+ Hilt를 사용해 UI테스트에 종속 항목을 삽입.



## 테스트 종속 항목 추가

```kotlin
//app/build.gradle
...
dependencies {

    // Hilt testing dependency
    androidTestImplementation "com.google.dagger:hilt-android-testing:$hilt_version"
    // Make Hilt generate code in the androidTest folder
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

## 맞춤 TestRunner

```kotlin
//라이브러리에 UI테스트 실행에 사용할 수 있는 HiltTestApplication이 포함되 있음.
class CustomTestRunner : AndroidJUnitRunner() {

    override fun newApplication(cl: ClassLoader?, name: String?, context: Context?): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

```kotlin
//app/build.gradle
...
android {
    ...
    defaultConfig {
        ...
        testInstrumentationRunner "com.example.android.hilt.CustomTestRunner"
    }
    ...
}
...
```



## 테스트 실행

```kotlin
@RunWith(AndroidJUnit4::class)
@HiltAndroidTest	// Hilt 구성요소 생성 담당.
class AppTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)	//구성요소 삽입 및 상태 관리에 사용. 

    ...
}
```