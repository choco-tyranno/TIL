<h1>Navigation</h1>



<h3> Environment</h3>

`````ko
dependencies {
...
  def nav_version = "2.3.5"

  // Kotlin
  implementation("androidx.navigation:navigation-fragment-ktx:$nav_version")
  implementation("androidx.navigation:navigation-ui-ktx:$nav_version")
}
`````





<h3>Create a navigation graph</h3>

+ (optional) Create 'navigation-res-directory' into 'res' directory.
+ Create navigation graph res file(e.g. name : nav_graph) into 'navigation-res-directory'.
+ Open nav_graph.xml. Click 'new destination icon(+)' for add all fragments what has created for destination target.
+ Connect arrow between destinations.
+ (optional) Change label attribute.





<h3>Add navigation graph in activity</h3>

+ Open 'activity.xml' and on design mode, search 'NavHostFragment' view 

  and drag & drop into activity design board.



<h3>Navigate</h3>

`````ko
class MainFragment : Fragment(){
...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        button.setOnClickListener {
            findNavController().navigate(R.id.action_mainFragment_to_secondFragment)
        }
    }
}
`````



<h3>Update Action bar</h3>

`````ko
import androidx.navigation.findNavController
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.ui.AppBarConfiguration
import androidx.navigation.ui.navigateUp
import androidx.navigation.ui.setupActionBarWithNavController

class MainActivity .. {
...
	private lateinit var appBarConfiguration: AppBarConfiguration

...
	override fun onCreate(savedInstanceState: Bundle?) {
    ...

    	val navHostFragment =
        	supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as 				NavHostFragment
    	val navController = navHostFragment.navController
    	appBarConfiguration = AppBarConfiguration(navController.graph)
    	setupActionBarWithNavController(navController, appBarConfiguration)
	}
	
	override fun onSupportNavigateUp(): Boolean {
    	val navController = findNavController(R.id.nav_host_fragment)
    	return navController.navigateUp(appBarConfiguration)
        	    || super.onSupportNavigateUp()
	}

...
}


//import 타겟 주소가 여러개인 경우가 있어 주의.
`````

