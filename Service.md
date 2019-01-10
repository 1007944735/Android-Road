## 一、Service简介 ##
Service是Android程序中四大组件之一，都是Context的子类，与Activity不同没有UI界面，运行于后台的组件。
Service默认并不会运行在子线程中，而是执行在UI线程中，所以不能再Service中执行耗时操作。

## 二、Service的启动方式 ##
- startService：启动一个Service，不进行通信，当启动的Activity销毁时，此Service不销毁。停止服务使用stopService。
- bindService：启动一个Service，将Activity与此Service进行绑定，可以进行通信。停止服务使用unbindService。
- startService和bindservice：同时使用startService和bindservice启动Service，适合在既要在长期运行，又要保持与Service的通讯时使用。停止服务需要使用stopService和unbindService。


## 三、Service的生命周期 ##
![](https://i.imgur.com/60cfagw.png)

上面的图就是Service的生命周期，左边是使用了startService所创建服务的生命周期，右图是使用了bindService所创建服务的生命周期。

- onCreate：系统在service第一次创建时执行此方法，在整个生命周期中**只会执行一次**。如果service已经运行，这个方法就不会被调用。

- onStartCommand：使用startService方法启动service都会回调该方法（**多次调用**）。一旦这个方法执行，service就会启动并长期运行。通过调用stopService方法来停止服务。

- onBind：使用bindService方法启动service会调用此方法，**一次调用**，绑定后下次在调用bindService时不会回调该方法。需要返回一个IBinder来使客户端能够与Service进行通信。

- onUnbind：当调用unbindService，想要解除与Service的绑定时系统会调用该方法。**一次调用**，调用后，在使用unbindService会抛出异常。

- onDestory：系统在Service不再被使用并要销毁时调用此方法，**一次调用**，Service在此方法中应进行资源的回收与释放。

## 四、创建Service ##

- Service：这是适合所有服务的基类。继承此基类，时需要创建一个用于执行所有服务工作的新线程，默认情况下，运行于主线程。
- IntentService：这是Service的子类，它使用工作线程逐一处理所有启动请求。如果不要求服务同时处理多个请求，这是最好的选择。只需要实现onHandleIntent方法即可。

### 扩展IntentService ###

IntentService执行以下操作：
- 创建默认的工作线程，用于在应用的主线程外执行传递给onStartCommand的所有Intent。
- 创建工作队列，用于将Intent逐一传递给OnHandleIntent实现。
- 处理完所有启动请求后停止服务，不用调用stopService。
- 提供onBind的默认实现（返回null）。
- 提供onStartCommand的默认实现，可将Intent依次发送到工作队列和onHandleIntent实现。

以下的代码来自官方文档：
```
public class HelloIntentService extends IntentService {

  /**
   * A constructor is required, and must call the super IntentService(String)
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      try {
          Thread.sleep(5000);
      } catch (InterruptedException e) {
          // Restore interrupt status.
          Thread.currentThread().interrupt();
      }
  }
}
```

如果还要重写其他回调方法，请确保滴啊用超类实现，以便IntentService能够妥善处理工作线程的生命周期。

例如，onStartCommand必须返回默认实现：
```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
    return super.onStartCommand(intent,flags,startId);
}
```
## 五、前台服务 ##
前台服务被认为是用户主动意思到的一种服务，因此在内存不足时，系统也不会考虑将其终止。前台服务必须为状态栏提供通知，放在“正在进行”标题下方，这意味着除非服务停止或从前台移除，否则不能清除通知。例如：通过服务播放音乐的音乐播放器设置为前台运行。

要请求让服务运行于前台，调用**startForeground**方法。此方法采用两个参数：唯一标识通知的整型数和状态栏的Notification。通过stopForeground方法可以取消通知，即前台服务降为后台服务。

> 提供给startForeground的整型ID不得为0.

例子：
```
public class ForeService extends Service{
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        beginForeService();
    }

    private void beginForeService() {
        //创建通知
        Notification.Builder mBuilder = new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentText("2017-2-27")
                .setContentText("您有一条未读短信...");
        //创建点跳转的Intent(这个跳转是跳转到通知详情页)
        Intent intent = new Intent(this,NotificationShow.class);
        //创建通知详情页的栈
        TaskStackBuilder stackBulider = TaskStackBuilder.create(this);
        //为其添加父栈 当从通知详情页回退时，将退到添加的父栈中
        stackBulider.addParentStack(NotificationShow.class);
        PendingIntent pendingIntent = stackBulider.getPendingIntent(0,PendingIntent.FLAG_UPDATE_CURRENT);
        //设置跳转Intent到通知中
        mBuilder.setContentIntent(pendingIntent);
        //获取通知服务
        NotificationManager nm = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        //构建通知
        Notification notification = mBuilder.build();
        //显示通知
        nm.notify(0,notification);
        //启动前台服务
        startForeground(1,notification);
    }
}
```

