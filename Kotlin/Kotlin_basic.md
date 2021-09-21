# :rocket:Kotlin

### variable & constant syntax : 

`````ko
var a = "variable
val b = "constant"
val c : Int = 3

// {line:3}처럼 타입을 선언해줄수 있지만, Kotlin은 타입 자동추론시스템이 적용되어있으므로 선호되지 않는 방식.
// 변수(var)에 의한 에러 가능성을 낮추기 위해 변수를 최소화 한다. 상수(val)를 주로 사용하자.
`````



### Method syntax :

`````ko
fun work(){}
fun work() = print()
fun work(a : Int, b : Int) : Int = a + b
fun work() : Unit = print()

//{line :1,2}처럼 return 타입을 생략할 수 있다. 선호되는 방식.
//수식처럼 표현도 가능해진다.
// Java의 void는 Unit으로 표현된다.
`````



### String calculation : 

`````ko
val name = "tyranno"
val title = "choco"
val str = "${title+name} is my name"

//문자열 결합에 +연산자를 사용하지 않고 ${}를 통해 내부 연산 가능.

`````



 ### Array & List syntax :

`````ko
val items1 = arrayOf(1,2,3)
val items2 = arrayListOf(1,2,3)
val items3 = listOf(1,2,3)

//listOf로 만들어진 List는 불변객체로, 원소 변경이 불가능하다.
//arrayListOf로 만들어진 itmes2는 기존 Java의 <code>items2.set(0,0)</code>가 가능하고, items2[0] = 0 으로도 가능하다.
`````





### If syntax : 

`````ko
val str = if(x%2==0) "zero" else "left"

//기본적으로 java의 if문과 사용법이 같다. Kotlin에선위처럼 if식?처럼 사용할 수도 있다.
`````



<h3>for syntax : </h3>

`````ko
for(i in 0..9){println(i)}

val numbers = listOf{1,2,3}

for(i in numbers){println(i)}

`````



<h3>when syntax(alternative to java switch syntax) : </h3>

`````ko
when(x){

1-> print("1입니다")

2-> print("2입니다")

3, 4, 5 -> print("3,4,5중 하나입니다")

in 6..20->print("6부터 20사이의 값입니다")

!in 8..10 ->

else ->

}


//when syntax도 식이기 때문에 값을 리턴한다면 대입이 가능(e.g. val result = when(){}).
`````









