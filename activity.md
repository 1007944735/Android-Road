## 一、Activity的生命周期 ##
### 1.正常情况下的生命周期 ###
来自官方文档的一张图
![](https://i.imgur.com/Ft9xF2W.png)
- onCreate：当Activity被创建时会调用。在这个方法中一般进行初始化工作，比如去加载界面，初始化Activity所需的数据，或者借助Bundle来恢复异常状态（旋转屏幕）下Activity结束时的状态。

- onStart：表示Activity正在被启动，这时的Activity已经出现，但没有出现在前台，无法和用户进行交互（不可见）。

- onResume：表示Activity已经可见，出现在前台并开始活动。

- onPause：表示Activity正在停止，仍然可见，正常情况下，onStop会被调用。onPause中不能进行耗时操作，因为onPause必须执行完毕，才会进行新Activity的onResume方法。与onResume对应。

- onStop：表示Activity即将停止，不可见。与onStart对应。

- onRestart：表示Activity正在重新启动。一般情况下，当Activity从不可见状态变成可见状态时，该方法就会被调用。

- onDestory：表示Activity即将被销毁，可以进行一些回收工作和资源的回收。

### 2.异常情况下的生命周期 ###
#### 情况1：资源相关的系统配置发生改变导致Activity被杀死并重新创建 ####
当突然旋转屏幕，由于系统配置发生了改变，在默认情况下，Activity会被销毁并重新创建，当然我们也可以阻止系统重新创建我们的Activity。其生命周期如下：

旧Activity ——> 发生意外情况 ——>onSaveInstanceState ——> onDestory ——> 新Activity ——> onCreate ——> onRestoreInstanceState

当旧Activity发生意外情况会调用onSaveInstanceState方法。onSaveInstanceState方法会保存当前Activity的状态，这个方法调用的时机是在onStop之前，并且只有在Activity发生意外时才会被调用，正常情况下不会被调用。旧Activity调用onSaveInstanceState后，仍会执行onPause，onStop，onDestory，然后重新创建新的Activity，系统调用onRestoreInstanceState，并且把Activity销毁时onSaveInstanceState保存的Bundle对象作为参数传给onRestoreInstanceState。从时序来讲，onRestoreInstanceState调用在onStart之后。

#### 情况2：资源内存不足导致低优先级的Activity被杀死 ####

##### Activity的优先级： #####

（1）前台Activity——正在和用户交互的Activity，优先级最高
（2）可见但非前台Activity——比如弹出一个Dialog，导致Activity可见但无法进行直接交互
（3）后台Activity——已经被暂停的Activity，优先级最低

##### 阻止Activity不被创建： #####

给Activity指定configChanges属性,如，不想让Activity在屏幕旋转时重新创建，可以给configChanges添加orientation。常用的有locale（切换系统语言），orientation（屏幕方向），keyboardHidden（键盘）。
## 二、Activity的启动模式 ##

### 启动模式的结构——栈 ###

Activity的管理采用任务栈的形式，“先进后出”的栈结构

### 四种启动模式 ###

- 标准模式（standard）：默认启动模式，每启动一个Activity，就会创建一个新的Activity实例并置于栈顶。谁启动了这个Activity，那么这个Activity就会运行在启动它的那个Activity任务栈中。
**特殊情况：如果在Servic或Application中启动一个Activity，并没有所谓的任务栈，可以使用标记位Flag来解决：为待启动的Activity设置FLAG_ACTIVITY_NEW_TASK标记位，创建一个新栈**

- 栈顶复用模式（singleTop）:如果新建的Activity位于任务栈的栈顶，那么不会被创建，而是复用栈顶的实例，会调用onNewIntent方法；如果新建的Activity不位于任务栈的栈顶，那么会新创建一个Activity实例并置于栈顶。

  应用场景：在通知栏点击收到的通知，会重新创建一个Activity实例，这是可以使用singleTop模式，否则每次点击都会创建一个新的Activity实例。

- 栈内复用模式（singleTask）：单例模式，即一个任务栈中只存在一个实例。可以在配置文件中设置该Activity需要加载到哪个栈中。
```
<activity android:name=".Activity1"
	android:launchMode="singleTask"
	android:taskAffinity="com.lvr.task"
	android:label="@string/app_name">
</activity>
```
launchMode：启动模式，taskAffinity：栈名

在这种模式下，如果指定的栈不存在，那么就会重新创建一个新的任务栈，将创建的Activity压入栈内；如果指定的栈存在，并且存在待创建的Activity实例，那么就将此Activity上面的Activity全部出栈，重用并让该Activity位于栈顶，然后调用onNewIntent方法。

应用场景：大多数App的主页。

- 单例模式（singleInstance）：启动一个Activity时，都会重新创建一个新的任务栈，并将Activity实例放入此栈中，以后再启动此Activity都会重用该栈内的实例。

应用场景：呼叫来电界面。

### Activity的Flags ###
常用的标记位：

- FLAG_ACTIVITY_NEW_TASK：作用是为Activity指定“singleTask”，效果一致。
- FLAG_ACTIVITY_SINGLE_TOP：作用是为Activity指定“singleTop”，效果一致。
- FLAG_ACTIVITY_CLEAR_TOP：作用是清除所有在待启动Activity上面的Activity，常与FLAG_ACTIVITY_SINGLE_TOP一起出现。