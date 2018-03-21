---
title: Activity
tags:
  - android
categories: android
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
---



## 什么是Activity

- **Activity 是 Android 的应用程序组件，它提供给用户一个交互式的接口功能**。Activity本身是没有界面的，所以Activity创建了一个窗口，开发人员可以通过 `setContentView(view)` 接口把 UI 放到 Activity 创建的窗口上，当 Activity 指向全屏窗口时，也可以用其他方式实现：作为悬浮窗口，或者嵌入到其他的 Activity 中。**Activity 是单独的，用于处理用户操作**。




## Activity 的生命周期

![Activity 生命流程图](./0_1314838777He6C.gif)

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



### Activity 生命周期的三个嵌套循环：

![Alt text](./1356669508_8478.png)

在Activity生命周期之中，系统调用了App生命周期中的回调方法集，这些生命周期回调方法就像个金字塔。
当系统创建了一个新的Activity实例，回调方法一级一级的从塔底向塔顶移动，当位于金字塔顶部的时候,这个Activity就位于用户前台，用户此时就可以与Activity互动了。
当用户要离开Activity的时候，系统调用另外一串方法，使Activity的状态从塔顶移动到塔底。在有些情况下，Activity只是完成部分的状态迁移并且等待用户的指令，并重新回到塔顶的状态。


根据Activity复杂度的不同，你或许不用实现所有的生命周期方法。可是，**理解每个生命周期回调函数的意义从而确保你的应用按照用户的期望正确的动作则非常重要**。正确的实现生命周期的回调方法，从而使应用正确的动作，主要有以下几点需要注意：
- 确保应用在用户使用你的时候可以接电话或者切换到其他应用而不崩溃。
- 确保你的应用在用户不使用的时候不消耗系统资源。
- 确保用户在从其他的应用切换回你的应用的时候能够继续之前的工作
  在用户屏幕切换或者其他动作的时候不崩溃或者丢失用户数据


**恰当的停止和重启你的Activity是其生命周期中非常重要的过程**，他们保证你的应用对用户来说一致都是可用的，并且不会丢失他们的数据，你需要：
- 当用户从你的应用切换到其他的应用，你的Activity能够正确的停止；当用户从跟其他的应用切换回你的应用，你的应用可以正确的重启。
- 当用户在你的Activity中启动了另外一个新的Activity，当前Activity在新的Activity创建之后停止；当用户点击返回按钮，原先的Activity可以正确重启。
- 当用户在使用你的应用的时候接到一个电话，你的Activity可以正确的停止。

#### 1. Entire LifeTime (整个生命周期)
- 存在于 `onCreate()` 和 `onDestroy()` 调用之间。
- 应该在 `onCreate()` 里执行设置“全局”状态(如定义布局)，并在 `onDestroy()` 里释放所有剩余资源。
>例如：一个正在后台运行下载网络数据的线程，它可以在 onCreate() 中创建该线程，然后在 onDestroy() 中停止线程。

#### 2. Visible LifeTime (可见生命周期)
- 存在于 `onStart()` 和 `onStop()` 调用之间。在此期间，用户可以看到屏幕上的 Activity 并与之交互。
- 当一个其他的 Activity 启动，并且这个 Activity 完全不可见的时候， `onStop()` 方法就会被调用。在两个方法，你可以保持该 Activity 需要展示给用户的资源。
>例如：在 onStart() 里注册一个BroadcastReceiver来监控你的UI变化，并在 onStop() 方法里注销它。

- 在整个生命周期的活动中，系统可能会调用 `onStart()`、`onStop()` 多次，因为活动回见交替进行隐藏或显示给用户。

#### 3. Foreground LifeTime (前台生命周期)
- 存在于 `onResume()` 和 `onPause()` 调用之间。在此期间，这个 Activity 再其他 Activity 的前面，拥有用户输入焦点。
- 一个 Activity 可以经常在前台状态发生转换。
>例如：当设备休眠或弹出对话框。因为经常发生转换，所以在这两个方法之间的代码应该是轻量级的，防止导致其他转换变慢使得用户需要等待。

- 当 Activity 进入 Paused 或 Stopped 状态时，它仍然保持着自身的所有实例和状态在任务返回堆栈中。所以我们考虑的是，**系统在内存不足的情况下，杀死的 Paused 或 Stopped 状态下的 Activity**。

- 当 Activity 在 Resumed 状态下，它是不会因系统内存不足而被直接杀死，只有进入 Paused 或 Stopped 状态下，在此期间如果系统内存不足，系统会直接结束这个 Activity，而不会触发 `onStop()` 和 `onDestory()`，**所以 `OnPause()` 是我们最大程度上保证 Activity 在销毁之前能够执行到的方法**。因此，如果某个 Activity 需要保存某些数据到数据库，应该在 `onPause()` 里编写持久化数据的代码。要注意应选择哪些信息必须保留在 `onPause()`，因为这个方法任何阻塞程序都会阻止过渡到下一个 Activity，这样给用户体验就感觉十分缓慢。




## Activity 堆栈

![Activity 栈](http://upload-images.jianshu.io/upload_images/1810327-875591e5b766ebfd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **每个Activity的状态是由它在Activity栈中的位置决定的(包含所有正在运行Activity的队列)。**

- **一个应用程序的优先级是受最高优先级的Activity影响的。**

- 当决定某个应用程序是否要终结且释放资源时，Android内存管理将使用栈来决定基于Activity的应用程序的优先级。




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

例如: 有三个Activity1、Activity2、Activity3，可相互启动，
其中Activity2为`singeIntent`模式，假设程序从Activity开始运行，
Activity1的`TaskID`为200，
那么当Activity1启动Activity2时，Activity2强会新启动一个Task，
及Activity2与Activity1不在同一个Task中运行。
假设Activity2的`TaskID`为201，再从Activity2启动Activity3时，
Activity3的`TaskID`为200，也就是说它被压到了Activity1启动的任务栈中。

若在Other应用中打开Activity2，假设Other的`TaskID`为200，那么打开Activity，将会新建一个`TaskID`运行，假设`TaskID`为201，如果这是再从Activity2启动Activity1或者Activity3，则会在创建一个Task。因此，若操作步骤为`Other-->Activity2-->Activity1`，这个过程将设计到三个Task。