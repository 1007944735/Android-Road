## Android动画 ##
### 动画分类 ###
- 逐帧动画
- 补间动画
- 属性动画

### 一、逐帧动画 ###
将动画拆分成帧的形式，每一帧就是一张图片，然后按续一次播放一组图片，就形成了动画

animation.xml
```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <!--android:oneshot 动画是否执行一次-->
    <item android:drawable="@drawable/image_1" android:duration="1000"/>
    <item android:drawable="@drawable/image_2" android:duration="1000"/>
    <item android:drawable="@drawable/image_3" android:duration="1000"/>
    <item android:drawable="@drawable/image_4" android:duration="1000"/>
    <item android:drawable="@drawable/image_5" android:duration="1000"/>
</animation-list>
```

布局文件
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/animation"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:scaleType="fitCenter"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="开始"
        android:onClick="start"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="暂停"
        android:onClick="stop"/>
</LinearLayout>
```
Activit文件,通过xml设置动画
```
public class TestAnimation extends AppCompatActivity {
    private ImageView animation;
    private AnimationDrawable animationDrawable;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_animation);
        animation = findViewById(R.id.animation);
        //通过逐帧动画的资源文件获取AnimationDrawable对象
        animationDrawable = (AnimationDrawable) getResources().getDrawable(R.drawable.frame_animation);
        animation.setBackground(animationDrawable);

        //直接设置为imageView的background
