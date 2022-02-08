# DiffUtil

+ 기존 리스트와 업데이트될 리스트, 두 리스트의 차이를 계산하고, 
  첫 번째 리스트를 두 번째 리스트로 변환하는 업데이트 리스트를 출력하는 유틸 클래스.
+ RecyclerView adapter 업데이트를 위한 계산에 사용할 수 있음.
+ BackgroundThread에서 단순화할 수 있는 `ListAdapter`와 `AsyncListDiffer` 참조.
+ Eugene W. Myers의 차이 알고리즘을 이용해 한 리스트를 다른 리스트로 변환하기 위한 최소 업데이트 수를 계산함.
+ O(N) 공간 복잡도로 두 리스트 간 최소 추가 및 삭제 작업 수를 찾음.
+ D는 Edit script의 길이인 O(N+D^2) 예상 시간 복잡도를 갖음.
+ 이동된 항목을 감지.
+ DiffUtil, ListAdapter, AsyncListDiffer는 List가 Immutable할 것이 요구됨. -> 요소 직접 수정 X.
+ 대신 콘텐츠가 변경될 때마다 새로운 리스트가 제공되어야함.(List 인스턴스를 교체)
+ 백그라운드 스레드에서 계산을 진행하고  DiffUtil.DiffResult를 가져온 뒤 메인 스레드에서 RecyclerView에 적용할 것이 권장됨.
+ 이동 감지가 활성화되면 O(MN)의 시간복잡도가 추가됩니다.(M : 추가된 항목 수, N : 제거된 항목 수)
+ 이미 리스트가 같은 제약 조건에 따라 정렬되어 있는 경우(예 : 게시물과 그 타임스탬프), 성능을 위해 이동 감지를 비활성화할 수도 있음.
+ 실제 알고리즘 실행 시간은 리스트의 변경되는 수와 비교 메서드의 비용에 따라 크게 달라짐.

> - 100 items and 10 modifications: avg: 0.39 ms, median: 0.35 ms
> - 100 items and 100 modifications: 3.82 ms, median: 3.75 ms
> - 100 items and 100 modifications without moves: 2.09 ms, median: 2.06 ms
> - 1000 items and 50 modifications: avg: 4.67 ms, median: 4.59 ms
> - 1000 items and 50 modifications without moves: avg: 3.59 ms, median: 3.50 ms
> - 1000 items and 200 modifications: 27.07 ms, median: 26.92 ms
> - 1000 items and 200 modifications without moves: 13.54 ms, median: 13.36 ms
>
> -> 구현 제약으로 인해 리스트의 최대 크기는 2^26일 수 있다.



## Methods

***

+ getOldListSize : 이전 리스트 사이즈 반환.
+ getNewListSize : 새로운 리스트 사이즈 반환.
+ areItemsTheSame(oldPosition : Int, newPosition : Int) : 두 객체가 같은 것인지 확인.
+ areContentsTheSame(oldPosition : Int, newPosition : Int) : 두 객체의 내용이 같은지를 확인. areItemsTheSame이 true를 반환하는 경우에 실행됨.
+ getChangePayload(oldPosition : Int, newPosition : Int) :  areItemsThemSame이 true를 areContentsTheSame이 false를 반환하는 경우에 실행됨. 변경 내용에 대한 페이로드를 가져옴.



## 사용법

***

+ Item 객체에 대한 DiffUtil.Callback의 구현체를 만듬.

```kotlin
package com.choco_tyranno.tictoccroc_metrosearch.presentation.ui

import androidx.recyclerview.widget.DiffUtil
import com.choco_tyranno.tictoccroc_metrosearch.domain.model.SubwayStationWithLines

class SubwayStationWithLinesDiffCallback(
    private val oldItemList : List<SubwayStationWithLines>,
    private val newItemList : List<SubwayStationWithLines>
) : DiffUtil.Callback() {

    override fun getOldListSize(): Int {
        return oldItemList.size
    }

    override fun getNewListSize(): Int {
        return newItemList.size
    }

    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldItemList[oldItemPosition].subwayStation.id == newItemList[newItemPosition].subwayStation.id
    }

    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldItemList[oldItemPosition] == newItemList[newItemPosition]
    }

    // getChangePayload() 메서드에서 반환되는 객체는 DiffResult에서 Adapter의 notifyItemRangeChanged(pos,count,payload)를 호출,
	// onBindViewHolder(... payloads : List) 메서드가 호출되며 리스트가 업데이트 됨.
	// 여기서 리스트는 업데이트가 필요한 아이템만 호출됨.
    override fun getChangePayload(oldItemPosition: Int, newItemPosition: Int): Any? {
        return super.getChangePayload(oldItemPosition, newItemPosition)
    }
}
```



RecyclerView.Adapter 구현체 update 메서드에 DiffUtil적용

```kotlin
class SearchResultAdapter constructor(private val viewModel: SearchViewModel) :
    RecyclerView.Adapter<SearchResultViewHolder>() {
    private var itemList : List<SubwayStationWithLines> = emptyList()

	...
        
    fun updateItemList(itemList : List<SubwayStationWithLines>){
        val diffCallback = SubwayStationWithLinesDiffCallback(this.itemList, itemList)
        val diffResult = DiffUtil.calculateDiff(diffCallback)
        this.itemList = itemList
        diffResult.dispatchUpdatesTo(this)
        //diff계산에서 반환된 DiffResult 반환 객체가 변경사항을 Adapter에 전달하고, Adapter가 변경사항에 대해 알림을 받음.
    }

}
```

