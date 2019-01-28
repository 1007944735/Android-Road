## 一、ActionBar ##

### 1. ActionBar简介 ###
ActionBar主要用于标识应用程序以及用户所处的位置并提供相关操作以及全局的导航功能。

![](https://i.imgur.com/yvpnzjX.png)

> 1. App icon：主要用于显示App的icon，如果当界面不是一级界面，还可以展示返回导航键
> 2. View Control：用于切换不同的视图或者展示费交互信息：如标题等
> 3. Action Button：用于展示App中重要的按钮，如果acitonbar中放不下，会放入到 Action overflow中
> 4. Action overflow：用于存放较少使用的按钮

### 2.基本使用 ###
> Activity是继承ActionBarActivity
> 或者将App或Activity的主题设置为@style/AppTheme

不展示ActionBar，通过下面代码
```
//获取ActionBar
        ActionBar bar=getSupportActionBar();
        //隐藏ActionBar
        bar.hide();
        //显示ActionBar
        bar.show();
```

App Icon区域中，一部分展示Logo,另一部分是展示返回导航键，导航键主要方法如下：
> 返回导航键的显示与隐藏：setDisplayHomeAsUpEnabled
> 返回导航键的图标设置：setHomeAsUpIndicator
> 返回导航键的事件处理：重写Activity的onOptionsItemSelected
```
@Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if(item.getItemId()==android.R.id.home){
            Toast.makeText(this, "返回键", Toast.LENGTH_SHORT).show();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
```

Logo的显示，隐藏和设置
```
bar.setLogo(R.drawable.android);
bar.setDisplayShowHomeEnabled(true);
bar.setDisplayUseLogoEnabled(true);
```

第二部分是Action Control

> 显示与隐藏标题：setDisplayShowTitleEnabled
> 设置标题：setTitle
> 设置副标题：setSubtitle
> 设置自定义view：setCustomView

Action Buttons和Action overflow

1. 重写onCreateOptionsMenu方法，生成菜单按钮，有两种方法，一种是xml添加，另一种是动态添加

xml文件
```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/select"
        android:title="搜索"
        app:showAsAction="always"
        />
    <item android:id="@+id/delete"
        android:title="删除"
        app:showAsAction="never"
        />
    <item android:id="@+id/add"
        android:title="添加"
        app:showAsAction="always"
        />
</menu>
```
> showAsAction决定菜单显示的位置，有三个值：always，显示在菜单栏中；never，显示在溢出栏中；ifRoom，如果空间够显示在菜单栏，否则显示在溢出栏

```
@Override
public boolean onCreateOptionsMenu(Menu menu) {
	getMenuInflater().inflate(R.menu.main,menu);
	return true;
}
```
2. 重写onOptionsItemSelected，以响应菜单的点击事件

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
	if(item.getItemId()==android.R.id.home){
		Toast.makeText(this, "返回键", Toast.LENGTH_SHORT).show();
		return true;//返回true，结束点击事件，不会进入下一级菜单
	}
	if(item.getItemId()==R.id.select){
		Toast.makeText(this, "搜索", Toast.LENGTH_SHORT).show();
		return true;
	}
	if(item.getItemId()==R.id.add){
		Toast.makeText(this, "添加", Toast.LENGTH_SHORT).show();
		return true;
	}
	if(item.getItemId()==R.id.delete){
		Toast.makeText(this, "删除", Toast.LENGTH_SHORT).show();
		return true;
	}
	return super.onOptionsItemSelected(item);//不返回true，避免拦截菜单项的点击事件
}
```
## 二、ToolBar ##
### 1. ToolBar概述 ###

随着Android版本的不断更新，ActionBar表现出差异，缺乏灵活性，所以google在v7包中推出了ToolBar。建议使用ToolBar替代ActionBar

### 2. ToolBar简单使用 ###

```
<android.support.v7.widget.Toolbar
	android:layout_width="match_parent"
	android:layout_height="?attr/actionBarSize"
	android:background="#ff0000"
	app:title="标题"
	app:subtitle="副标题"
	app:navigationIcon="@mipmap/left"
	app:logo="@mipmap/android"
	app:titleMarginStart="40dp"
	app:titleTextColor="#ffffff"
	app:subtitleTextColor="#ffffff">
