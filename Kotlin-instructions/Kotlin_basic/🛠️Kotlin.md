# ๐ ๏ธKotlin

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

> ์์ฑ์ : ํด๋์ค๋ฅผ ํตํด ๊ฐ์ฒด๋ฅผ ์์ฑํ  ๋ ์๋์ผ๋ก ์ํ๋  ์ฝ๋๋ฅผ ์์ฑํ๋ ๊ณณ.
>
> ๋ฉ์๋์ ๋น์ทํด ๋ณด์ด์ง๋ง ๋ฐํํ์์ด ์์ด ๋ฉ์๋๋ผ ๋ถ๋ฅด์ง ์์.
>
> ํด๋์ค์ ๋ณ์ ๊ฐ ์ด๊ธฐํ์ ์ฃผ๋ก ์ด์ฉ๋จ.

`````kotlin
class TestClass constructor(val a1 :Int,var a2 :Int)

//์ฃผ์์ฑ์์์๋ constructor ํค์๋ ์๋ต๊ฐ๋ฅ.
//๋ฐ๋๊ฐ ํ์์์ผ๋ฉด ๋ฐ๋๋ธ๋ก๋ ์๋ต๊ฐ๋ฅ.
//๋ฐ๋ ์์ constructor ํค์๋๋ฅผ ๋ฌ์ ๋ณด์กฐ์์ฑ์๋ฅผ ๋ง๋ค ์ ์์ผ๋, ์ฃผ์์ฑ์๊ฐ ์๋ค๋ฉด ์ฃผ์์ค์๋ฅผ ํธ์ถํด์ค์ผํจ.

class TestClass2(val a1: Int, val a2:Int){
    constructor : this(100, 200){
    }
}
//๋ณด์กฐ ์์ฑ์๋ฅผ ํตํ๋ ๊ฒฝ์ฐ ๋ณด์กฐ์์ฑ์๊ฐ ๋จผ์  ํธ์ถ๋๊ณ  ์ฃผ์์ฑ์๋ฅผ ๋จผ์  ์คํํ๋ค ๋ณด์กฐ์์ฑ์์ ๋ธ๋ก์ด ์คํ๋๋ค.
`````

### init ์ฝ๋ ๋ธ๋ก :

> ๊ฐ์ฒด ์์ฑ์ ์๋์ผ๋ก ์ฒ๋ฆฌ๋๋ ์ฝ๋๋ฅผ ๋ง๋ค ์ ์์.
>
> ๊ธฐ๋ณธ ํธ์ถ ์์ : main constructor -> init
>
> sub constructor๋ฅผ ํตํ ๊ฐ์ฒด ์์ฑ์ ํธ์ถ ์์ : main constructor -> init -> subconstructor



## Access modifier

> ### Class :
>
> private : ์ธ๋ถ์์ ๊ฐ์ฒด ์์ฑ ๋ถ๊ฐ.
>
> public(๊ธฐ๋ณธ) : ์ธ๋ถ์์ ๊ฐ์ฒด๋ฅผ ์์ฑํ  ์ ์๋ค.
>
> ~~protected~~
>
> internal : ๊ฐ์ ๋ชจ๋์ธ ๊ฒฝ์ฐ์๋ง ๊ฐ์ฒด๋ฅผ ์์ฑํ  ์ ์๋ค.

> ### variable, method :
>
> private : ์ธ๋ถ์์ ์ ๊ทผ ๋ถ๊ฐ.
>
> public(๊ธฐ๋ณธ) : ์ธ๋ถ์์ ์ ๊ทผ์ด ์์ ๋ก์. ๋ณ์์ ๊ฒฝ์ฐ getter์ setter ์๋ ์์ฑ๋จ.
>
> protected : ์์๊ด๊ณ์์๋ง ์ ๊ทผ ๊ฐ๋ฅ.
>
> internal : ๊ฐ์ ๋ชจ๋์ธ ๊ฒฝ์ฐ์๋ง ์ ๊ทผ ๊ฐ๋ฅ.

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

> var๋ก ์ ์ธ๋ ๋ณ์์ ์ด๊ธฐํ๋ฅผ ๋ค๋ก ๋ฏธ๋ฃฐ ์ ์๋ค.
>
> ๋ณ์์ ๊ฐ์ ์ฌ์ฉํ๊ธฐ ์ ์ ๋ฐ๋์ ์ด๊ธฐํ๊ฐ ์ด๋ค์ ธ์ผ ํ๋ค.
> ->'UninitializedPropertyAccessException' ๋ฐ์.
>
> val๋ก ์ ์ธ๋ ๋ณ์๋ ์ค๋ฅ๊ฐ ๋ฐ์ํ๋ค.

> ํ๋กํผํฐ์ ๊ฐ์ ๋์ค์ ์ํํ  ๋์๋ ์ง์ฐ์ด๊ธฐํ๋ฅผ ์ฌ์ฉํ๋ค.
>
> var ๋ณ์๋ lateinit์ ์ฌ์ฉ.
>
> val ๋ณ์๋ lazy ์ฝ๋๋ธ๋ก ์ฌ์ฉ.

