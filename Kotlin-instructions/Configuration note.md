<h1> Configuration note</h1>



<h3>Change java version</h3>

```
android{
...
	compileOptions {
    	sourceCompatibility JavaVersion.VERSION_11
    	targetCompatibility JavaVersion.VERSION_11
	}
	kotlinOptions {
    	jvmTarget = '11'
	}
}

// build.gradle(Module level). Direct change and sync pjt. It works.
/* Alternatives : Go File > Project Structure > Modules tap. And change Source compatibility/ Target compatibility.
*/
```



## Change Gradle JDK

> Move to 'Settings > Build, Execution, Deployment > Build Tools > Gradle' tap
>
> And then change Gragle JDK to purpose version.



## Kotlin JVM target

> Go : file > settings > Other settings > Kotlin compiler.
>
> Change Target JVM version.





