<h1> Dependency injection</h1>

> 의존관계에 있는 객체를 외부에서 생성해서 인자로 받아서 사용하고자 하는 것. 
>
> 테스트 용이. 
>
> 모듈 단위로 객체 관리.



안드로이드 프레임워크 특성상 Activity 등 특정 시점에 자동으로 생성되는 것들이 있다.

이러한 객체들은 생성자 파라미터로 의존 객체를 받을 수 없으므로 이것을 가능하게 해주는 라이브러리들이 Dagger / Hilt.



<h3>-Dagger</h3>

> 단점 : 
>
> Application 객체 안에 컴포넌트를 만들어야 하는 등 사전에 해야할 작업이 다소 많음.
>
> Jetpack 라이브러리와 같이 사용하려다 보면 코드가 복잡해진다. 



<h3>-Hilt</h3>

> Dagger를 쉽게 사용할 수 있도록 개선된 것이 Hilt.

`````ko
@HiltAndroidApp
class App : Application()

//Application 준비
`````

`````ko
@Singleton
class SampleRepository @Inject constructor()

//주입할 객체
`````

`````ko
@AndroidEntryPoint
class MainActivity : AppCompatActivity(){
	@Inject
	lateInit var repository : SampleRepositiry

...
}

//주입 객체를 사용하는 곳. 
`````

