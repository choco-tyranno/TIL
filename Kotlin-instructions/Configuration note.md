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



<h3>Kotlin JVM target</h3>

> Go : file > settings > Other settings > Kotlin compiler.
>
> Change Target JVM version.





