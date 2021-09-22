# :rocket:Kotlin



### variable & constant syntax : 

`````ko
var a = "variable
val b = "constant"
val c : Int = 3

// {line:3}처럼 타입을 선언해줄수 있지만, Kotlin은 타입 자동추론시스템이 적용되어있으므로 선호되지 않는 방식.
// 변수(var)에 의한 에러 가능성을 낮추기 위해 변수를 최소화 한다. 상수(val)를 주로 사용하자.
`````



<h3> Data type</h3>

`````ko
val str1 :String? = null     //가능
val str2 :String = null		//불가능

//kotlin에선 error를 피하기 위해 기본적으로 변수에 null이 들어갈 수 없다.
//Type 뒤에 ?가 붙은 타입만이 null이 가능한 타입. 이를 사용하기 위해선 null check가 필요.

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



<h3>Class</h3>

`````ko
class Person(var name:String){
}


//class 정의할때 기본적으로 생성자를 넣어줄수있다.
//public과 같은 접근제한자를 쓸필요 X. 디폴트가 public.
//생성시 new 키워드 안씀.
//java에서 field라고 부르는 것들을 kotlin에선 property라고 부름
//init{}이라는 코드가 존재. 생성자가 호출된 이후에 실행되는 코드 블록.
//함수도 public이 기본이고 명시하지 않는다. private으로 사용할 경우에만 명시한다.
//java에서의 @Override 어노테이션이 kotlin에선 override 키워드로 사용됨.


person.name = "김길동" 

//직접 접근하는 것처럼 생겼지만 내부적으로는 getName을 호출한다.
`````



<h3>data class</h3>

`````ko
data class Person(val name:String, var age :Int)


//data class는 toString/getter & setter/equals & hashCode를 재정의 한 것으로 본다.
`````



<h3> Inheritance , Interface</h3>

`````ko
class MainActivity : AppCompatActivity(), View.OnClickListener{}


//상속받는 클래스의 어떤 생성자가 호출되는지까지 표기되어있다.
//한편, 인터페이스는()를 안붙인다.
`````



<h3>확장함수</h3>

`````ko
fun String.myLength() : Int {return this.length}

//Kotlin에선 String이라는 final 클래스에도 상속없이 함수를 추가할 수 있다.

`````



<h3>자바와 코틀린의 혼용</h3>

상호 호환이 가능. 단, 몇가지 주의점이 있다.



