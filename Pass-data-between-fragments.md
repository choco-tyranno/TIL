<h1>Pass data between fragments</h1>



<h3>Using ViewModel</h3>

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



