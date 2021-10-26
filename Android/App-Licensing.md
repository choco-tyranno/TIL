# App Licensing

Using oss-license-library supported by Google play service.



## Setting Environment

`````ko
buildscript {
	...
	dependencies {
		classpath 'com.google.android.gms:oss-licenses-plugin:<version>' 
	} 
}

//in build.gradle(Project level)
`````

`````ko
dependencies{
	...
	implementation 'com.google.android.gms:play-services-oss-licenses:<version>'
}

//in build.gradle(App level)
`````



## Start OssLicensesMenuActivity

`````ko
class MainActivity : AppCompatActivity(){
	override fun onCreate(savedInstanceState: Bundle?){
		...
		button.setOnClikckListener {
			startActivity(Intent(this, OssLicensesMenuActivity::class.java)
			)
			
			//optional :
			OssLicensesMenuActivity.setActivityTitle("오픈소스 라이선스")
		}
	}
}
`````



## + Internal workings

> getDependencies()
>
> https://github.com/google/play-services-plugins/blob/master/oss-licenses-plugin/src/main/groovy/com/google/android/gms/oss/licenses/plugin/DependencyTask.groovy
>
>
> generateLicenses()
>
> https://github.com/google/play-services-plugins/blob/master/oss-licenses-plugin/src/main/groovy/com/google/android/gms/oss/licenses/plugin/LicensesTask.groovy

