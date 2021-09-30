<h1> Activity Lifecycle</h1>





<h2>How to detect changing lifecycle</h2>

`````ko
class MainActivity ... {
	override fun onCreate(savedInstanceState: Bundle?){
		super.onCreate(savedInstanceState)
		Log.d(TAG, "MainActivity created")
		...
		
		registerActivityLifecycleCallbacks(object : Application.ActivityLifecycleCallbacks{
            override fun onActivityStarted(activity: Activity) {
                Log.d(TAG, "onActivityStarted")
            }

            override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {
                Log.d(TAG, "onActivityCreated")
            }

            override fun onActivityResumed(activity: Activity) {
                Log.d(TAG, "onActivityResumed")
            }

            override fun onActivityPaused(activity: Activity) {
                Log.d(TAG, "onActivityPaused")
            }

            override fun onActivityStopped(activity: Activity) {
                Log.d(TAG, "onActivityStopped")
            }

            override fun onActivitySaveInstanceState(activity: Activity, outState: Bundle) {
                Log.d(TAG, "onActivitySaveInstanceState")
            }

            override fun onActivityDestroyed(activity: Activity) {
                Log.d(TAG, "onActivityDestroyed")
            }
        })
	}
	
	companion object{
		private val TAG = MainActivity::class.java.simpleName
	}
}


//onActivityCreated() cannot detected. -> alternatively, use Log.d() in onCreate().
`````

