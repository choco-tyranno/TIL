<h1>Pass data between fragments</h1>



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

// recieve destination fragment에선 setFragmentResultListener()를 통해 데이터를 받는다.
`````

