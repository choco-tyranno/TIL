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

// 주생성자에서는 constructor 키워드 생략가능.
//생성자 파라미터의 변수 또는 상수는 field화 되어 setter 및 getter가 자동생성됨.
//바디가 필요없으면 바디블록도 생략가능.
//바디 안에 constructor 키워드를 달아 보조생성자를 만들 수 있으나, 주생성자가 있다면 주생설자를 호출해줘야함.

class TestClass2(val a1: Int, val a2:Int){
    constructor : this(100, 200){
    }
}
//보조 생성자를 통하는 경우 보조생성자가 먼저 호출되고 주생성자를 먼저 실행한뒤 보조생성자의 블록이 실행된다.
`````













