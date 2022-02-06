# Git basic

+ Git bash의 기본 텍스트 편집기는 Vi 에디터.
+ Vi는 Command mode / Edit mode / Last line mode 세가지 모드가 있음.

```shell
$git commit

// Vi 켜기
```

+ Vi 편집기가 켜지면 i, a, o 중에 하나의 키를 눌러 edit mode로 전환.
+ 커밋 메시지를 작성하고 `esc`키를 눌러 edit mode에서 빠져나옴.
+ 커밋 메시지 입력 화면에 #으로 시작하는 줄은 주석처리되어 해당 줄은 커밋메시지로 저장되지 않음.
+ `:wq` 명령으로 커밋 메시지를 저장하고 종료.
+ 또는 `:q!`명령으로 저장하지 않고 나가기. 

