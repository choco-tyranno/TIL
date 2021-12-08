# 🛠️Kotlin

## Principle of operation

> Kotlin -> Java -> Class
>
> Kotlin -> C -> Binary Code
>
> Kotlin -> JavaScript



## Characteristics

> + Simple grammer
> + Null safety - Nullable variable & Non-null variable
> + Exception handling not forced
> + All primitive types are managed as Object
> + Operator reimplementation supported
> + Object oriented programming & Functional programming supported



## Work field

> + Server side - Spring Framework 5 or over
> + Android
> + JavaScript - 'Web field using JavaScript, ECMAScript' & 'node.js developing field'
> + Native - iOS, Mac, watchOS, tvOS, Android Native, Windows, Linux ... etc
> + Data science - data analysis, machine learning, deep learning developing field
> + ETC ..



## Constructor

`````kotlin
class TestClass constructor(val a1 :Int,var a2 :Int)

//주생성자에서는 constructor 키워드 생략가능.
//바디가 필요없으면 바디블록도 생략가능.
//바디 안에 constructor 키워드를 달아 보조생성자를 만들 수 있으나, 주생성자가 있다면 주생설자를 호출해줘야함.

class TestClass2(val a1: Int, val a2:Int){
    constructor : this(100, 200){
    }
}
//보조 생성자를 통하는 경우 보조생성자가 먼저 호출되고 주생성자를 먼저 실행한뒤 보조생성자의 블록이 실행된다.
`````



## Access modifier

> ### Class :
>
> private : 외부에서 객체 생성 불가.
>
> public(기본) : 외부에서 객체를 생성할 수 있다.
>
> ~~protected~~
>
> internal : 같은 모듈인 경우에만 객체를 생성할 수 있다.

> ### variable, method :
>
> private : 외부에서 접근 불가.
>
> public(기본) : 외부에서 접근이 자유로움. 변수의 경우 getter와 setter 자동 생성됨.
>
> protected : 상속관계에서만 접근 가능.
>
> internal : 같은 모듈인 경우에만 접근 가능.



## LateInit

> var로 선언된 변수의 초기화를 뒤로 미룰 수 있다.
>
> 변수의 값을 사용하기 전에 반드시 초기화가 이뤄져야 한다.
> ->'UninitializedPropertyAccessException' 발생.
>
> val로 선언된 변수는 오류가 발생한다.

> 프로퍼티의 값을 나중에 셋팅할 때에는 지연초기화를 사용한다.
>
> var 변수는 lateinit을 사용.
>
> val 변수는 lazy 코드블록 사용.

`````kotlin
class LateInitTest {
    var a1 : Int
    lateinit var a2 : String //'lateinit' modifier is not allowed on properties of primitive types.
    init{
        a1 = 100
    }
}
`````

`````kotlin
//리플렉션을 이용한 초기화 체크 방법
fun testMethod(){
    if(::a2.isInitialized == false){
        a2 = "테스트"
    }
    println("a2 : $a2")
}
`````

`````kotlin
//lazy 코드블록을 이용한 지연 초기화
//사용할 때 값을 초기화한다는 의미 O
//나중에 프로퍼티의 값을 셋팅한다는 의미 X
//메모리 절약의 이점

val a3 : String by lazy{
    "테스트"
}

`````









