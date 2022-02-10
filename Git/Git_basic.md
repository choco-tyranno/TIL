# Git basic

* 협업시 Commit이 누구의 것인지 기록하는 것은 매우 중요.

  -> Git저장소에 커밋시 기록할 사용자 이름과 이메일 설정 요구됨.

  

### 로컬에서 기본 값으로 사용할 Git 사용자 이름과 이메일 설정(Global option)

***

* global option 사용 시, 현재 시스템의 모든 Git 작업에 대해 적용됨.

```shell
$git config --global user.name "<USER_NAME>"
$git config --global user.email <USER_EMAIL>
```

```shell
$cat ~/.gitconfig
// ~/.gitconfig에 추가된 내용 확인.
```



### 저장소 별로 전용 사용자 설정하기

***

* global 옵션으로 설정한 정보보다 우선 적용됨.

```shell
$git config user.name "<USER_NAME>"
$git config user.email <USER_EMAIL>
```



### 현재 저장소의 user name, email 확인하기

***

```shell
$git config user.name
$git config user.email
```



### Git 사용자 이름, 이메일 정보 삭제하기

***

```shell
$git config <--global> --unset user.name
$git config <--global> --unset user.email
// 전역 설정 삭제 또는 개별 저장소 설정 삭제.
```



### 커밋 정보 출력

```shell
$git show <COMMIT_ID>
// commit id, author info(user name, user email), commit date, message가 출력됨.
```







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





## Commit 되돌리기 관련

***

### Reset

+ HEAD포인터를 완전히 옮겨버림.

```shell
$git reset --hard <COMMIT_ID>

// HEAD 포인터를 해당 커밋 위치로 변경.
// --hard옵션은 Untracked file들을 제외하고 HEAD에 추가된 변경사항들을 되돌림.
// Untracked file도 없애려는 경우에는 $git clean 명령어 이용.

//$git reset --hard <HEAD^, HEAD@{1}, HEAD~1> 같은 명령으로 바로 이전 커밋으로 되돌릴 수 있지만,
//⚠️주의가 필요하기 때문에 reflog 기능에 대한 선행학습 필요.
```

### Clean

```shell
$git clean -n
// Untracked files 확인

$git clean -f
// Untracked fils 삭제
```

### Revert

+ 커밋 내용을 되돌리는 새로운 커밋을 생성.
+ ⚠️원격저장소에 push한 내용을 변경하려는 경우, `force push` 해야하지만, 이는 `협업시에 절대 금기`(원격저장소에 이를 막아놓는 것이 일반적).
+ 이러한 상황에서 사용할 수 있는 것이 Revert

```shell
$git revert <COMMIT_ID>	//에디터 실행
$git revert <COMMIT_ID> -e //에디터 실행
$git revert <COMMIT_ID> --edit //에디터 실행

$git revert HEAD // HEAD가 항상 바로직전 커밋을 가르키는 것은 아니므로 명시적 ID를 입력하는 것이 낫다.
$git revert --no-edit <COMMIT_ID> // 새롭게 생성된 revert commit의 커밋 메시지가 자동으로 생성됨.(Revert "<해당 커밋의 커밋 메시지>" 혈식)

$git revert --no-edit <COMMIT_ID1> <COMMIT_ID2> // 여러개의 커밋을 한꺼번에 되돌리기. Revert커밋1이 먼저 생성되고 2가 나중에 생성됨.
$git revert -n <COMMIT_ID1> <COMMIT_ID2> // -n 옵션을 붙이면 두개의 커밋을 되돌린 내용이 index에 추가되지만 커밋은 만들어지지 않은 상태가됨.
// 필요에 따라 이 상태에서 커밋에 필요한 추가적인 변경 작업을 하고 $git revert --continue를 실행하면 커밋이 진행됨. 에디터 실행. 
```

+ patch 파일로 커밋 내용을 저장하고 수동으로 revert 할 수도 있음.

```shell
// Merge commit을 되돌리려면 $git show <MERGE_COMMIT_ID>를 통해 Merge된 commit을 확인하고 -m 옵션과 함께 그 확인된 인덱스를 이용해 되돌림.
$git revert -m <인덱스(1부터 시작)> <MERGE_COMMIT_ID> 
```

### Amend

+ 커밋의 내용을 덮어씀.
+ 수정할 내용을 스테이징하고 `$git commit --amend` 명령어 실행.
+ 스테이지에 추가된 내용을 반영하면서 커밋 메시지도 변경가능.
  -> 커밋 메시지 변경할 때 사용.





## Commit 없이 임시로 변경사항 저장

***

### Stash

> 용도 :  예컨대 코드 수정 중 Hotfix 요청이 들어오는 경우 사용.
>
> 1. 변경사항 전체를 하드 리셋하거나,
> 2. 저장소를 하나 더 클론받아오거나,
> 3. 급하게 전체 커밋하거나
> 4. 하는 수고를 덜 수 있음.

```shell
$git stash
//$git stash save의 줄임 표현.
//-m을 붙여 메시지를 붙일 수 있음.
//붙이지 않으면 현재상태를 나타내는 메시지가 자동생성됨.
//실행 후 워킹트리가 클린한 상태가 됨.
//--index 옵션을 붙이면 스테이지 상태까지 같이 복원됨.

$git stash list
// stash된 리스트를 보여줌

$git stash show <STASH_NAME>
// stash된 내용 확인 가능.

$git stash pop
// stash 스텍에서 꺼내어 복구함(apply & drop).

$git stash apply
// stash 스텍에서 복사하여 적용하고, 꺼내어 없애진 않음.

$git stash drop
// stash 스텍에서 제거함

$git stash branch <BRANCH_NAME>
//새로운 브랜치를 만들고 pop을 실행.

$git stash clear
//모든 stash를 삭제(stack 비우기).

$git stash show -p | git apply --reverse
//가장 최근 apply된 stash를 다시 되돌림.
//⚠️apply한 stash를 기반으로 되돌리기 때문에, drop이 실행된 경우에는 사용할 수 없음.

$git stash show -p <STASH_NAME> | git apply --reverse
//apply된 특정 stash를 되돌림.
//⚠️apply한 stash를 기반으로 되돌리기 때문에, drop이 실행된 경우에는 사용할 수 없음.
```

> stash는 사용시 변경사항은 Stack자료 구조로 저장되기 때문에 LIFO를 따름.
>
> stash@{0}와 같이 인덱스가 있는 임시 이름이 생성됨. 

```shell
//기타

$git checkout -f
//현재 HEAD의 모든 변경사항을 되돌려야 할 때.

$git stash -k
//-k는 --keep-index의 줄임표현.
//스테이징된 파일을 제외하고 stash함.

$git stash -u
//-u는 --include-untracked의 줄임표현.
//untracked file들도 함께 stash해줌.
```









