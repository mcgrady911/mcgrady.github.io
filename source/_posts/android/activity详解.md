---
title: Activity详解
date: 2018-03-27 10:57:45
tags: android
categories: android
---



## 应用组件

### Activity
#### 什么是Activity

- **Activity 是 Android 的应用程序组件，它提供给用户一个交互式的接口功能**。Activity本身是没有界面的，所以Activity创建了一个窗口，开发人员可以通过 `setContentView(view)` 接口把 UI 放到 Activity 创建的窗口上，当 Activity 指向全屏窗口时，也可以用其他方式实现：作为悬浮窗口，或者嵌入到其他的 Activity 中。**Activity 是单独的，用于处理用户操作**。

#### Activity 的生命周期

![activity_life](..\images\android\activity_life.png)

```java
public class Activity extends ApplicationContext {

     // 活动第一次启动的时候，触发该方法，可以在此时完成活动的初始化工作。
     protected void onCreate(Bundle savedInstanceState);     

     // “用户可见” 状态，表示活动将被展现给用户。
     protected void onStart();     

     // 处于停止状态的活动需要再次展现给用户的时候，触发该方法。
     protected void onRestart();   

     // “用户可交互” 状态，当活动和用户发送交互的时候，触发该方法。
     protected void onResume();    

     /* activity正在停止。
     此时可停止动画或将活动的状态持久化，比如正在编辑的数据库记录等。*/
     protected void onPause();     

     /* activity即将停止，可以做一些稍微重量级的回收工作（不能太耗时）。
     （如果内存吃紧，系统会直接结束这个活动，而不会触发 onStop() 方法）*/
     protected void onStop();      

     /* activity即将被销毁，可以做一些回收工作和最终的资源释放。
     （如果内存吃紧，系统会直接结束这个活动，而不会触发 onDestroy() 方法）*/
     protected void onDestroy();

     /* 系统调用该方法，允许活动保存之前的状态。(如：保存在一串字符串的光标所处位置等。)
     通常情况下，开发者不需要重写该方法，在默认的实现中，已经提供了自动保存所涉及到的用户组件的所有状态信息*/
     protected void onDestroy();
 }
```


#### Activity 的四种启动模式

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

### Service

### BroadcastReceiver

- `onReceive()` 方法中不能进行复杂工作否则会导致 ANR，`onReceive()` 方法一旦执行完，系统可能就认为这个广播接收器已经没用了，随时会杀掉包含这个广播接收器的进程，包括这个进程启动的线程。使用 `goAsync()` 方法可以在 `PendingResult#finish()` 方法执行前为广播接收器的存活争取更多的时间，但最好还是使用 `JobScheduler` 等方式进行长时间处理工作
- `sendBroadcast()`发送常规广播，广播接收器收到的顺序是不可控的。
- `sendOrderedBroadcast()`发送有序广播，根据广播接收器的优先级有序的传递广播，相同优先级的顺序不可控，广播接收器可以选择继续传递给下一个，也可以选择直接丢掉。
- 为了保证广播的`action`全局唯一性，最好声明应用的包名成静态字符串常量作为前缀。

### ContentProvider



## 激活组件

应用不能也不应该直接激活其它应用的任意一个组件，但系统可以，通过发送意图（**Intent**）激活你的组件。

### Intent（意图）

如果**Intent通过组件类名显式指明了唯一的目标组件，那么这个Intent就是显式的**，否则就是隐式，**隐式Intent一般只描述执行动作的类型**，必要时可以携带数据，系统会根据Intent的描述决定激活哪个组件，如果有多个组件符号激活条件，系统一般会弹出选择框让用选择到底激活哪个组件。

### IntentFilter



## 序列化

### Parcelable
Parcelable 对象用来在进程间、Activity 间传递数据，保存实例状态也是用它，Bundle 是它的一个实现，最好只用它存储和传递少量数据，别超过 50k，否则既可能影响性能又可能导致崩溃



## 其它

- `Activity` 可以通过 `getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)` 保持屏幕常亮，这是最推荐、最简单、最安全的保持屏幕常亮的方法，给 view 添加 `android:keepScreenOn="true"` 也是一样的。这个只在这个 `Activity` 生命周期内有效，所以大可放心，如果想提前解除常亮，只需要清除这个 flag 即可。