</android.support.v7.widget.Toolbar>
```
- navigationIcon:设置左边的导航键
- logo：设置图标
- titleMarginStart：设置标题距离左边的距离

效果图：![](https://i.imgur.com/uNAd0tU.jpg)

修改标题和副标题的文字大小

```
//标题字颜色和大小
<style name="toolbarTitle" parent="TextAppearance.Widget.AppCompat.Toolbar.Title">
  	<item name="android:textColor">#00ff00</item>
	<item name="android:textSize">16sp</item>
</style>
//副标题字颜色和大小
<style name="toolbarSubtitle" parent="TextAppearance.Widget.AppCompat.Toolbar.Subtitle">
	<item name="android:textColor">#00ff00</item>
	<item name="android:textSize">12sp</item>
</style>
```
使用：
```
app:titleTextAppearance="@style/toolbarTitle"
app:subtitleTextAppearance="@style/toolbarSubtitle"
```
>如果想要标题居中，可以在Toolbar中嵌套一个TextView，使TextView居中


左边导航键点击事件：
```
Toolbar toolbar=findViewById(R.id.toolbar);
toolbar.setNavigationOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {
		Toast.makeText(MainActivity.this, "点击导航键", Toast.LENGTH_SHORT).show();
	}
});
```

### 2. ToolBar配合menu使用 ###
使用xml方式创建menu
```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/select"
        android:title="搜索"
        app:showAsAction="ifRoom"
        android:icon="@mipmap/select"/>
    <item android:id="@+id/add"
        android:title="添加"
        app:showAsAction="ifRoom"
        android:icon="@mipmap/add"/>
    <item android:id="@+id/delete"
        android:title="删除"
        app:showAsAction="never"
        android:icon="@mipmap/delete"/>
</menu>
```
Toolbar加载menu
```
toolbar.inflateMenu(R.menu.main);
//点击事件
toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
	@Override
	public boolean onMenuItemClick(MenuItem item) {
		if(item.getItemId()==R.id.add){
			Toast.makeText(MainActivity.this, "add", Toast.LENGTH_SHORT).show();
		}
		if(item.getItemId()==R.id.delete){
			Toast.makeText(MainActivity.this, "delete", Toast.LENGTH_SHORT).show();
		}
		if(item.getItemId()==R.id.select){
			Toast.makeText(MainActivity.this, "select", Toast.LENGTH_SHORT).show();
		}
		return false;
	}
});
```
```
<style name="ToolbarPopupTheme">
	<item name="android:colorBackground">#2f99ff</item>
	<item name="actionOverflowMenuStyle">@style/OverflowMenuStyle</item>//增加一个item，控制menu
</style>
<style name="OverflowMenuStyle" parent="Base.Widget.AppCompat.Light.PopupMenu.Overflow">
	<item name="overlapAnchor">false</item>//使menu位置位于toolbar之下
