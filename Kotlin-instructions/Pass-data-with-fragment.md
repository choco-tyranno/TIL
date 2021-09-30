<h1>Pass data with fragment</h1>



<h3>1. Using ViewModel</h3>

````ko
// 1.

class FirstFragment : Fragment(R.layout.fragment_first) {
    val viewModel by activityViewModels<MainViewModel>()
//activityViewModels() : lifecycle owner를 activity로.

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val firstButton = view.findViewById<Button>(R.id.first_button)
        firstButton.setOnClickListener {
            viewModel.data = "Hello"
		   firstButton.findNavController()
			.navigate(R.id.action_firstFragment_to_secondFragment)
        }
    }
}
````

`````ko
// 2.
class SecondFragment : Fragment(R.layout.fragment_second) {
    val viewModel by activityViewModels<MainViewModel>()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val secondButton = view.findViewById<Button>(R.id.second_button)
        secondButton.setOnClickListener {
            Toast.makeText(requireContext(),viewModel.data, 		Toast.LENGTH_SHORT).show()
            secondButton.findNavController().navigate(R.id.action_secondFragment_to_firstFragment)
        }
    }

}
`````



<h3> 2. Using FragmentResult API</h3>

`````ko
dependencies {
	implementation("androidx.fragment:fragment-ktx:1.3.6")
}

`````

<h3> 2-1. Implicit way 

`````ko
class FirstFragment : Fragment(R.layout.fragment_first) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val firstButton = view.findViewById<Button>(R.id.first_button)

        firstButton.setOnClickListener {
            setFragmentResult("requestKey"
                , bundleOf("data" to "Hi")
            )
            firstButton.findNavController()
            .navigate(R.id.action_firstFragment_to_secondFragment)
        }
    }
}

//Start destination fragment에서 setFragmentResult()로 result 데이터를 bundle로 만든다. 
`````

`````ko
class SecondFragment : Fragment(R.layout.fragment_second) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val secondButton = view.findViewById<Button>(R.id.second_button)

        setFragmentResultListener("requestKey"){requestKey, bundle ->
            val data = bundle.getString("data","")
            Toast.makeText(requireContext(),data, Toast.LENGTH_SHORT).show()
        }

        secondButton.setOnClickListener {
            secondButton.findNavController()
            .navigate(R.id.action_secondFragment_to_firstFragment)
        }
    }
}

// 암시적 Intent.
// recieve destination fragment에선 setFragmentResultListener()를 통해 데이터를 받는다.
`````

<h3> 2-2. Explicit way</h3>

``````ko
class FirstFragment : Fragment(R.layout.fragment_first) {
    val getStartActivityForResult = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ){activityResult ->
        activityResult?.data?.let { intent->
            intent.extras?.let { bundle ->
                Toast.makeText(
                    requireContext(),
                    "result:${bundle.getString("data","No data")}"
                    , Toast.LENGTH_SHORT
                ).show()
            }
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val firstButton = view.findViewById<Button>(R.id.first_button)

        firstButton.setOnClickListener {
            Intent(requireContext(), ResultActivity::class.java).apply {
                getStartActivityForResult.launch(this)
            }
        }
    }
}

// 명시적 Intent.
// 어떤 숫자를 정의해놓고 무엇이 호출되었는가에 따라 처리를 해주는 암시적방식과 달리,
// 관심사가 분리되는 형태로 명시적으로 처리를 정의하고 또 그것을 실행하는 식으로 나타남. 
``````

`````ko
class ResultActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_result)
        findViewById<Button>(R.id.result_button).setOnClickListener {
            val intent = Intent()
            intent.putExtra("data","Hello")
            setResult(RESULT_OK,intent)
            finish()
        }
    }
}
`````



<h1>Grant permission using FragmentResult API</h1>

`````ko

class FirstFragment : Fragment(R.layout.fragment_first) {
    val requestPermission = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ){ granted ->
        if(granted) Toast.makeText(requireContext(),"권한 승인",Toast.LENGTH_SHORT).show()
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val firstButton = view.findViewById<Button>(R.id.first_button)

        firstButton.setOnClickListener {
            requestPermission.launch(Manifest.permission.ACCESS_FINE_LOCATION)
        }
    }
}
`````
