## 一、BroadcastReceiver概述 ##
Android应用可以从Android系统和其他Android应用发送或接受广播消息，类似于**发布-订阅**设计模式。BroadcastReceiver是Android四大组件之一。
> Android广播一般分为两个角色：广播发送者，广播接收者

## 二、BroadcastReceiver作用 ##
- 用于监听/接收应用发出的广播消息，并作出响应。
- 应用场景：
   a、不同组件之间通信（包括应用内/不同应用之间）
   b、与And容地系统在特定情况下的通信
   > 如当电话呼入时，网络可用时

   c、多线程通信
## 三、BroadcastReceiver注册方式 ##
- 静态注册，即在AndroidManifest中注册

	> 静态注册即使App不运行，也会接受广播
	```
	<receiver android:name=".mBroadcastReceiver">
            <intent-filter>
                <action android:name="BROADCAST_ACTION"/>
            </intent-filter>
    </receiver>
	```
- 动态注册，即在代码中注册
	> 动态注册只有在App运行时才会接受广播
	> 动态广播最好**在onResume中注册，在onPause销毁**
	
	`registerReceiver(BroadcastReceiver，IntentFilter)`
## 四、BroadcastReceiver分类 ##
- 普通广播
- 系统广播
- 有序广播
- 粘性广播
- App应用内广播

### 1. 普通广播 ###
开发者自身定义intent 的广播（最常用）,发送广播使用如下：
```
Intent intent=new Intent();
//对应BroadcastReceiver中的intentFilter的action
intent.setAction(BROADCAST_ACTION);
//发送广播
sendBroadcast（intent）；
```
- 若被注册了的广播接收者中注册是intentFilter的action与上述匹配，则会接受此广播（回调onReceive）。如下
```
<receiver android:name=".mBroadcastReceiver">
     <intent-filter>
           <action android:name="BROADCAST_ACTION"/>
     </intent-filter>
</receiver> 
```
- 若发送广播有相应权限，那么广播接收者也需要有相应的权限

### 2. 系统广播 ###
- Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播
- 每个广播都有特定的intent-filter，如下：

|系统广播|Action|
|:----:|:----:|
|监听网络变化	|android.net.conn.CONNECTIVITY_CHANGE|
|关闭或打开飞行模式|Intent.ACTION_AIRPLANE_MODE_CHANGED|
|充电时或电量发生变化|Intent.ACTION_BATTERY_CHANGED|
|电池电量低|Intent.ACTION_BATTERY_LOW|
|电池电量充足（即从电量低变化到饱满时会发出广播|Intent.ACTION_BATTERY_OKAY|
|系统启动完成后(仅广播一次)|Intent.ACTION_BOOT_COMPLETED|
|按下照相时的拍照按键(硬件按键)时|Intent.ACTION_CAMERA_BUTTON|
|屏幕锁屏|Intent.ACTION_CLOSE_SYSTEM_DIALOGS|
|设备当前设置被改变时(界面语言、设备方向等)|Intent.ACTION_CONFIGURATION_CHANGED|
|插入耳机时|Intent.ACTION_HEADSET_PLUG|
|未正确移除SD卡但已取出来时(正确移除方法:设置--SD卡和设备内存--卸载SD卡)|Intent.ACTION_MEDIA_BAD_REMOVAL|
|插入外部储存装置（如SD卡）|Intent.ACTION_MEDIA_CHECKING|
|成功安装APK|Intent.ACTION_PACKAGE_ADDED|
|成功删除APK|Intent.ACTION_PACKAGE_REMOVED|
|重启设备|Intent.ACTION_REBOOT|
|屏幕被关闭|Intent.ACTION_SCREEN_OFF|
|屏幕被打开|Intent.ACTION_SCREEN_ON|
|关闭系统时|Intent.ACTION_SHUTDOWN|
|重启设备|Intent.ACTION_REBOOT|

### 3. 有序广播 ###
- 发出去的广播被广播接收者按照先后顺序接收（针对广播接收者而言）
- 广播接收者接收广播的顺序规则
	1. 按照priority属性从大到小（-1000~1000）
	```
	<receiver android:name=".MyReceiver1">  
    	<intent-filter android:priority="200">  
    		<action android:name="com.song.123"/>  
    	</intent-filter>  
    </receiver> 
	```
	2. priority值相同时，动态广播优先
- 特点
	1. 接收广播顺序接收
	2. 先接受的广播接收者可以对广播进行截断，即后接收的广播接收者不再接收到此广播
	
		> abortBroadcast() //拦截广播
	3. 先接受的广播接收者可以对广播进行修改，即后接收的广播接受者接收到被修改过的广播
- 发送方式
	> sendOrderedBroadcast(intent)
### 4. App应用内广播 ###
- 背景：Android中的广播可以跨App直接通信（exported：四大组件中都有的属性，是否支持其他应用调用此组件，对于有intent-filter情况下默认值为true）
- 冲突，可能出现的问题：
	- 其他App针对性发出与当前App intent-filter相匹配的广播，由此导致当前App不选接收广播并处理
	- 其他App注册与当前App一致的intent-filter用于接收广播，获取广播具体信息；即会出现安全性&效率性的问题。
- 解决方案：使用App应用内广播（Local Broadcast）

	> App应用内广播相当于一种局部广播，广播的接收者和发送者都属于同一个App
	> 相比于全局广播（普通广播），具有效率高和安全性高的优点
- 使用方法1：将全局广播设置为局部广播
	1. 注册广播时将exported属性设置为false，使得非本App内部发送的广播不会被接收
	2. 在广播发送和接收的时候增加相应的权限permission，用于权限验证
	3. 发送广播时指定接收广播的包名，此广播只会被发送到此包中的App内与之相对应的广播接收者中
	> intent.setPackage(PackageName) 指定包名
- 使用方法2：使用封装好的LocalBroadcastManager类
	> 使用LocalBroadcasrManager发送的广播只能使用动态注册，不能使用静态注册

	```
	//注册应用内广播接收器
	//步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
	mBroadcastReceiver = new mBroadcastReceiver(); 
	IntentFilter intentFilter = new IntentFilter(); 

	//步骤2：实例化LocalBroadcastManager的实例
	localBroadcastManager = LocalBroadcastManager.getInstance(this);

	//步骤3：设置接收广播的类型 
	intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

	//步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
	localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);

	//取消注册应用内广播接收器
	localBroadcastManager.unregisterReceiver(mBroadcastReceiver);

	//发送应用内广播
	Intent intent = new Intent();
	intent.setAction(BROADCAST_ACTION);
	localBroadcastManager.sendBroadcast(intent);
	```
### 5. 粘性广播 ###
粘性广播在发送后一直会存在于系统的消息容器里，等待对应的处理器去处理，如果暂时没有处理器处理这个消息则一直在消息容器中处于等待状态，粘性广播的Receiver如果被销毁，那么下次重建时会自动接收到消息数据。此广播在5.0中被移除。
