# Layouts and binding expressions

## How to input string direct in @{} databinding expression.

`````ko
<TextView
	android:text='@{settings_viewmodel==null ? "null":"nonnull"}'
		
//O
//wrap whole proprerty @{settings_viewmodel==null ? "null":"nonnull"} with ''
//string value requires with ""
	
	
<TextView
	android:text="@{settings_viewmodel==null ? 'null':'nonnull'}"
	
//X
//if string value go with '', cannot resolved it.
`````

  



