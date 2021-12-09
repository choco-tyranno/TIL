# WorkManager

> WorkManager API makes it easy to schedule deferrable, asynchronous tasks that must be run reliably.
>
> These APIs let you create a task and hand it off to WorkManager to run when the work constraints are met.



## Declaring dependencies

`````kotlin
dependencies {
    // (Java only)
    implementation("androidx.work:work-runtime:$work_version")

    // Kotlin + coroutines
    implementation("androidx.work:work-runtime-ktx:$work_version")

    // optional - RxJava2 support
    implementation("androidx.work:work-rxjava2:$work_version")

    // optional - GCMNetworkManager support
    implementation("androidx.work:work-gcm:$work_version")

    // optional - Test helpers
    androidTestImplementation("androidx.work:work-testing:$work_version")

    // optional - Multiprocess support
    implementation "androidx.work:work-multiprocess:$work_version"
}
`````













