---
title: Activity详解
date: 2018-03-27 10:57:45
tags: android
categories: android
---


## 什么是Activity

- **Activity 是 Android 的应用程序组件，它提供给用户一个交互式的接口功能**。Activity本身是没有界面的，所以Activity创建了一个窗口，开发人员可以通过 `setContentView(view)` 接口把 UI 放到 Activity 创建的窗口上，当 Activity 指向全屏窗口时，也可以用其他方式实现：作为悬浮窗口，或者嵌入到其他的 Activity 中。**Activity 是单独的，用于处理用户操作**。




## Activity 的生命周期

![activity_life](..\images\android\activity_life.png)

```java
public class Activity extends ApplicationContext {

     // 活动第一次启动的时候，触发该方法，可以在此时完成活动的初始化工作。
     protected void onCreate(Bundle savedInstanceState);     

     // “用户可见不可交互” 的状态，表示活动将被展现给用户。
     protected void onStart();     

     // 处于停止状态的活动需要再次展现给用户的时候，触发该方法。
     protected void onRestart();   

     // “用户可交互” 状态，当活动和用户发送交互的时候，触发该方法。
     protected void onResume();    

     /* 表示activity正在停止。
     此时可停止动画或将活动的状态持久化，比如正在编辑的数据库记录等。*/
     protected void onPause();     

     /* 表示activity即将停止，可以做一些稍微重量级的回收工作（不能太耗时）。
     （如果内存吃紧，系统会直接结束这个活动，而不会触发 onStop() 方法）*/
     protected void onStop();      

     /*表示activity即将被销毁，可以做一些回收工作和最终的资源释放。
     （如果内存吃紧，系统会直接结束这个活动，而不会触发 onDestroy() 方法）*/
     protected void onDestroy();

     /*系统调用该方法，允许活动保存之前的状态。(如：保存在一串字符串的光标所处位置等。)
     通常情况下，开发者不需要重写该方法，在默认的实现中，已经提供了自动保存所涉及到的用户组件的所有状态信息*/
     protected void onDestroy();
 }
```


## Activity 的四种启动模式

>设置Activity的启动模式，需在AndroidManifest.xml中对应的<activity>标签设置android:launchMode属性。
```xml
<activity android:name=".MainActivity" android:launchMode="standard" />  
```


### standard

**标准模式，每次启动activity都会创建新的实例，不论该实例是否已存在。**

因此，一个任务栈中可以有多个实例，每个实例也可以属于不同的任务栈。



### singleTop

**栈顶复用模式，可有多个实例，但不允许多个相同的Activity叠加。**

若当前activity已位于任务栈的栈顶，那么此时activity不会被重建，同时`onNewIntent(Intent intent)`方法被回调（通过方法参数可以取出当前请求的信息）；如果当前activity的实例不是位于栈顶，那么新的activity仍然会重新创建。

例如：
```flow
a1=>end: A (standard)
a2=>end: B (standard)
a3=>end: B (standard)

a1(right)->a2(right)->a3(right)
```

```flow
a1=>end: A (standard)
a2=>end: B (singleTop)
a3=>end: B.onNewIntent()

a1(right)->a2(right)->a3(right)
```



### singleTask

**栈内复用模式，任务栈中有且只有一个该activity的实例。**
若activity不存在，则会在当前任务栈创建一个新的实例，若存在，则会把任务栈中在其之上的求他activity 销毁掉并调用它的`onNewIntent()`方法。



### singleInstance

**单实例模式，有且只有一个实例，并且这个实例将独立运行在Task中，不允许有别的Activity存在**。



## IntentFilter的匹配规则

### action
### category
### data