<h1>Get the last known location</h1>



<h3>Declare dependencies for Google Play services</h3>

`````ko
apply plugin: 'com.android.application'

...

dependencies {
    implementation 'com.google.android.gms:play-services-location:18.0.0'
}
`````



<h3>Use 'FusedLocationProviderClient'</h3>

`````ko
class MainViewModel @Inject constructor(
    private val service: MaskService,
    private val fusedLocationClient : FusedLocationProviderClient
    ) : ViewModel(){
    
...
}

//In this case, DI applied. Because of context required problem with using ViewModel.
`````

<h3>'LocationModule' for providing FusedLocationProviderClient</h3>

```
@InstallIn(SingletonComponent::class)
@Module
object LocationModule {

    @Provides
    fun provideFusedLocationClient(@ApplicationContext context: Context) : FusedLocationProviderClient {
        return LocationServices.getFusedLocationProviderClient(context)
    }
}
```

<h3>Get the last known location</h3>

`````ko
/* FOCUS CODE:
fusedLocationClient.lastLocation
        .addOnSuccessListener { location : Location? ->
            // Got last known location. In some rare situations this can be null.
        }
*/



@HiltViewModel
class MainViewModel @Inject constructor(
    private val service: MaskService,
    private val fusedLocationClient : FusedLocationProviderClient
    ) : ViewModel(){

    val itemLiveData = MutableLiveData<List<Store>>()
    val loadingLiveData = MutableLiveData<Boolean>()
    init {
        fetchStoreInfo()
    }

    @SuppressLint("MissingPermission")
    fun fetchStoreInfo(){
        loadingLiveData.value = true
        fusedLocationClient.lastLocation.addOnSuccessListener { location->
            viewModelScope.launch {
                val storeInfo = service.fetchStoreInfo(location.latitude, location.longitude)
                itemLiveData.value = storeInfo.stores
                loadingLiveData.value = false
            }
        }.addOnFailureListener {exception->
            loadingLiveData.value = false
        }
    }
}

//(@SuppressLint("MissingPermission") : deligate Requesting permission to activity.
`````

`````ko
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    private val viewModel : MainViewModel by viewModels()
    private val requestPermission = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ){map ->
        if(map[Manifest.permission.ACCESS_FINE_LOCATION]!! or
            map[Manifest.permission.ACCESS_COARSE_LOCATION]!!){
            viewModel.fetchStoreInfo()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED && 										ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            requestPermission.launch(arrayOf(
                Manifest.permission.ACCESS_FINE_LOCATION,
                Manifest.permission.ACCESS_COARSE_LOCATION
            ))
            return
        }
	...
	}
...
}

//Define requestPermission : ActivityRequestLauncher.
//And checking is permission has been granted. If nor, request launcher will be executed.
`````

