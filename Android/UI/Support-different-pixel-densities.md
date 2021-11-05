# Support different pixel densities

+ ## Use density-independent pixels

  1. Convert dp units to pixel units.

     `````kotlin
     px = dp * (dpi / 160)
     `````

     ```kotlin
     // The gesture threshold expressed in dp
     private const val GESTURE_THRESHOLD_DP = 16.0f
     ...
     private var mGestureThreshold: Int = 0
     ...
     override fun onCreate(savedInstanceState: Bundle?) {
         super.onCreate(savedInstanceState)
     
         // Get the screen's density scale
         val scale: Float = resources.displayMetrics.density
         // Convert the dps to pixels, based on density scale
         mGestureThreshold = (GESTURE_THRESHOLD_DP * scale + 0.5f).toInt()
     
         // Use mGestureThreshold as a distance in pixels...
     }
     ```

     The DisplayMetrics.density field specifies the scale factor you must use to convert dp units to pixels.

     > medium-density screen : 1.0
     >
     > high-density screen : 1.5
     >
     > extra-high-density screen : 2.0
     >
     > low-density screen : .0.75

     

  2. Use pre-scaled configuration values.