//        animation.setBackgroundResource(R.drawable.frame_animation);
//        animationDrawable= (AnimationDrawable) animation.getBackground();


    }

    public void start(View view) {
        if (animationDrawable != null && !animationDrawable.isRunning())
            animationDrawable.start();
    }

    public void stop(View view) {
        if (animationDrawable != null && animationDrawable.isRunning())
            animationDrawable.stop();
    }
}
```

也可以通过java设置动画
```
private AnimationDrawable createFrameAnim(){
        AnimationDrawable animationDrawable=new AnimationDrawable();
	animationDrawable.addFrame(getResources().getDrawable(R.drawable.image_1),1000);
	animationDrawable.addFrame(getResources().getDrawable(R.drawable.image_2),1000);
	animationDrawable.addFrame(getResources().getDrawable(R.drawable.image_3),1000);
	animationDrawable.addFrame(getResources().getDrawable(R.drawable.image_4),1000);
	animationDrawable.addFrame(getResources().getDrawable(R.drawable.image_5),1000);

	animationDrawable.setOneShot(false);
	animation.setBackground(animationDrawable);
 	return animationDrawable;
}
```

### 二、补间动画 ###
补间动画作用于view，分为平移，旋转，缩放和透明度

- 平移（translate）
	java代码
	```
	TranslateAnimation animation=new TranslateAnimation(0,200,0,0);
	animation.setDuration(3000);//动画进行时间
	rectangle.startAnimation(animation);
	```

	TranslateAnimation(float fromXDelta, float toXDelta, float fromYDelta, float toYDelta)
	- fromXDelta：在x方向上起始值
	- toXDelta：在x方向上结束值
	- fromYDelta：在y方向上起始值
	- toYDelta：在y方向上结束值

	xml代码
	```
	<translate
        android:duration="3000"
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200"
        android:toYDelta="0"/>
	```
	```
	Animation animation= AnimationUtils.loadAnimation(this,R.anim.animaiton);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```
- 旋转（rotate）
	java代码
	```
    RotateAnimation animation=new RotateAnimation(0,360, Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
    animation.setDuration(3000);//动画进行时间
    rectangle.startAnimation(animation);
	```

	RotateAnimation(float fromDegrees, float toDegrees, int pivotXType, float pivotXValue,int pivotYType, float pivotYValue)
	- fromDegrees：旋转开始时的角度
	- toDegrees：旋转结束时的角度（正为顺时针，负为逆时针）
	- pivotXType：旋转轴点的x坐标模式
	- pivotXValue：旋转轴点的x坐标的相对值
	- pivotYType：旋转轴点的y坐标模式
	- pivotYValue：旋转轴点的y坐标的相对值

	> pivotXType = Animation.ABSOLUTE:旋转轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
	> pivotXType = Animation.RELATIVE_TO_SELF:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
	> pivotXType = Animation.RELATIVE_TO_PARENT:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)

	xml代码
	```
    <rotate
        android:fromDegrees="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="360"
        android:duration="3000"/>
	```
	> pivotX,pivotY可以是百分比，数字。
	```
	Animation animation= AnimationUtils.loadAnimation(this,R.anim.animaiton);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```

- 缩放（scale）
	java代码
	```
	ScaleAnimation animation=new ScaleAnimation(1f,0.5f,1f,0.5f,Animation.RELATIVE_TO_SELF,0.5f,Animation.RELATIVE_TO_SELF,0.5f);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```

	ScaleAnimation(float fromX, float toX, float fromY, float toY,int pivotXType, float pivotXValue, int pivotYType, float pivotYValue)
	- fromX：缩放x方向上的起始缩放倍数
	- toX：缩放x方向上的结束缩放倍数
	- fromY：缩放y方向上的起始缩放倍数
	- toY：缩放y方向上的结束缩放倍数
	- pivotXType：缩放轴点的y坐标模式
	- pivotXValue：缩放轴点的y坐标的相对值
	- pivotYType：缩放轴点的y坐标的相对值
	- pivotYValue：缩放轴点的y坐标的相对值

	> pivotXType = Animation.ABSOLUTE:旋转轴点的x坐标 =  View左上角的原点 在x方向 加上 pivotXValue数值的点(y方向同理)
	> pivotXType = Animation.RELATIVE_TO_SELF:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 自身宽度乘上pivotXValue数值的值(y方向同理)
	> pivotXType = Animation.RELATIVE_TO_PARENT:旋转轴点的x坐标 = View左上角的原点 在x方向 加上 父控件宽度乘上pivotXValue数值的值 (y方向同理)

	xml代码
	```
    <scale
        android:fromXScale="1"
        android:fromYScale="1"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0.5"
        android:toYScale="0.5"
        android:duration="3000"/>
	```
	> toXScale,toYScale可以是百分比，数字。
	```
	Animation animation= AnimationUtils.loadAnimation(this,R.anim.animaiton);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```

- 透明度（alpha）
	java代码
	```
	AlphaAnimation animation=new AlphaAnimation(1f,0f);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```

	AlphaAnimation(float fromAlpha, float toAlpha) 
	- fromAlpha：起始透明度
	- toAlpha：结束透明度

	> 透明度：0~1，0为透明，1为不透明


	xml代码
	```
    <alpha
        android:fromAlpha="1"
        android:toAlpha="0.5"
        android:duration="3000"/>
	```

	```
	Animation animation= AnimationUtils.loadAnimation(this,R.anim.animaiton);
	animation.setDuration(3000);
	rectangle.startAnimation(animation);
	```

- 动画集合

在xml中使用set，在java代码中使用AnimationUtils.loadAnimation加载xml文件

```
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="3000"
    android:startOffset ="1000"
    android:fillBefore = "true"
    android:fillAfter = "false"
    android:fillEnabled= "true"
    android:repeatMode= "restart"
    android:repeatCount = "0"
    >
    <translate
        android:duration="3000"
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200"
        android:toYDelta="0"/>

    <alpha
        android:fromAlpha="1"
        android:toAlpha="0.5"
        android:duration="3000"/>

    <rotate
        android:fromDegrees="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="360"
        android:duration="3000"/>

    <scale
        android:fromXScale="1"
        android:fromYScale="1"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0.5"
        android:toYScale="0.5"
        android:duration="3000"/>

</set>
```
set标签里的属性，所有动画共享
- android:duration：动画持续时间
- android:startOffset：动画延迟开始时间
- android:fillBefore：动画结束后，是否停留在动画开始状态，默认为true
- android:fillAfter：动画结束后，是否停留在动画结束状态，默认为false
- android:fillEnabled：是否应用fillbefore，对fillAdter无影响，
- android:repeatMode：选择重复播放模式
- android:repeatCount：重复播放次数，-1为一直播放

>  repeatMode:restart和reverse；restart：正序重放，reverse：逆序重放

- 插值器（interpolator）


## 三、属性动画 ##
属性动画作用于任何java对象，可以实现各种动画效果，属性动画通过不断对值进行改变，并不断将该值赋给对象的属性，从而实现在该属性上的动画效果，属性动画有两个常用的类，ValueAnimator类和ObjectAnimator类。

- ValueAnimator
	该类有三个方法
        ValueAnimator.ofArgb();
        ValueAnimator.ofInt();
        ValueAnimator.ofFloat();



