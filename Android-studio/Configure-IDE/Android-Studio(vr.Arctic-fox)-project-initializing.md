# Android Studio(vr.Arctic fox) - project initializing

> To enable Java 11 feature, set `compileOptions` to the desired java version and set `compileSdkVersion` to 30 or above. 

+ Gradle Scripts -> build.gradle(Module)

  ```kotlin
  android{
      compileOptions {
          sourceCompatibility JavaVersion.VERSION_11
          targetCompatibility JavaVersion.VERSION_11
      }
      kotlinOptions {
          jvmTarget = '11'
      }
  }
  
  ```

+ Setting ->Build,Execution, Deployment -> Build Tools -> Gradle
  `Set : ` Gradle JDK : version 11

+ Setting -> Other Settings -> Kotlin Compiler
  `set : ` Target JVM version : 11

+ Gradle Scripts -> settings.gradle
  `remove :` 'jcenter()' code line 

  ```kotlin
  dependencyResolutionManagement{
      repositories {
          jcenter() // Warning: this repository is going to shut down soon
      }
  }



























