---
title: Android-Service知识点整理
date: 2017-04-12 14:08:08
categories: Android
tags: android-service
---

1. Service基础
2. Service和Activity通信
3. Service、Thread、Process

<!-- more -->

1. ## Service基础
 - ### 后台Service
    + #### 启动关闭方式:
      > Service只创建一次，即只要存在无论启动的方式与次数，都不在重建而是引用
      [更多介绍](http://blog.csdn.net/pi9nc/article/details/18764415)
      ```java
      1. Intent intent = new Intent(this, MyService.class);  
      　 startService(intent)
      　 stopService(intent)
      　 [注]
      　 周期:　与Activity周期无关，onCreate()->onStartCommand()->onDestroy()
      　 重复启动:　调用多次onStartCommand()         　

      2. Intent bindIntent = new Intent(this, MyService.class);
      　 bindService(bindIntent, connection, Context.BIND_AUTO_CREATE);
      　 unbindService(connection);
      　 [注]
      　 周期:　与Activity周期同步
      　 重复启动:　不调用任何方法，直到unbindService(connection)
      　 connection:　private ServiceConnection connection = new ServiceConnection() { ... }，用以Service与Activity通信
      　 Context.BIND_AUTO_CREATE:　flags标签，表示Service不存在即创建
      ```

 - ### 前台Service
   > 在通知栏一直显示, 可在内存不足的情况下也不被系统杀掉，例如: 天气服务
    ```java
    1. private void showNotification(String mes){
        　PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, new Intent(this, MainActivity.class), 0);
        　NotificationCompat.Builder builder = new NotificationCompat.Builder(this)
        　　　.setSmallIcon(R.drawable.ic_launcher)
        　　　.setContentTitle("Service 小通知")
        　　　.setContentText(mes)
        　　　.setContentIntent(pendingIntent);
        　startForeground(1, builder.build());
    　}

    2. public void onCreate() {
        　super.onCreate();
        　showNotification("onCreate execute and show this message");
    　}
    ```

2. ## Service和Activity通信
 - ### IBinder
   > 此方法适用于线程内通信，线程间会在ServiceConnection中调用IBinder方法时，报异常
     <font color=#0099ff>原理:　通过bindService()把ServiceConnection传递给Service,　同时在bind成功后，调用onBind方法，把自定义实现的IBinder回传给ServiceConnection,　进行调用</font>
     [更多介绍](http://blog.csdn.net/yuzhiboyi/article/details/7558176)
 ```java
 /** Service中 */
 public class ServiceOne extends Service {
   private OneBinder oneBinder = new OneBinder();

   @Override
    public IBinder onBind(Intent intent) {
        return oneBinder;
    }

    public class OneBinder extends Binder{

        public void doingEvent(String name){
            Log.i(TAG, "doingEvent and the name -->> " + name);
        }
    }
 }

 /** Activity中 */
 public class OneActivity extends AppCompatActivity{
   private ServiceConnection oneConnection = new ServiceConnection() {
      @Override
      public void onServiceConnected(ComponentName name, IBinder service) {
        ((ServiceOne.OneBinder)service).doingEvent("你猜");
      }

      @Override
      public void onServiceDisconnected(ComponentName name) { ... }
  }

  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        Intent bindIntent = new Intent(this, MyService.class);
        bindService(bindIntent, oneConnection, Context.BIND_AUTO_CREATE);
    }

 }
 ```
 - ### AIDL
 > 此方式实现了进程通信
   <font color=#0099ff>原理:
   </font>

3. ## Service、Thread、Process
  - Service与Activity都运行在主线程(UI线程)，Thread.currentThread().getId() == 1
  - Service中的任务需要避开主线程，防止ANR发生
    > + Thread方式
    ``` java
      new Thread(new Runnable() {  
            @Override  
            public void run() { ... }  
      }).start();
    ```
    > + Process方式
    ``` xml
      <service  
        android:name="com.example.servicetest.MyService"  
        android:process=":remote" />   
    ```
  - Service用Process方式时已处于新的com.example.test_service:remote进程，此时通信要通过AIDL
