<h1>Draw dragShadow from layout resource</h1>

>  case : using layout resource file.

<h3>Make view and measure, layout.

`````java
//Inflate shadowView from layout resource file.
//No displayed. Has no size.
//If you insert true in 3rd param and ViewGroup in 2nd param, this view will be attaching immediately.
View shadowView = layoutInflater.inflate(R.layout.item_shadow, null, false);
//<code> View.MeasureSpec.UNSPECIFIED </code> is one of the measure mode(=we has no value of exactly measured).
//It cause to measure the shadowView size.
//So, static layout dimens are ignored.
shadowView.measure(View.MeasureSpec.UNSPECIFIED, View.MeasureSpec.UNSPECIFIED);
//Position of left,top,right,bottom relative to parent.
//Assign a size and position to a view and all of its descendants.
shadowView.layout(0, 0, width, height);

`````



<h3>And insert the shadowView into View.DragShadowBuilder.class constructor param.

```java
triggerView.startDragAndDrop(ClipData.newPlainText("", ""), new MyDragShadow(shadowView), null, 0);
```