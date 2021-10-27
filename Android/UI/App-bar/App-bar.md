# App bar

+ ActionBar
+ Toolbar

Preview : 

<img src="https://developer.android.com/images/training/appbar/appbar_sheets.png">

### summary :

> ActionBar : 
>
> + some themes set up action bar as an app bar by default.
>
> + App bar features have gradually been added to the native ActionBar over various Android releases.
>
>   -> The native ActionBar behaves differently depending on what version of the Android system a device may be using.



> Toolbar : 
>
> + The widest range covering widget.
>
> + Easy customizability.
>
> + The most recent features are added to the support library's version of Toolbar.
>
>   ->It helps ensure that app will have consistent behavior across the widest range of devices.

### e.g. Diff on providing material design experience

> ### Material design providing versions :
>
> + Toolbar : Android2.1(API level 7) or later
>
> + Native action bar : Android 5.0(API level 21) or later





## Add Toolbar as app bar

1. Add the [v7 appcompat](https://developer.android.com/tools/support-library/features#v7-appcompat) support library to your project.

   ```markdown
   ~~dependencies {...implementation "com.android.support:support-core-utils:28.0.0"}~~
   //deprecated
   ```

   `````kotlin
   dependencies {
   	...
   	implementation 'androidx.appcompat:appcompat:1.3.1'
   }
   `````

2. Make sure the activity extends [AppCompatActivity](https://developer.android.com/reference/androidx/appcompat/app/AppCompatActivity).

   ```kotlin
   class MyActivity : AppCompatActivity() {
     // ...
   }
   ```

3. Set <application> element to use one of appcompat's [NoActionBar](https://developer.android.com/reference/androidx/appcompat/R.style#Theme_AppCompat_NoActionBar) themes.

   ```kotlin
   <application
       android:theme="@style/Theme.AppCompat.Light.NoActionBar"
       />
   ```

4. Add a Toolbar to the activity's layout.

   ```kotlin
   <androidx.appcompat.widget.Toolbar
       android:id="@+id/app_bar"
       android:layout_width="match_parent"
       android:layout_height="?attr/actionBarSize"
       android:background="@color/design_default_color_primary"
       android:elevation="4dp"
       android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
       bind:popupTheme="@style/ThemeOverlay.AppCompat.Light"
       />
   ```

5. Call the activity's setSupportActionBar() method.

   ```kotlin
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContentView(R.layout.activity_main)
       setSupportActionBar(findViewById(R.id.app_bar))
   }
   
   //pass the activity's toolbar.
   //It makes accessable with a reference to and appcompat ActionBar object.
   ```