</style>
```
> app:popupTheme="@style/ToolbarPopupTheme" 设置弹出菜单的样式
效果图:

![](https://i.imgur.com/fcj7fxf.jpg)

### 3. Toolbar与AppBarLayout,CoordinatorLayout，CollapsingToolbarLayout配合使用 ###

##### AppBarLayout #####

AppBarLayout继承于LinearLayout,是一个垂直的LinearLayout，实现了Material Design的许多功能。一般依赖于CoordinatorLayout，结合ToolBar，TabLayout使用。

**主要属性和方法**
	
- app:elevation：设置阴影
- app:stateListAnimator：为控件定义一系列的动画，并随着控件状态的改变展示不同的动画效果（Api>21）
- app:expanded：设置AppBarLayout默认状态（true为展开，false为折叠），需同时设置app:layout_scrollFlags才有效果
- app:layout_scrollFlags：五种状态，scroll，enterAlways，enterAlwaysCollapsed，exitUntilCollapsed，snap

> scroll:AppBarLayout的子view设置后，该view会随着滚动控件滚动（想要滚动，必须设置scroll）
> enterAlways：向下滚动时，AppBarLayout会立即出现
> enterAlwaysCollapsed：需要为ToolBar设置minHeight或者设置高度，向下滚动时，会先显示minHeight部分内容，下滑到底部时才会显示全部内容
> exitUntilCollapsed：向上滚动时，AppBarLayout不会全部向上滚动，会显示ToolBar设置minHeight内容，下滑时和enterAlwaysCollapsed类似
> snap：上滑时，当layout_height显示少于一半，layout_heightde内容全部移出屏幕；下滑时，当layot_height显示大于一半，layout_height内容全部显示

- addOnOffsetChangedListener：当AppBarLayout的竖直方法上的偏移量发生变化的时候回调
- removeOnOffsetChangedListener：移出offSetChange时回调
- setExpanded (boolean expanded)：设置AppBarLayout是展开状态还是折叠状态
- setExpanded (boolean expanded, boolean animate)：设置AppBarLayout是展开状态还是折叠状态，animate控制展开或者折叠时是否需要动画
- setOrientation：设置AppBarLayout子view的排列方式
- getTotalScrollRange：返回所有子view的滑动范围

##### CoordinatorLayout #####
CoordinatorLayout是一个容器，主要是协调子view之间的交互，作为顶层的布局，AppBarLayout需要放在CoordinatorLayout中才有效果

```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</android.support.design.widget.CoordinatorLayout>
```

##### CollapsingToolbarLayout #####
CollapsingToolbarLayout是一个可以折叠的ToolBar，继承至FrameLayout，给它设置app:layout_scrollFlags,可以控制包含在CollapsingToolbarLayout中的控件响应layout_behavior事件时作出相应的scrollFlags滚动事件

**主要属性或方法**

- app:contentScrim：当上滑到toolbar高度期间直到达到toolbar高度时，给toolbar设置的背景色，以及过渡色，也可以设置图片
- app:layout_scrollFlags：和AppBarLayout的一样
- app:expandedTitleGravity：展示时标题的位置
- app:expandedTitleMarginStart：展开时标题离左边的距离，类似的还有expandedTitleMarginEnd，expandedTitleMarginButtom，expandedTitleMarginTop
- app:collapsedTitleGravity：折叠时标题的位置
- app:expandedTitleTextAppearance:展开时标题的样式
- app:collapsedTitleTextAppearance：折叠时标题的样式
- app:layout_collapseMode：给子view设置滚动方式，有三个属性如下：
	- off：默认值，唯有折叠效果
	- pin：上滑时固定在顶部
	- parallax：子view和滚动控件以一定的视差滚动
- app:layout_collapseParallaxMultiplier：配合parallax使用，定义滚动差值

> 当CollapsingToolbarLayout包含Toolbar时，Toolbar的titleTextAppearance和subtitleTextAppearance会无效，使用
CollapsingToolbarLayout的expandedTitleTextAppearance和collapsedTitleTextAppearance设置标题样式

##### 具体例子 #####
xml文件
```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:collapsedTitleTextAppearance="@style/collapsedTitleStyle"
            app:contentScrim="#ff0000"
            app:expandedTitleTextAppearance="@style/expandedTitleStyle"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/image"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:scaleType="centerCrop"
                android:src="@mipmap/bg_toolbar"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7" />


            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?android:attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/ToolBarPopupTheme"
                app:title="标题"
                app:titleTextColor="#ff0000">

            </android.support.v7.widget.Toolbar>

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="16dp"
            android:text="@string/large_text" />
    </android.support.v4.widget.NestedScrollView>
</android.support.design.widget.CoordinatorLayout>
```
Acitivity文件
```
public class MainActivity extends AppCompatActivity {
    private Toolbar toolbar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        toolbar=findViewById(R.id.toolbar);
        toolbar.inflateMenu(R.menu.menu_test);
        toolbar.setNavigationIcon(R.mipmap.left);
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "返回键", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
效果图

![](https://i.imgur.com/XprEBQk.gif)