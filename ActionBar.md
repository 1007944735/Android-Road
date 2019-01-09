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
[https://blog.csdn.net/da_caoyuan/article/details/79557704](https://blog.csdn.net/da_caoyuan/article/details/79557704)


