## View的测量、布局及绘制原理 ##
### 一、View绘制的总体流程 ###

![](https://i.imgur.com/1PY1vga.png)

view的绘制是顺序是按照DecorView-->ViewGroup-->View一层层地绘制下来，依次经历测量（measure），布局（layout）和绘制（draw）


### 二、Measure流程 ###

measure是测量每个控件的大小

调用measure方法，进行一些逻辑的处理，再调用onMearsure方法（需重写），在其中调用setMeasuredDimension方法设定view的宽高信息，最终完成view的测量

```
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
}
```

measure方法中会传入widthMeasureSpec和heightMeasureSpec两个参数，表示view的宽高信息

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
	getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

在onMeasure()中调用setMeasureDimense方法，设置view的宽高

#### MeasureSpec的确定 ####
MeasureSpec是View中的内部类，是32位的int类型。MeasureSpec由两部分组成，一部分是测量模式（高两位，即31位，30位），另一部分是尺寸大小（低30位）

![](https://i.imgur.com/dCXEt3m.png)

可以从MeasuSpec中获取相应的测量模式和尺寸大小

```
//获取宽的测量模式
int widthMode=MeasureSpec.getMode(widthMeasureSpec);
//获取宽的大小
int widthSize=MeasureSpec.getSize(widthMeasureSpec);
//获取高的测量模式
int heightMode=MeasureSpec.getMode(heightMeasureSpec);
//获取高的大小
int heightSize=MeasureSpec.getSize(heightMeasureSpec);
```

**mode有三种模式**

- UNSPECIFIED ：不对view进行限制，要多大给多大
- EXACTLY：对应LayoutParam的match_parent和具体数值。view的最终大小就是SpecSize所指定的值
- AT_MOST：对应layoutPa的wrap_content。view的大小不超过父容器的大小

**MeasureSpec是如何确定的**
顶层容器DecorView是根据手机屏幕的大小和自身的LayoutParam决定的，而其他的view（包括ViewGroup）是根据自身的LayoutParam和父容器的MeasureSpace决定的，具体的计算规则如下表

![](https://i.imgur.com/Z6PU2eu.png)

当子view的LayoutParam的布局格式是具体大小时，无论父容器是什么测量模式，子view的大小都是其本身，且是EXACTLY；
当子View是match_parent时，则模式与父容器相同，且大小等同于父容器剩余大小；
当子View是wrap_content时，则模式是AT_MOST,大小等同于父容器剩余大小，不超过父容器大小；
**当子View是match_parent或wrap_content时，子View大小没有区别，所以在自定义View时，会重写onMeasure（）处理wrap_content情况，比如直接设置一个具体的值**

```
if(widthMode==MeasureSpec.AT_MOST&&heightMode==MeasureSpec.EXACTLY){
	setMeasuredDimension(200,heightMeasureSpec);
}else if(widthMode==MeasureSpec.EXACTLY&&heightMode==MeasureSpec.AT_MOST){
	setMeasuredDimension(widthMeasureSpec,200);
}if(widthMode==MeasureSpec.AT_MOST&&heightMode==MeasureSpec.AT_MOST){
	setMeasuredDimension(200,200);
}else {
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```
![](https://i.imgur.com/lRWpLNg.png)	

### 二、Layout流程 ###

layout是设置子View在父容器中的位置，主要通过上下左右四个点来确定的。ViewGroup先确定自身的布局，然后再onLayout()中调用子View的layout()方法，确定子view在父容器中的布局

 > 使用view.layout(int left, int top, int right, int bottom)设置子view的布局
 > left:表示view的左边界距离父容器左边的距离
 > top：表示view的上边界距离父容器左边的距离
 > right：表示view的右边界距离父容器上边的距离
 > bottom：表示view的下边界距离父容器上边的距离

![](https://i.imgur.com/MdCKHrl.png)

### 三、绘制过程 ###

view是通过onDraw方法绘制在屏幕上的，view的一般绘制遵循如下几步

1. 绘制背景
2. 绘制自己
3. 绘制children
4. 绘制装饰

![](https://i.imgur.com/SAka17w.png)

### 四、自定义View ###

```
public class CircleView extends View {
    private int circleColor;//圆颜色
    private int circleRadius;//圆半径
    private final int DEFAULT_RADIUS = 50;//默认圆半径

    private Paint mPaint;

    public CircleView(Context context) {
        this(context, null);
    }

    public CircleView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray attr = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        //获取自定义属性
        circleColor = attr.getColor(R.styleable.CircleView_circle_color, getResources().getColor(R.color.colorPrimary));
        circleRadius = attr.getInt(R.styleable.CircleView_circle_radius, DEFAULT_RADIUS);
        attr.recycle();

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //获取宽的测量模式
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        //获取宽的大小
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        //获取高的测量模式
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        //获取高的大小
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        int widthPadding=getPaddingLeft()+getPaddingRight();
        int heightPadding=getPaddingTop()+getPaddingBottom();
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.EXACTLY) {
            setMeasuredDimension(circleRadius * 2+widthPadding, heightSize);
        } else if (widthMode == MeasureSpec.EXACTLY && heightMode == MeasureSpec.AT_MOST) {
            setMeasuredDimension(widthSize, circleRadius * 2+heightPadding);
        } else {
            setMeasuredDimension(circleRadius * 2+widthPadding, circleRadius * 2+heightPadding);
        }

    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int leftPadding=getPaddingLeft();
        int topPadding=getPaddingTop();
        int rightPadding=getPaddingRight();
        int bottomPadding=getPaddingBottom();
        //内边距
        int cx=leftPadding+(getWidth()-leftPadding-rightPadding)/2;
        int cy=topPadding+(getHeight()-topPadding-bottomPadding)/2;
        mPaint.setColor(circleColor);
        //绘制圆
        canvas.drawCircle(cx, cy, circleRadius, mPaint);
    }
}
```

```
<declare-styleable name="CircleView">
	<attr name="circle_color" format="color"/>
	<attr name="circle_radius" format="integer"/>
</declare-styleable>
```

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical">
    <com.sgevf.myapplication.CircleView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@color/colorAccent"
        app:circle_color="@color/colorPrimary"
        android:paddingTop="10dp"
        android:paddingLeft="20dp"/>
</LinearLayout>
```

![](https://i.imgur.com/9BsQn5w.png)