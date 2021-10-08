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





## Strategy

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









