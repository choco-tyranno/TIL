# Android Studio(vr.Arctic fox) - project initializing

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
+  Setting -> Other Settings -> Kotlin Compiler
  `set : ` Target JVM version : 11