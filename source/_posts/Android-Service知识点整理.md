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
   > 此方式实现了进程通信，用法类似IBinder，只是在实现上存在差异
     <font color=#0099ff>原理:　新建TestAIDL.aidl文件后，IDE会生成配套的TestAIDL.java文件，其内部实现了类似IBinder的使用实现，同时通过BinderProxy实现进程间的通信</font>
     [更多介绍](http://www.open-open.com/lib/view/open1469493649028.html)
     #### 同一个应用内
     ``` java
      1. 使用android-studio，new 一个 AIDL，此过程还会同时生成配套的相关.java文件(自己建有点麻烦，还是交给android-studio吧)，其内容
         interface TestAIDL {
           /** 自动生成 */
           void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString);
           /** 自定义方法 */
           int plus(int a, int b);
           String toUpperCase(String str);
        }
      2. 在app->build->generated->source->aidl->debug->com->example->test->TestAIDL.java生成了TestAIDL.java文件
         其中的内部内类Stub(public static abstract class Stub extends android.os.Binder implements com.sky.minetest.service.OneAIDLService{})用以实现IBinder的能力
      3. 在ServiceConnection中得到TestAIDL的实例和方法,　即可调用完成通信
         private ServiceConnection oneConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                TestAIDL test = TestAIDL.Stub.asInterface(service);
                int result = aidlService.plus(1, 2);
                String str = aidlService.toUpperCase("abcde");
            }
          }
     ```
    > #### 不同应用
      配置应用A的AndroidManifest.xml
      ``` xml
       <service  
          android:name="com.example.test.Myservice"  
          android:process=":remote" >  
          <intent-filter>  
              <action android:name="com.example.test.Myservice"/>  
          </intent-filter>  
       </service>
      ```
      > B应用的java调用
      ```java
        1. 按照应用A的报名和路径，把A中的AIDL文件拷贝到B应用中
        2. 用隐式Intent方式启动A的Myservice
           Intent intent = new Intent("com.example.test.Myservice");  
           bindService(intent, connection, BIND_AUTO_CREATE);
        3. 和同一应用内使用ServiceConnection调用AIDL的方式相同使用AIDL的方法
        4. 完成通信
      ```


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