`note :` lateinit var๋ก ์ ์ธ๋ ๋ณ์๋ always not null. ๋ด๋ถ์ ์ผ๋ก Function<>์ด ๋ด์ฅ๋์ด์๋ค. 

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
//๋ฆฌํ๋ ์์ ์ด์ฉํ ์ด๊ธฐํ ์ฒดํฌ ๋ฐฉ๋ฒ
fun testMethod(){
    if(::a2.isInitialized == false){
        a2 = "ํ์คํธ"
    }
    println("a2 : $a2")
}
`````

`````kotlin
//lazy ์ฝ๋๋ธ๋ก์ ์ด์ฉํ ์ง์ฐ ์ด๊ธฐํ
//์ฌ์ฉํ  ๋ ๊ฐ์ ์ด๊ธฐํํ๋ค๋ ์๋ฏธ O
//๋์ค์ ํ๋กํผํฐ์ ๊ฐ์ ์ํํ๋ค๋ ์๋ฏธ X
//๋ฉ๋ชจ๋ฆฌ ์ ์ฝ์ ์ด์ 

val a3 : String by lazy{
    "ํ์คํธ"
}

`````



## Overriding

> + Overriding : ๋ถ๋ชจ ํด๋์ค๊ฐ ๊ฐ์ง ๋ฉ์๋๋ฅผ ์์ ํด๋์ค์์ ์ฌ์ ์ํ๋ ๊ฐ๋.
> + ๋ถ๋ชจ๊ฐ ๊ฐ์ง ๋ฉ์๋์ ์ด๋ฆ, ๋งค๊ฐ๋ณ์ ํํ ๋ชจ๋ ๋์ผํด์ผ ํ๋ค.
> + ๋ชจ๋  ๊ฐ์ฒด๋ ๋ถ๋ชจ ํด๋์คํ ์ฐธ์กฐ ๋ณ์์ ๋ด์ ์ ์๋ค.
> + ๋ถ๋ชจ ํด๋์คํ ์ฐธ์กฐ๋ณ์๋ฅผ ์ฌ์ฉํ๋ฉด ๋ถ๋ชจ ํด๋์ค์ ์ ์๋์ด ์๋ ๋ฉค๋ฒ๋ง ์ฌ์ฉ์ด ๊ฐ๋ฅํ๋ค.
> + ๋ถ๋ชจํ ์ฐธ์กฐ๋ณ์๋ฅผ ํตํด Overriding๋ ์์ ๋ฉ์๋๋ฅผ ํธ์ถํ  ์ ์๋ค.
> + ์์ ํด๋์ค ์์ญ์์ superํค์๋๋ฅผ ํตํด ๋ถ๋ชจ์ ๋ฉ์๋๋ฅผ ํธ์ถํ  ์ ์๋ค. 

```kotlin
open class SuperClass{
    open fun superMethod(){
        println("superMethod์๋๋ค.")
    }
}
class SubClass : SuperClass(){
    override fun superMethod(){
        println("subMethod์๋๋ค.")
    }
}

fun main{
    val obj1 : SuperClass = SubClass()
    obj1.superMethod()
    //output: subMethod์๋๋ค.
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



## Generics: in, out, where

> ### `out` modifier :
>
> ์ฝํ๋ฆฐ์์ out ํค์๋๋ ์ปดํ์ผ๋ฌ์  ์ ๋ค๋ฆญ์ ํ์ 'T' ๊ฐ ๋จ์ง Source<T>์ ๋ฉค๋ฒ๋ค๋ก๋ถํฐ ๋ฆฌํด๋ ๋ฟ(produced)์ด๋ผ๋ ๊ฒ(์๋น๋์ง ์์)์ ๋ชํํ๊ฒ ์๋ฆฌ๋ ์ค๋ช์ ํค์๋.

```ko
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

>### `in` modifier :
>
>in ํค์๋๋ ๋จ์ง ์๋น๋ ๋ฟ ์์ฐ๋์ง ์๋ ๊ฒ์ ์๋ฆฐ๋ค.

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, you can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

> ### `where` clause :
>
> where์ ์ ํ๋์ ๋งค๊ฐ๋ณ์ 'T' ๊ฐ ๋ ์ด์์ ํ์์์ ๋ํ๋ผ๋ ์ฌ์ฉ.

```kotl
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```



## ์์ฃผ์ฐ๋ Annotation

> + @JvmName 
> + @JvmStatic
> + @JvmField
> + @Throws
> + @JvmOverloads

```kotlin
@JvmName("testListString")
fun test(a : List<String>)

@JvmName("testListInt")
fun test(a : List<Int>)
//Generic์ ๋ค์ด์๋ชฌ๋ ์ฐ์ฐ์ ๋ด๋ถ ํ์์ ์ปดํ์ผ ํ์์ ์ง์์ง๊ธฐ ๋๋ฌธ์ @JvmName์ผ๋ก ๊ตฌ๋ถํด์ค๋ค.
```

```kotlin
class Test {
    companion object {
        @JvmStatic var count : Int = 0 //Companion ๊ฐ์ฒด ์์ด ์๋ ์ง์  Test ํด๋์ค์ getter/setter๋ฅผ ๋ง๋ค์ด์ค.
    }
}
```

```kotlin
class Test {
    @JvmField
    var count = 0 // getter/setter ์์ฑ๋์ง ์์.
}
```

```kotlin
void convertStringToInt(String str) throws NumberFormatException {...} // java

@Throws(NumberFormatException::class)
fun convertStringToInt(str: String){...} // kotlin
```

```kotlin
class Test @JvmOverloads constructor(
        var arg1: String, var arg2: Int = 0, var arg3: Int = 0)
// java๋ default argument๊ฐ ์๊ธฐ ๋๋ฌธ์ @JvmOverloads๋ฅผ ํตํด ์์ฑ์ ์ค๋ฒ๋ก๋ฉ์ ๋ง๋ค์ด์ค๋ค.
```

