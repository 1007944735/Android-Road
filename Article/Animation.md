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

- **平移（translate）**
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
- **旋转（rotate）**
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

- **缩放（scale）**
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

- **透明度（alpha）**
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

- **动画集合**

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


## 三、属性动画 ##
属性动画作用于任何java对象，可以实现各种动画效果，属性动画通过不断对值进行改变，并不断将该值赋给对象的属性，从而实现在该属性上的动画效果，属性动画有两个常用的类，ValueAnimator类和ObjectAnimator类。

- ValueAnimator
	该类有三个方法
        ValueAnimator.ofArgb(int values)
        ValueAnimator.ofInt(float values)
        ValueAnimator.ofFloat(int values)
		ValueAnimator.ofObject(int values)

具体使用
```
ValueAnimator animator=ValueAnimator.ofInt(100,200);
animator.setDuration(5000);
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	@Override
	public void onAnimationUpdate(ValueAnimator animation) {
		int value= (int) animation.getAnimatedValue();
		rectangle.setText(value+"");
		Log.d("TAG", "onAnimationUpdate: "+value);
	}
});
animator.start();
```

ValueAnimator.ofInt()可以传入多个值，动画开始时，将进行平滑过渡，比如传入10，20，50，会先从10平滑到20再从20平滑到50。addUpdateListener是监听值的变化，通过animation.getAnimatedValue()获取，还有addPauseListener（），addListener（）。其他方法和ofInt（）类似


- **ObjectAnimation**

直接对对象的属性进行改变操作，从而实现动画效果

具体使用
```
ObjectAnimator animator=ObjectAnimator.ofFloat(rectangle,"TranslationX", 100);
animator.setDuration(5000);
animator.start();
```

ObjectAnimator方法中传入的第二个参数是对象的属性，并且对象对这个属性提供了set和get方法，因为本质上是让ObjectAnimator类根据传入的属性名寻找该对象对应的属性名的set和get方法，从而对对象进行赋值

解决方案：
- 手动设置对象类的set和get方法
	1. 如果是自定义的view，可以直接添加set和get方法
	2. 如果是系统的view，可以通过继承该对象，从而给对象添加上该属性，还可以包装原始动画对象，间接给对象加上set和get方法

第一种方法比较简单,下面是第二种方法的使用
```
public class PropertyAnimation extends AppCompatActivity {
    private TextView rectangle;
	//包装类
    private TextViewWrapper wrapper;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_property_animation);
        rectangle=findViewById(R.id.rectangle);
    }

    public void move(View view) {
        wrapper=new TextViewWrapper(rectangle);
		//传入的是包装类
        ObjectAnimator animator=ObjectAnimator.ofInt(wrapper,"width", 100);
        animator.setDuration(5000);
        animator.start();
    }

    private class TextViewWrapper {
        private TextView view;
        public TextViewWrapper(TextView view){
            this.view=view;
        }

        public void setWidth(int width){
            view.getLayoutParams().width=width;
            view.requestLayout();
        }

        public int getWidth(){
            return view.getLayoutParams().width;
        }
    }
}

```
- **组合动画**

实现组合动画的类是AnimatorSet,可以java设置或者xml设置

```
AnimatorSet.play(Animator anim)   ：播放当前动画
AnimatorSet.after(long delay)   ：将现有动画延迟x毫秒后执行
AnimatorSet.with(Animator anim)   ：将现有动画和传入的动画同时执行
AnimatorSet.after(Animator anim)   ：将现有动画插入到传入的动画之后执行
AnimatorSet.before(Animator anim) ：  将现有动画插入到传入的动画之前执行
```
具体使用
```
AnimatorSet set=new AnimatorSet();
ObjectAnimator animator=ObjectAnimator.ofFloat(rectangle,"translationX", 100);
ObjectAnimator animator1=ObjectAnimator.ofFloat(rectangle,"rotation", 100);
set.setDuration(3000);
set.play(animator).with(animator1);
set.start();
```




## 四、插值器 ##

插值器（Interpolator）是用来指定动画如何变化的，可以自定义插值器来达到自己想要的动画，插值器可以在java中用Animator.setInterpolator或者在xml文件中通过 android:interpolator来设定,android中内置了9种插值器，如下

|Interpolator class|Resouce ID|说明|
|:----:|:----:|:----:|
|LinearInterpolator|@android:anim/linear_interpolator|线性插值器，做匀速运动|
|AccelerateInterpolator|@android:anim/accelerate_interpolator |加速插值器，做加速运动|
|DecelerateInterpolator|@android:anim/decelerate_interpolator |减速插值器，做减速运动|
|AccelerateDecelerateInterpolator|@android:anim/accelerate_decelerate_interpolator |先加速后减速插值器|
|AnticipateInterpolator|@android:anim/anticipate_interpolator|先反方向运动一点，然后再向正方向做加速运动|
|OvershootInterpolator|@android:anim/overshoot_interpolator|先加速在减速，最后会超过终点位置|
|AnticipateOvershootInterpolator|@android:anim/anticipate_overshoot_interpolator |AnticipateInterpolator和OvershootInterpolator效果的叠加|
|AnticipateOvershootInterpolator|@android:anim/anticipate_overshoot_interpolator |AnticipateInterpolator和OvershootInterpolator效果的叠加|
|BounceInterpolator|@android:anim/bounce_interpolator|弹跳插值器，类似球掉落在地上后会弹|
|CycleInterpolator|@android:anim/cycle_interpolator|周期插值器|

**自定义插值器**

自定义插值器需要继承TimeInterpolator，实现getInterpolation方法

具体使用
```
private class MyInterpolator implements TimeInterpolator {

	@Override
	public float getInterpolation(float input) {
		float t=2.0f*input-1.0f;
		return 0.5f*(t*t*t+1.0f);
	}
}
```


## 五、动画的监听 ##

动画的监听是使用addListener，addPauseListener和addUpdateListener

具体使用
```
animator.addListener(new Animator.AnimatorListener() {
	@Override
	public void onAnimationStart(Animator animation) {
		//动画开始
	}

	@Override
	public void onAnimationEnd(Animator animation) {
		//动画结束
	}

	@Override
	public void onAnimationCancel(Animator animation) {
		//动画取消
	}

 	@Override
	public void onAnimationRepeat(Animator animation) {
		//动画重复播放时
	}
});
animator.addPauseListener(new Animator.AnimatorPauseListener() {
	@Override
	public void onAnimationPause(Animator animation) {
		//动画暂停
	}

	@Override
	public void onAnimationResume(Animator animation) {
		//动画重新启动
 	}
});
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
	@Override
  	public void onAnimationUpdate(ValueAnimator animation) {
		//获取动画过程中的当前值
   	}
});
```

