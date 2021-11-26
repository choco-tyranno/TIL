# App-build

## Shrink, obfuscate, and optimize app

> If you using Android Gradle plugin 3.4.0 or higher, the plugin no longer uses Proguard to perform compile-time code optimization.
>
> Instead, the plugin works with the R8 compiler to handle the following compile-time tasks :
>
> + Code shrinking(or tree-shaking)
> + Resource shrinking
> + Obfuscation
> + Optimization
>
> note : R8 works with all of your existing ProGuard rules files.
>
> R8 is the default compiler that converts your project's Java bytecode into the DEX format that runs on the Android platform. But it is not enabled by default.



### Enable shrinking, obfuscation, and optimization

````kotlin
android {
    buildTypes {
        release {
            // Enables code shrinking, obfuscation, and optimization for only
            // your project's release build type.
            minifyEnabled true

            // Enables resource shrinking, which is performed by the
            // Android Gradle plugin.
            shrinkResources true

            // Includes the default ProGuard rules files that are packaged with
            // the Android Gradle plugin. To learn more, go to the section about
            // R8 configuration files.
            proguardFiles getDefaultProguardFile(
                    'proguard-android-optimize.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
````









 