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

> 생성자 : 클래스를 통해 객체를 생성할 때 자동으로 수행될 코드를 작성하는 곳.
>
> 메서드와 비슷해 보이지만 반환타입이 없어 메서드라 부르지 않음.
>
> 클래스의 변수 값 초기화에 주로 이용됨.

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

### init 코드 블록 :

> 객체 생성시 자동으로 처리되는 코드를 만들 수 있음.
>
> 기본 호출 순서 : main constructor -> init
>
> sub constructor를 통한 객체 생성시 호출 순서 : main constructor -> init -> subconstructor



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

## Operator

> + Arithmethic operator :
>
>   ```kotlin
>   expression / Translated to
>   a + b / a.plus(b)
>   a - b / a.minus(b)
>   a * b / a.times(b)
>   a / b / a.div(b)
>   a % b / a.rem(b), //a.mod(b) deprecated
>   a..b / a.rangeTo(b)
>     
>   ```
>
>   

## Lateinit

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

`note :` lateinit var로 선언된 변수는 always not null. 내부적으로 Function<>이 내장되어있다. 

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



## Overriding

> + Overriding : 부모 클래스가 가진 메서드를 자식 클래스에서 재정의하는 개념.
> + 부모가 가진 메서드의 이름, 매개변수 형태 모두 동일해야 한다.
> + 모든 객체는 부모 클래스형 참조 변수에 담을 수 있다.
> + 부모 클래스형 참조변수를 사용하면 부모 클래스에 정의되어 있는 멤버만 사용이 가능하다.
> + 부모형 참조변수를 통해 Overriding된 자식 메서드를 호출할 수 있다.
> + 자식 클래스 영역에서 super키워드를 통해 부모의 메서드를 호출할 수 있다. 

```kotlin
open class SuperClass{
    open fun superMethod(){
        println("superMethod입니다.")
    }
}
class SubClass : SuperClass(){
    override fun superMethod(){
        println("subMethod입니다.")
    }
}

fun main{
    val obj1 : SuperClass = SubClass()
    obj1.superMethod()
    //output: subMethod입니다.
}
```



## Companion object

> Support to declare static member.
>
> When java use kotlin class, static member is declared in Companion object. 

`````kotlin
//CompanionTest.kt
class CompanionTest{
    companion object {
        @JvmStatic fun test1(){} //this annotation support the java can access like java class access way. how to access : CompanionTest.test1(); 
        fun test2(){} // if you access this method in java code space,
        //how to access : CompanionTest.Companion.test2();
    }
}
`````

`````java
//Main.java
class Main{
    public void main(){
        CompanionTest.test1(); //O
        //CompanionTest.test2(); //X Error.
        CompanionTest.Companion.test2(); //O
        
    }
}
`````



