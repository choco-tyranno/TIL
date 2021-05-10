<h1>Drag Shadow</h1>


<p>start drag and drop operation : 
</p>

`````java
//myView is the tool of drag event start by long clicked. 
onLongClick(View myView){
    //ClipData : transferred data by the drag and drop operation.
    //ShadowBuilder : making shadow used for dragging image.
    //myLocalState : transffered object. for example, myView can be passing on it. this is optional the nullable. note - how to get : <code> dragEvent.getLocalState(); </code>
   	//flag : optional. default <code> 0 </code>;
	myView.startDragAndDrop(ClipData data,ShadowBuilder shadowBuilder
                            ,Object myLocalState,int flag);   
}

`````



<p> prepare drag shadow builder using View.DragShadowBuilder : </p>

> note : extends is optional in case by case. if you just wanna create drag shadow clone with start tool view (in this case : <code>myView</code>), extends is not always needed.

`````java
public class CardShadow extends View.DragShadowBuilder {
    private final float textStrokeWidth;
    private final float boundStrokeWidth;
    private final int rectWidth;
    private final int rectHeight;
    private final String text;
    private final Paint textPaint, boundPaint;
    private final Rect textRect;
    private final float ascent, descent;
    private final float xRadius, yRadius;

    // prepare Paints of drag shadow.
    public CardShadow(View v) {
        super(v);
        this.textStrokeWidth = v.getResources().getInteger(R.integer.createCardFab_shadow_textStrokeWidth);
        this.boundStrokeWidth = v.getResources().getInteger(R.integer.createCardFab_shadow_boundStrokeWidth);
        this.rectWidth = v.getResources().getInteger(R.integer.createCardFab_shadow_boundRectWidth);
        this.rectHeight = v.getResources().getInteger(R.integer.createCardFab_shadow_boundRectHeight);
        this.xRadius = v.getResources().getInteger(R.integer.createCardFab_shadow_xRadius);
        this.yRadius = v.getResources().getInteger(R.integer.createCardFab_shadow_yRadius);
        this.boundPaint = new Paint();
        boundPaint.setColor(Color.BLUE);
        boundPaint.setStyle(Paint.Style.STROKE);
        boundPaint.setStrokeWidth(boundStrokeWidth);
        this.textPaint = new Paint();
        this.text = v.getResources().getString(R.string.card_shadow_text);
        this.textRect = new Rect();
        textPaint.setTextSize(v.getResources().getInteger(R.integer.createCardFab_shadow_textSize));
        textPaint.getTextBounds(text, 0, text.length(), textRect);
        this.ascent = textPaint.getFontMetrics().ascent;
        this.descent = textPaint.getFontMetrics().descent;
        textPaint.setStyle(Paint.Style.FILL_AND_STROKE);
        textPaint.setStrokeWidth(textStrokeWidth);
        textPaint.setColor(Color.BLACK);
    }

    // <code> outShadowSize.set(int x, int y) </code> is setting of shadow size using x, y coordinates.
    // <code> outShadowTouchPoint.set(int x, int y) </code> is setting of shadow touch point using x, y coordinates.
    @Override
    public void onProvideShadowMetrics(Point outShadowSize, Point outShadowTouchPoint) {
        outShadowSize.set(rectWidth + (int) boundStrokeWidth * 2, rectHeight + (int) boundStrokeWidth * 2);
        outShadowTouchPoint.set(rectWidth/2+(int)boundStrokeWidth, rectHeight+(int)boundStrokeWidth);
    }

    // canvas : object in which to draw the shadow image. the system creates this object based on the dimensions it received from <code> onProvideShadowMetrics(Point, Point) callback </code>
    // <code> onDrawShadow() </code> draws the shadow image.
    @Override
    public void onDrawShadow(Canvas canvas) {
        canvas.drawText(text, (float) (textRect.right - textRect.left) / 2 + boundStrokeWidth,
                (float)(-textRect.top+rectHeight)/2 + boundStrokeWidth, textPaint);
        canvas.drawRoundRect(new RectF(boundStrokeWidth, (float) boundStrokeWidth,
                rectWidth + boundStrokeWidth, rectHeight + boundStrokeWidth), xRadius, yRadius, boundPaint);
    }
`````



