# Design Pattern - Behavioral

## Behavioral Patterns

- Observer / Listener
- Strategy
- Command
- State
- Chain of Responsibility
- Visitor
- Mediator
- Memento

> ref : https://github.com/dbacinski/Design-Patterns-In-Kotlin
>
> Project maintained by [@dbacinski](http://twitter.com/dbacinski) (Dariusz Baciński)



## 1. Observer / Listener

> The observer pattern is used to allow an object to publish changes to its state. Other objects subscribe to be immediately notified of any changes.

### Example

`````ko
interface TextChangedListener {
	fun onTextChanged(oldText:String, newText: String)
}

class PrintingTextChangedListener : TextChangedListener {
	private var text = ""
	
	override fun onTextChanged(oldText:String, newText: String) {
		text = "Text is changed: $oldText -> $newText"
	}
}

class TextView {
	val listeners = mutableListOf<TextChangedListener>()
	var text = String by Delegates.observable("<empty>") { _, old, new ->
		listeners.forEach {it.onTextChanged(old, new) }
	}
}

`````

### Usage

`````ko
val textView = TextView().apply{
	listener = PrintingTextChangedListener()
}

with(textView) {
	text = "Lorem ipsum"
	text = "dolor sit amet"
]
`````

### Output

`````ko
Text is changed <empty> -> Lorem ipsum
Text is changed Lorem ipsum -> dolor sit amet
`````





## 2. Strategy

> The strategy pattern is used to create an interchangeable family of algorithms from which the required process is chosen at run-time.

#### Example

`````ko
class Printer(private val stringFormatterStrategy: (String) -> String){
	fun printString(str: String){
		println(stringFormatterStrategy(str))
	}
}

val lowerCaseFormatter: (String) -> String = {it.toLowerCase()}
val upperCaseFormatter = {it: String -> it.toUpperCase()}
`````

#### Usage

`````ko
val inputString = "LOREM ipsum DOLOR sit amet"

val lowerCasePrinter = Printer(lowerCaseFormatter)
lowerCasePrinter.printString(inputString)

val upperCasePrinter = Printer(upperCaseFormatter)
upperCasePrinter.printString(inputString)

val prefixPrinter = Printer{"Prefix: $it"}
prefixPrinter.printString(inputString)

`````

#### Output

`````ko
lorem ipsum dolor sit amet
LOREM IPSUM DOLOR SIT AMET
Prefix: LOREM ipsum DOLOR sit amet
`````









