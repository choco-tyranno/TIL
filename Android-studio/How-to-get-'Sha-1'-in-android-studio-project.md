# How to get 'Sha-1' in android studio project

> ## 3가지 방법 : 
>
> 1. keytool command 이용.
>
> 2. Gradle 뷰 -> Tasks -> android -> signingReport실행.
>    ->Gradle뷰에서 Tasks가 안보이는 경우:
>
>    File - Settings -> Experimental 에서 'Do not build Gradle task list during Gradle sync' 체크 해제.
>
>    File -> Sync Project with Gradle Files 실행.
>    Happy..
>
> 3. Ctrl 두번 단축키로 Run anything 다이얼로그를 띄운다음 'gradle signingReport' 입력.