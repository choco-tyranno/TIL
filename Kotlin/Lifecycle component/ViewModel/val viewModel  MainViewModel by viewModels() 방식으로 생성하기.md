<h1>val viewModel : MainViewModel by viewModels() 방식으로 생성하기
    
</h1>



+ viewModels()에 빨간줄이 뜬다면 Project structure > Modules >Source Compatibility와 Target Compatibility 가 Java Version 1.8이상으로 되어있는지 확인.
+ Kotlin의 경우 Java 1.8을 적용시키기 위해선 추가적으로 build.gradle의 kotlinOptions{ jvmTarget = "1.8"}이 필요.