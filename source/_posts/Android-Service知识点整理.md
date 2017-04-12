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
> ### 后台Service
>> #### 启动关闭方式:
>> Service只创建一次，即只要存在无论启动的方式与次数，都不在重建而是引用
>> ```
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
   重复启动:　不调用任何方法
   connection:　private ServiceConnection connection = new ServiceConnection() { ... }，用以Service与Activity通信
   Context.BIND_AUTO_CREATE:　flags标签，表示Service不存在即创建
```
>
>
>
