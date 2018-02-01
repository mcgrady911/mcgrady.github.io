# Android 基础知识

[TOC]

## Android 四大组件
### Activity


#### Activity 与 View 的区别

- activity负责控制部分，view负责显示部分。
- activity中加载相应的view才能显示画面，view是具体的画面布局(layout)，wegit控件组成。
- 每个activity都有对应的view。
- activity是个容器，而view是这个容器里的一员，且只能在这里才能正常工作。



#### Activity 生命周期

![Alt text](./0_1314838777He6C.gif)



Activity 生命周期的三个嵌套循环**：
![Activity 生命周期的三个嵌套循环](./1356669508_8478.png)



其生命周期涉及的函数有**：

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

	 // 当一个正在前台运行的活动因其他活动需要前台运行而转入后台运行时，触发该方法。
	 // (这时需要将活动的状态持久化，比如正在编辑的数据库记录等。)
     protected void onPause();                    

	 // 当活动不需要展示给用户的时候，触发该方法。
	 // 如果内存吃紧，系统会直接结束这个活动，而不会触发 onStop() 方法。
     protected void onStop();                           

	 // 当活动销毁时，触发该方法。
	 // 如果内存吃紧，系统会直接结束这个活动，而不会触发 onDestroy() 方法。
     protected void onDestroy();      

	 // 系统调用该方法，允许活动保存之前的状态。(如：保存在一串字符串的光标所处位置等。)
	 // 通常情况下，开发者不需要重写该方法，在默认的实现中，已经提供了自动保存所涉及到的用户组件的所有状态信息
     protected void onDestroy();                                
 }
```

- **启动Activity**:
  `onCreate()`—>`onStart()`—>`onResume()`，Activity进入运行状态。
- Activity退居后台:
  当前Activity转到新的Activity界面或按Home键回到主屏：
  `onPause()`—>`onStop()`，进入停滞状态。
- **Activity返回前台**:
  `onRestart()`—>**`onStart()`**—>`onResume()`，再次回到运行状态。
- **Activity退居后台，且系统内存不足**
  系统会杀死这个后台状态的Activity（此时这个Activity引用仍然处在任务栈中，只是这个时候引用指向的对象已经为null），若再次回到这个Activity,则会走`onCreate()`–>`onStart()`—>`onResume()`(将重新走一次Activity的初始化生命周期)
- 锁定屏与解锁屏幕
   **只会调用onPause()，而不会调用onStop()方法，开屏后则调用onResume()**

根据Activity复杂度的不同，你或许不用实现所有的生命周期方法。可是，**理解每个生命周期回调函数的意义从而确保你的应用按照用户的期望正确的动作则非常重要**。正确的实现生命周期的回调方法，从而使应用正确的动作，主要有以下几点需要注意：
- 确保应用在用户使用你的时候可以接电话或者切换到其他应用而不崩溃。
- 确保你的应用在用户不使用的时候不消耗系统资源。
- 确保用户在从其他的应用切换回你的应用的时候能够继续之前的工作
  在用户屏幕切换或者其他动作的时候不崩溃或者丢失用户数据


**恰当的停止和重启你的Activity是其生命周期中非常重要的过程**，他们保证你的应用对用户来说一致都是可用的，并且不会丢失他们的数据，你需要：
- 当用户从你的应用切换到其他的应用，你的Activity能够正确的停止；当用户从跟其他的应用切换回你的应用，你的应用可以正确的重启。
- 当用户在你的Activity中启动了另外一个新的Activity，当前Activity在新的Activity创建之后停止；当用户点击返回按钮，原先的Activity可以正确重启。
- 当用户在使用你的应用的时候接到一个电话，你的Activity可以正确的停止。



#### Activity 的四种启动模式

设置Activity的启动模式，需在AndroidManifest.xml中对应的<activity>标签使用
`android:launchMode="standard|singleInstance|singleTask|singleTop"`
来控制Acivity任务栈。

**任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态，当按下back按钮的时候，栈内的Activity会一个一个的出栈，并且调用其`onDestory()`方法。如果栈内没有Activity，那么系统就会回收这个栈，每个APP默认只有一个栈，以APP的包名来命名。

***1. standard***
标准模式。每次激活Activity时都会创建Activity实例，并将其压入任务栈栈顶，而不管这个Activity是否已经存在。Activity的启动三回调(`onCreate()->onStart()->onResume()`)都会执行。

*可有多个相同的实例，也允许多个相同的Activity叠加。*

***2. singleTop***
栈顶复用模式。这种模式下，如果新Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，所以它的启动三回调就不会执行，同时Activity的`onNewIntent()`方法会被回调。如果Activity已经存在但是不在栈顶，那么作用于*standard模式*一样。

*可有多个实例，但不允许多个相同的Activity叠加。*

***3. singleTask***
栈内复用模式。创建这样的Activity的时候，系统会先确认它所需任务栈已经创建，否则先创建任务栈。然后放入Activity，如果栈中已经有一个Activity实例，那么这个Activity就会被调到栈顶，`onNewIntent()`，并且singleTask会清理在当前Activity上面的所有Activity(clear top)。

*在Task栈中将有且只有一个该Activity的实例。*

***4. singleInstance***
加强版的singleTask模式，这种模式的Activity只能单独位于一个任务栈内，由于栈内复用的特性，后续请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了。

*有且只有一个实例，并且这个实例将独立运行在Task中，不允许有别的Activity存在。*

>Activity的堆栈管理以ActivityRecord为单位，所有的ActivityRecord都放在一个List里面。
>可以认为一个ActivityRecord就是一个Activity栈。



***如何退出activity？ 如何安全退出调用多个activity的application？***

`finish()` `killProcess()` `System.exit()` 均可退出。

安全退出调用多个activity的步骤：
1. 创建app继承Application
```xml
<application
	// 声明该类为整个应用程序全局的实例
	android:name=".Application">
```
2. 创建整个应用程序全局实例
```java
	// Application.class
	ArrayList<Activity> activities;
```

3. 其他Activity中
```java
	public void onCreate(Bundle saveInstanceState) {
		App app = (App)getApplication();  // 获取应用程序全局的实例引用
		app.activities.add(this);  // 把当前Activity放入集合中
	}

	public void onDestory() {
			App app = (App)getApplication();  // 获取应用程序全局的实例引用
			app.activities.remove(this);  // 把当前Activity从集合中移除
	}
```

4. 安全退出：
   在菜单中退出按钮事件中定义如下代码：
```java
	App app = (App)getApplication();
	Lsit<Activity> activities = app.activities;
	for (Activity act : activities) {
		act.finish();  // 显式结束
	}
```
***startActivityForResult(Intent intent,int requestCode)***
***onActivityResult(int requestCode, int resultCode,Intent data)***

当你想在Activity中得到新打开Activity关闭后返回的数据，你需要使用系统提供的`startActivityForResult(Intent intent,int requestCode)`方法打开新的Activity，新的Activity关闭后会向前面的Activity传回数据，为了得到传回的数据，你必须在前面的Activity中重写`onActivityResult(int requestCode, int resultCode,Intent data)`方法：
```java
public class MainActivity extends Activity {  
      @Overrideprotected void onCreate(Bundle savedInstanceState) {  
      Button button =(Button)this.findViewById(R.id.button);  
      button.setOnClickListener(new View.OnClickListener(){  
		//点击该按钮会打开一个新的Activity  
        publicvoid onClick(View v) {  
        //第二个参数为请求码，可以根据业务需求自己编号  
        startActivityForResult(new Intent(MainActivity.this, NewActivity.class),  1);  
	    }});  
     }  
     
    //第一个参数为请求码，即调用startActivityForResult()传递过去的值  
    //第二个参数为结果码，结果码用于标识返回数据来自哪个新Activity  
    @Override protected voidonActivityResult(int requestCode, int resultCode, Intent data) {  
    String result =data.getExtras().getString(“result”));//得到新Activity关闭后返回的数据  
    }  
}
```

***如果后台的Activity由于某原因被系统回收了，如何在被系统回收之前保存当前状态？***

当你的程序中某一个ActivityA在运行时，主动或被动地运行另一个新的ActivityB，这个时候A会执行`onSaveInstanceState()`。B完成以后又会来找A，这个时候就有两种情况：
- A被回收，被回收的A就要重新调用`onCreate()`方法，不同于直接启动的是这回`onCreate()`里是带上了参数`savedInstanceState`。
- A没有被回收，没被收回的就直接执行`onResume()`，跳过`onCreate()`了。

若Activity结束前，重写了`onSaveInstanceState()`方法，并将状态数据保存；那么可以在重新调用此Activity时，通过`onCreate()`的`Bundle`恢复状态数据。
```java
@Override
public void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	...
	if (savedInstanceState != null) {
		// Restore the activity state
	}
	...
}
@Override
public void onSaveInstanceState(Bundle savedInstanceState){
	// Store the activity state
}
```

***横竖屏切换时，Activity的生命周期？***

不设置Activity的`android:configChanges`时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次。

设置Activity的`android:configChanges="orientation"`时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次。

设置Activity的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新调用各个生命周期，只会执行`onConfigurationChanged`方法。


***如何将一个Activity设置成窗口的样式？***

在`AndroidManifest.xml中`定义Activity的地方一句话
```xml
// 均可变成半透明
android:theme="@android:style/Theme.Dialog"
android:theme="@android:style/Theme.Translucent"
```


***当启动一个Activity并且新的Activity执行完后需要返回到启动它的Activity来执行的回调函数是？***
`startActivityResult()`


***Activity一般会重载那几个方法用来维护其生命周期？***
`onCreate()` `onStart()` `onDestory()` `onRestart()` `onResume()` `onPause()` `onStop()`


***对一些资源以及状态的操作保存，最好是保存在生命周期的哪个函数中进行？***
`onPause()`


***有关Activity生命周期描述正确的是？***

未设置Activity的`android:configChanges`属性，切换屏幕横纵方向时会重新调用`onCreate()`方法。

当再次启动某个`launchMode`设置为`singletask`的Activity，它的`onNewIntent()`方法会被触发。


#### Activity缓存方法
有a、b两个Activity，当从a进入b之后一段时间，可能系统会把a回收，这时候按back，执行的不是a的onRestart而是onCreate方法，a被重新创建一次，这是a中的临时数据和状态可能就丢失了。

可以用Activity中的`onSaveInstanceState()`回调方法保存临时数据和状态，这个方法一定会在活动被回收之前调用。方法中有一个Bundle参数，`putString()`、`putInt()`等方法需要传入两个参数，一个键一个值。数据保存之后会在`onCreate()`中恢复，`onCreate()`也有一个Bundle类型的参数。

示例代码：
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //这里，当Acivity第一次被创建的时候为空
        //所以我们需要判断一下
        if( savedInstanceState != null ){
            savedInstanceState.getString("anAnt");
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putString("anAnt","Android");
    }
```

***1. onSaveInstanceState (Bundle outState)***
当某个activity变得“容易”被系统销毁时，该activity的**onSaveInstanceState**就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。

注意上面的双引号“容易”？言下之意就是该activity还没有被销毁，而仅仅是一种可能性。这种可能性有哪些？通过重写一个activity的所有生命周期方法，包括`onSaveInstanceState`和`onRestoreInstanceState`方法，我们可以清楚地知道当某个activity（假定为activity A）显示在当前task的最上层时，其`onSaveInstanceState`方法会在什么时候被执行，有这么几种情况：
- **当用户按下HOME键时**
  这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则。
- **长按HOME键，选择运行其他的程序时**
- **按下电源按键（关闭屏幕显示）时**
- **从activity A中启动一个新的activity时**
- **屏幕方向切换时**
  （如果不指定configchange属性） 在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以`onSaveInstanceState`一定会被执行。

总而言之，`onSaveInstanceState`的调用遵循一个重要原则，**即当系统“未经你许可”时销毁了你的activity，则`onSaveInstanceState`会被系统调用**，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。另外，需要注意的几点：

1. 布局中的每一个View默认实现了`onSaveInstanceState()`方法，这样的话，这个UI的任何改变都会自动的存储和在activity重新创建的时候自动的恢复。但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，将不会存储它的状态。

2. 由于默认的`onSaveInstanceState()`方法的实现帮助UI存储它的状态，所以如果你需要覆盖这个方法去存储额外的状态信息时，你应该在执行任何代码之前都调用父类的`super.onSaveInstanceState()`方法 既然有现成的可用，那么我们到底还要不要自己实现`onSaveInstanceState()`？这得看情况了，如果你自己的派生类中有变量影响到UI，或你程序的行为，当然就要把这个变量也保存了，那么就需要自己实现，否则就不需要。

3. 由于`onSaveInstanceState()`方法调用的不确定性，**你应该只使用这个方法去记录activity的瞬间状态（UI的状态）**。**不应该用这个方法去存储持久化数据**。**当用户离开这个activity的时候应该在onPause()方法中存储持久化数据（例如应该被存储到数据库中的数据）**。

4. `onSaveInstanceState()`如果被调用，这个方法会在`onStop()`前被触发，但系统并不保证是否在`onPause()`之前或者之后触发。


***2. onRestoreInstanceState (Bundle outState)***
需要注意的是，`onSaveInstanceState`方法和`onRestoreInstanceState方法`“不一定”是成对的被调用的。

`onRestoreInstanceState`被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的`onRestoreInstanceState`方法不会被执行。

另外，`onRestoreInstanceState`的bundle参数也会传递到`onCreate()`法中，你也可以选择在`onCreate()`方法中做数据还原。 还有`onRestoreInstanceState()`在`onStart()`之后执行。 至于这两个函数的使用，给出示范代码（留意自定义代码在调用super的前或后）：
```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
        savedInstanceState.putBoolean("MyBoolean", true);
        savedInstanceState.putDouble("myDouble", 1.9);
        savedInstanceState.putInt("MyInt", 1);
        savedInstanceState.putString("MyString", "Welcome back to Android");
        // etc.
        super.onSaveInstanceState(savedInstanceState);
}

@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        boolean myBoolean = savedInstanceState.getBoolean("MyBoolean");
        double myDouble = savedInstanceState.getDouble("myDouble");
        int myInt = savedInstanceState.getInt("MyInt");
        String myString = savedInstanceState.getString("MyString");
}
```


### Service
**Service是一个没有用户界面的在后台运行耗时操作的应用组件。**
1. Service 的应用场景
2. Service 进程的优先级

#### Service 的生命周期
![Alt text](./2012050219083160.jpg)


**Service 常用生命周期回调方法如下**：

`onCreate()`该方法在服务被创建时调用，该方法只会被调用一次，无论调用多少次`startService()`或`bindService()`方法，服务也只被创建一次。

`onDestroy()`该方法在服务被终止时调用。


#### Service 的启动方式
5. Service 是如何向 Activity 传递数据的
- **在Activity里注册一个BroadcastReceiver，Service完成某个任务就可以发一个广播，接收器收到广播后通知activity做相应的操作**。
- **使用bindService来关联Service和Application**，应用`.apk`里的所有组件一般情况都运行在同一个进程中，所以不需要用到`IPC`，`bindService`成功后，Service的Client可以得到Service返回的一个iBinder引用，具体的参见Service的文档及`onBind`的例子，这样Service的引用就可以通过返回的iBinder对象得到，如：
```java
public class LocalService extends Service {
	// This is the object that receives interactions from clients. See
	// RemoteService for a more complete example.
	private final IBinder mBinder = new LocalBinder();

	public class LocalBinder extends Binder {
		LocalService getService() {
			return LocalService.this;
		}
	}

	@Override
	public IBinder onBind(Intent intent) {
		return mBinder;
	}
}
```

之后Client通过这个iBinder对象得到Service对象引用之后，可以直接和Service通讯，比如读取Service中的值或是调用Service的方法。


***用什么方式使 Service 和Activity 之间传递长久保存的对象？***


***`Context.bindService()`启动Service有关的生命周期方法？***
- 第一步：继承Service类 `public class SMSServiceextendsService{}`
- 第二步：在`AndroidManifest.xml`文件中的`<application>`节点里对服务进行配置:
```xml
<serviceandroid:name=".DemoService"/>
```
- 使用`startService()`方法启用服务，调用者与服务之间没有关连，即使调用者退出了，服务仍然运行。
- 在服务未被创建时，系统会先调用服务的`onCreate()`方法，接着调用`onStart()`方法。如果调用`startService()`方法前服务已经被创建（多次调用`startService()`方法并不会导致多次创建服务，但会导致多次调用`onStart()`方法）。
- 只有调用`Context.stopService()`方法结束服务，服务结束时会调用`onDestroy()`方法。
- 使用`bindService()`方法启用服务，调用者与服务绑定在了一起，调用者一旦退出，服务也就终止。当调用者与服务已经绑定，多次调用`Context.bindService()`方法并不会导致该方法被多次调用。
- `onUnbind()`只有采用`Context.bindService()`方法启动服务时才会回调该方法。该方法在调用者与服务解除绑定时被调用，调用该方法也会导致系统调用服务的`onUnbind()`-->`onDestroy()`方法。


***Android关于Service生命周期的`onCreate()`和`onStart()`说法正确的是？***
- 当第一次启动的时候先后调用`onCreate()`和`onStart()`方法
- 如果Service已经启动，只会执行`onStart()`方法，不在执行`onCreate()`方法


***下列关于IntentService与Service的关系描述错误的是？***
- IntentService是Service的子类
- IntentService在运行时会启动新的线程来执行任务
- `启动方式不同`
- `没有区别`


***IntentService的使用场景与特点？***
- Acitivity的进程，当处理Intent的时候，会产生一个对应的Service
- Android的进程处理器现在会尽可能的不kill掉你
- 非常容易使用

**IntentService**是Service的子类，是一个异步的、会自动停止的服务，很好解决了传统的Service中处理完耗时操作忘记停止并销毁Service的问题，与Service不同的是，IntentService在执行`onCreate`操作的时候，内部开了一个线程，去你执行你的耗时操作。

**优点**
- 一方面不需要自己去new Thread
- 另一方面不需要考虑在什么时候关闭该Service

>onStartCommand()中回调了onStart()，onStart()中通过mServiceHandler发送消息到该handler的handleMessage中去。最后handleMessage中回调onHandleIntent(intent)。


***Service的两种启动方法，有什么区别？***
1.在Context中通过`public boolean bindService(Intent service,ServiceConnection conn,int flags)`方法来进行Service与Context的关联并启动，并且Service的生命周期依附于Context。

2.通过`public ComponentName startService(Intent service)`方法去启动一个Service，此时Service的生命周期与启动它的Context无关。

要注意的是**都需要在xml里注册你的Service**
```
<service
	android:name=".packnameName.youServiceName"
	android:enabled="true" />
```


### BroadcastReceiver

***BroadcastReceiver 的应用场景***


***BroadcastReceiver 的生命周期***
![Alt text](./d4150c0f-c2e1-3b04-83a9-f1b1757ff4fb.jpg)

**生命周期只有10s左右，如果在 onReceive() 内做超过10s的事情，就会报错 。**

每次广播到来时 , 会重新创建BroadcastReceiver对象，并且调用`onReceive()`方法，执行完以后，该对象即被销毁。当`onReceive()`方法在10s内没有执行完毕，Android 会认为该程序无响应。
所以在BroadcastReceiver里不能做一些比较耗时的操作，否侧会弹出ANR。


***什么情景下用到广播？***
- Android中的广播事件有两种：
  - 系统广播事件，如：
    - ACTION_BOOT_COMPLETED（系统启动完成后出发）
    - ACTION_TIME_CHANGED（系统时间改变时出发）
    - ACTION_BATTERY_LOW（低电量时出发）
    - 拍摄了一张照片
  - 开发者自定义的广播事件，如：
    - 通知其他应用程序一些数据下载完成并处于可用状态
- 应用程序可以拥有任意数量的广播接收器以对所有它感兴趣的通知信息予以响应
- 通知可以用很多种方式来吸引用户的注意力，如：
  - 闪动背光
  - 震动
  - 播放声音


***Android引入广播机制的用意？***

从MVC的角度考虑(应用程序内)


***注册广播有几种方式，这些方式有何区别？***
- 静态注册：在AndroidManifest.xml文件中进行注册，当App退出后，Receiver仍然可以接收到广播并且进行相应的处理。
- 动态注册：在代码中动态注册，当App退出后，也就没办法再接受广播。

```java
// 1. 代码动态注册：
// 生成广播处理
smsBroadCastReceiver = new SmsBroadCastReceiver();
//实例化过滤器并设置要过滤的广播
IntentFilter intentFilter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
//注册广播
BroadCastReceiverActivity.this.registerReceiver(smsBroadCastReceiver,intentFilter);


// 2. 在AndroidManifest.xml中配置广播
<!--广播注册-->
<receiverandroid:name=".SmsBroadCastReceiver">
	<intent-filterandroid:priority="20">
		<actionandroid:name="android.provider.Telephony.SMS_RECEIVED"/>
	</intent-filter>
</receiver>

<!--权限申请-->
<uses-permissionandroid:name="android.permission.RECEIVE_SMS"></uses-permission>
```


### ContentProvider

***请介绍下ContentProvider是如何实现数据共享的？***
- 创建一个属于你自己的ContentProvider或者将你的数据添加到一个已经存在的Contentprovider中，前提是有相同数据类型并且有写入ContentProvider的权限。


***在多个应用中读取共享存储数据时，需要用到的query方法，是哪个对象的方法？***
- `ContentResolver`
  - 运行在提供数据端，`ContentResolver`一个运行在调用端。
  - **在多个应用中读取共享存储数据，只能用`ContentResolver`读取数据**


***简要解释一下Activity、Intent、IntentFilter、Service、Broadcase、BroadcaseReceiver？***
- Activity呈现了一个用户可以操作的可视化用户界面
- Service不包含可见的用户界面，而是在后台无限地运行，可以连接到一个正在运行的服务中，连接后，可以通过服务中暴露出来的借口与其进行通信
- BroadcastReceiver是一个接收广播消息并作出回应的component，broadcastreceiver没有界面
- Intent：ContentPprovider在接收到ContentResolver的请求时被激活
- Activity、Sservice和Broadcastreceiver是被称为Intent的异步消息激活的。
- 对于Activity和Service来说，Intent指定了请求的操作名称和待操作数据的URI
- Intent对象可以显式的指定一个目标component。如果这样的话，android会找到这个component(基于manifest文件中的声明)并激活它。但如果一个目标不是显式指定的，android必须找到响应intent的最佳component。
- 它是通过将Intent对象和目标的intentfilter相比较来完成这一工作的。一个component的intentfilter告诉android该component能处理的intent。intentfilter也是在manifest文件中声明的。



## 数据库相关

***如何将打开`resaw`目录中的数据库文件？***
- 在Android中不能直接打开`resaw`目录中的数据库文件，而需要在程序第一次启动时将该文件复制到手机内存或SD卡的某个目录中，然后再打开该数据库文件。
- 复制的基本方法是使用`getResources().openRawResource`方法获得`resaw`目录中资源的`InputStream`对象，然后将该`InputStream`对象中的数据写入其他的目录中相应文件中。
- 在AndroidSDK中可以使用`SQLiteDatabase.openOrCreateDatabase`方法来打开任意目录中的SQLite数据库文件。


***什么时候用到数据库？***
- 本地数据库SQLite主要作用是存储和保留网络数据，保证无网络数据或网络异常下能够使用，同时减少网络流量的损耗。
- 如果数据是已知的、静态的，可以在本地SQLite中存储、读取。这样不会因网络问题而降低效率和成功率。
- 如果数据未知、有实时的变化或者有与其他用户交互、共享的数据必然需要后台服务器数据。



***Android数据存储的方式有哪几类？***

**SharedPreferences**
- 适用范围：保存少量的数据，且这些数据的格式非常简单，如：字符串、基本类型的值。比如：应用程序的各种配置信息（是否打开音效、是否使用震动效果、小游戏玩家积分等）、解锁口令密码等
- 核心原理：保存给予XML文件存储的`key-value`键值对数据。
  SharedPreferences本身是一个接口，程序无法直接创建实例，只能通过`Context`提供的`getSharedPreferences(String name, int mode)`方法获取实例；且只能获取数据而不支持存储和修改，存储修改时通过`SharedPreferences.edit()`获取内部接口`Editor`对象实现。

**文件存储数据**
关于文件存储，Activity提供了openFileOutput()方法可以用于把数据输出到文件中，具体的实现过程与在J2SE环境中保存数据到文件中是一样的。
- 适用范围：文件可用来存放大量的数据，如：文本、图片、音频等。
- 核心原理: Context提供了两个方法来打开数据文件里的文件IO流：
  - `FileInputStream openFileInput(String name)`
  - `FileOutputStream(String name , int mode)`

**SQLite**
SQLite是轻量级嵌入式数据库引擎，它支持 SQL 语言，并且只利用很少的内存就有很好的性能。
![Alt text | 500*300](./18beb977b2de155e7e422ca2d78f73a9.jpg)
- 特点：
  - 面向资源有限的设备
  - 没有服务器进程
  - 所有数据存放在同一文件中跨平台
  - 可自由复制

**ContentProvider**
- 数据在Android当中是私有的，当然这些数据包括文件数据和数据库数据以及一些其他类型的数据。
- Content Provider提供了一种多应用间数据共享的方式，比如：联系人信息可以被多个应用程序访问。
- 标准的Content Provider：
  - Android提供了一些已经在系统中实现的标准Content Provider，比如联系人信息，图片库等等，你可以用这些Content Provider来访问设备上存储的联系人信息，图片等等。

**网络存储数据**
- 可以调用WebService返回的数据或是解析HTTP协议实现网络数据交互。
- 可以通过地区名称查询该地区的天气预报，以POST发送的方式发送请求到webservicex.net站点，访问WebService.webservicex.net站点上提供查询天气预报的服务。


***如何将SQLite数据库（`dictionary.db`文件）与apk文件一起发布？***
- 可以将dictionary.db文件复制到Eclipse Android工程中的`resaw`目录中。所有在resaw目录中的文件不会被压缩，这样可以直接提取该目录中的文件。


***请继承`SQLiteOpenHelper`实现：***
- 创建一个版本为1的`diaryOpenHelper.db`的数据库，同时创建一个`diary`表（包含一个`_id`主键并自增长，`topic`字符型100长度，`content`字符型1000长度）
- 在数据库版本变化时请删除`diary`表，并重新创建出`diary`表。
```java
public class DBHelperextends SQLiteOpenHelper{

	public final static String DATABASE NAME = "diaryOpenHelper.db";
	public final static int DATABASE VERSION=1;

	//创建数据库
	public DBHelper(Context context,String name,CursorFactory factory,int version) {
		super(context,name,factory,version);
	}
	//创建表等机构性文件
	public void onCreate(SQLiteDatabase db) {
		String sql="createtablediary"+
				"("+
				"_id integer primary key auto increment,"+
				"topic varchar(100),"+
				"content varchar(1000)"+
				")";
		db.execSQL(sql);
	}
	//若数据库版本有更新，则调用此方法
	public void onUpgrade(SQLiteDatabase db,int oldVersion,int newVersion) {
		String sql = "drop table ifexist sdiary";
		db.execSQL(sql);
		this.onCreate(db);
	}
}
```


***在Android中使用`SQLiteOpenHelper`这个辅助类时，可以生成一个数据库，并可以对数据库版本进行管理的方法可以是？***
- `getWriteableDatabase()`
- `getReadableDatabase()`


## 进程相关

### AsyncTask
***AsyncTask 的实现原理和使用的优缺点？***


### Handler
***Handler 异步实现的原理和使用的优缺点***


***Handler 的机制原理？*** 
- **Andriod提供了Handler和Looper来满足线程间的通信**。
- Handler：构造Handler对象来与Looper沟通，以便push新消息到MessageQueue里;或者接收Looper从MessageQueue取出所送来的消息。
  - Looper：用来管理特定线程内对象之间的消息交换(MessageExchange)。一个线程可以产生一个Looper对象，由它来管理此线程里的MessageQueue(消息队列)。
  - MessageQueue(消息队列)：用来存放线程放入的消息。


***请解释下在单线程模型中Message、Handler、MessageQueue、Looper之间的关系？***
- 简单地说：Handler获取当前线程中的Looper对象，而Looper用来存放MessageQueue中取出的Message，再由Handler进行Message的分发和处理，按照先进先出执行。
- MessageQueue（消息队列）：用来存放通过Handler发布的消息，通常附属于某一个创建它的线程，可以通过`Looper.myQueue()`得到当前线程的消息队列。
- Handler：可以发布和处理一个Message或者操作一个Runnable，通过Handler发布消息，Message将只会发送到与它关联的MessageQueue，然而也只能处理该MessageQueue中的消息
- Looper：是Handler和消息队列之间的通讯桥梁，程序组件首先通过Handler把消息传递给Looper，然后Looper把Message放入MessageQueue。
- Message：消息的类型，在Handler类中的`handleMessage`方法中获取单个消息进行处理、
- 在单线程模型下，为了解决线程通信问题，Android设计了MessageQueue，线程间可以通过该MessageQueue并结合Handler和Looper组件进行信息交换。


***Android系统对对象提供了资源池有哪些？***
- `Message`
  - 提供了消息池，有静态方法Obtain从消息池中取对象
- `AsyncTask`
  - 是线程池改造的，池里 默认提供（核数+1）个线程进行并发操作，最大支持（核数  * 2 + 1）个线程，超过后会丢弃其他任务


***为满足线程间通信，Android提供了？ ***. 
- `Handler`和`Looper`


***遇到下列哪种情况时需要把进程移到前台？*** 
- 进程正在运行一个与用户交互的Activity ，它的onResume()方法被调用
- 进程有一正在运行的BroadcastReceiver，它的onReceive()方法正在执行
- 进程有一个Service，并且在Service的某个回调函数（onCreate()、onStart()、或onDestroy()）内有正在执行的代码
- 进程有一个Service，该Service对应的Activity正在与用户交互

![Alt text](./389768_1460998773645_D83E313AA1780C3549DC4D610DE7601F.png)


***我们都知道Hanlder是线程与Activity通信的桥梁，如果线程处理不当，你的机器就会变得越慢，那么线程销毁的方法是？***
- `onDestroy()`


***下面哪种进程最重要，最后被销毁？***
- 服务进程
- 后台进程
- 可见进程
- `前台进程`

重要性依次是：前台进程--->可见进程--->服务进 --->后台进程和空进程
所以销毁的顺序是逆方向。


***AndroidDvm的进程和Linux的进程，应用程序的进程是否为同一个概念？***
- DVM指`Dalvik`虚拟机，每一个Android应用程序都在它自己的进程中运行，都拥有一个独立的`Dalvik`虚拟机实例。而每一个DVM都是在Linux中的一个进程，所以说可以认为是同一个概念。


***谈谈Android的IPC（进程间通信）机制***
- IPC是内部进程通信的简称，是共享"命名管道"的资源。
- 是为了让Activity和Service之间可以随时的进行交互，故在Android中该机制，只适用于Activity和Service之间的通信，类似于远程方法调用，类似于C/S模式的访问。通过定义AIDL接口文件来定义IPC接口。Servier端实现IPC接口，Client端调用IPC接口本地代理。


## 系统相关
***简述Android应用程序结构是哪些？***
- LinuxKernel (Linux内核)
- Libraries (系统运行库或者是c/c++核心库)
- Application Framework (开发框架包)
- Applications (核心应用程序)


***下列哪些情况下系统会弹出Froce Close（强制关闭）对话框？如何避免？能否捕获导致其的异常？***
- 应用运行时抛出了OutOfMemoryError
- 应用运行时抛出了RuntimeException
- 一般像空指针啊，可以看起`logcat`，然后对应到程序中来解决错误


***下列哪些情况下，系统可能会弹出ANR对话框？***
- 在Activity中，Main线程消息队列中的消息在5秒内没有得到响应
- 在BroadcastReceiver中，onReceive()方法执行时间超过10秒


***什么是ANR？如何避免？***

**ANR(Application Not Responding) 应用程序无响应**
在 Android 上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应 ANR。用户可以选择“等待”而让程序继续运行，也可以选择“强制关闭”。默认情况下，在 android 中 Activity 的最长执行时间是 5 秒，BroadcastReceiver 的最长执行时间则是 10 秒。
- 三种常见类型：
  - **KeyDispatchTimeout (5s) `主要类型`**
    按键或触摸事件在特定时间内无响应
  - **BroadcastTimeout (10s)**
    BroadcastReceiver 在特定时间内无法处理完成
  - **ServiceTimeout (20s) `小概率类型`**
    Service 在特定的时间内无法处理完成


***Android本身的api并未声明会抛出异常，则其在运行时有无可能抛出Runtime异常，你遇到过吗?诺有的话会导致什么问题?如何解决?***
- 比如nullpointerException
- 比如`textview.setText()`时，`textview`没有初始化。会导致程序无法正常运行出现ForceClose。打开控制台查看logcat信息找出异常信息并修改程序。



***如何避免应用闪退？***


***常见的Java Exception（RuntimeException）异常***
1. **NullPointException  空指针引用异常**  
2. **ClassCastException  类型强制转换异常**  
3. **IllegalArgumentException  传递非法参数异常**  
4. **ArithmeticException  算术运算异常**
5. ArrayStoreException  向数组中存放与声明类型不兼容对象异常
6. **IndexOutOfBoundsException  下标越界异常**
7. NegativeArraySizeException  创建一个大小为负数的数组错误异常
8. **NumBerFormatException  数字格式异常**
9. SecurityException  安全异常
10. UnsupportedOperationException  不支持的操作异常


***下列关于数据持久化的描述正确的有？***

数据持久化就是将内存的数据保存到外存


***Android 数据持久化主要有几种？***
1. 保存到Shared Preferences
2. 保存到手机内存
3. 保存到SDCard中
4. 保存到SQlite数据库中


***请解释下Android程序运行时权限与文件系统权限的区别。运行时权限***
- Dalvik(android授权)
- 文件系统linux内核授权


***系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。***

通过直接发送Uri把参数带过去，或者通过manifest里的intentfilter里的data属性。


***Android数字签名，下列关于 Android数字签名描述正确的是？***
-  Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
-  如果要正式发布一个Android程序，可以使用集成开发工具生成的调试证书来发布。

**Android通过数字签名来标识应用程序的作者和应用程序之间建立信任关系，这个数字签名由应用程序的作者完成。**
>注意：Android系统不会安装任何一款未经数字签名的apk程序。

- Apk程序的两种模式：
  - 调试模式（Debug Mode）
    - 在调试模式下， ADT会自动的使用debug密钥为应用程序签名，因此我们可以直接运行程序。
      debug密钥：一个名为`debug.keystore`的文件
      存放位置 ：`C:\Users\username\.android\debug.keystore`
    - Debug签名的应用程序有这样两个风险：
    1. debug签名的应用程序不能在Android Market上架销售，它会强制你使用自己的签名。
    2. debug.keystore在不同的机器上所生成的可能都不一样，就意味着如果你换了机器进行apk版本升级，那么将会出现上面那种程序不能覆盖安装的问题。
       `注意：不可忽视的问题！如果你开发的程序有很多使用客户，就相当于软件不具备升级功能，所以一定要有自己的数字证书来签名！`

  - 发布模式（Release Mode）
    -  **当要发布程序时，开发者需要使用自己的数字证书给apk包签名**，有以下两种方法：
    1. 通过DOS命令来对APK签名。
    2. 使用ADT Export Wizard进行签名

11. 请使用命令行的方式创建一个名字为myAvd，SDK版本为2.2，SD卡是在D盘的根目录下，名字为`scard.img`，并指定屏幕大小HVGA。
- `androidcreateavd-tandroid-8-nmyAvd-cd:/avd/scard.img--skin480*320`。


***MVC模式在Android中的具体体现？***
![Alt text | 500*300](./20150605112142444.jpg) ![Alt text | 300*300](./MVC-Process.png)

**视图层（View）**：
- 视图层实现数据的显示，一般没有程序上的逻辑，为了实现试图上的刷新功能，是投入需要访问它的监视数据模型（Model）,因此应该事先在被它监视的数据那里注册。
- 一般采用xml文件进行界面的描述，使用的时候可以非常方便的引入（也可通过java 和 javascript 通信，使用 javascript + html 等方式作为view层）。

**控制层（Controller）**：
- **控制器起到了不同层面间的组织作用，用于控制应用程序的流程**。负责转发事件，对事件进行处理并且响应，“事件”包括用户的行为和数据模型上的改变。
- android的重支持的重任通常落在众多的activity的肩上，这句话也就暗含了不要再activity中写代码，要通过activity交割model业务逻辑层处理，这样做的另外一个原因是android中的activity的响应时间是5s，如果耗时操作放在这里，程序很容易被回收掉。
3. 模型层（model）：
- 用于封装与应用程序的业务逻辑相关数据以及对数据的处理方法。如：数据库操作、网络操作、业务计算等操作。
- “Model 不依赖 View 和 Controller”，模型不关心它会被如何显示和操作。但模型中数据的变化一般会通过一种刷新机制被公布。为了实现这种机制，那些用于监视次模型的试图必须事先在此模型上注册，从而，**视图可以了解在数据模型上发生的改变。**



***5.2 以下哪些设计模式是需要的？***

一个系统，提供多个http协议的接口，返回的结果Y有json格式和jsonp格式。Json的格式为`{"code":100,"msg":"aaa"}`,为了保证该协议变更之后更好的应用到多个接口，为了保证修改协议不影响到原先逻辑的代码，以下哪些设计模式是需要的?协议的变更指的是日后可能返回xml格式，或者是根据需求统一对返回的消息进行过滤?

- `Aadapter`
- `factory method（工厂模式）`
- proxy（代理模式）
- `decorator（装饰zhe）`
- composite（组合模式）

**Aadapter**：
- 新增功能但不能修改原来代码，原来代码实现思路——标准接口 Target 定义 interface， ConcreteTarget 就是当前解析 json 的类（实现 Target接口 ）。
- 新增功能这样实现—— Adaptee 是新增功能的所属类，Adapter 实现 Target 接口并集成 Adaptee，这样的 Adapter就有了新的功能了，因此需要适配器模式。调用实例如下：
```java
public static void main(String[] args) {
	// 使用普通功能类
	Target concreteTarget = new ConcreteTarget();
	concreteTarget.request();
	// 使用特殊功能类，即适配类
	Target adapter = new Adapter();
	adapter.request();
}
```
**工厂模式 (Factory Method )**：
- 为多个http协议的接口，在客户端代码中，告诉要请求的接口名称，会调用不同的类来处理，显然是工厂方法

**装饰者模式 (composite)**：
- 是用来动态添加功能的，就是过滤 消息，比如非法字符&&&之类的，消息过长之类


#### NDK

 **NDK是一些列工具的集合**，NDK提供了一系列的工具，帮助开发者迅速的开发C/C++的动态库，并能自动将so和java应用打成apk包。

**NDK集成了交叉编译器**，并提供了相应的mk文件和隔离cpu、平台等的差异，开发人员只需简单的修改mk文件就可以创建出so。


***AIDL的全称是什么？如何工作？能处理哪些类型的数据？***
- Android Interface Define Language
- 当A进程要去调用B进程中的service时，并实现通信，通常是通过AIDL来操作的
- A工程：首先我们在net.blogjava.mobile.aidlservice包中创建一个RemoteService.aidl文件，在里面我们自定义一个接口，含有方法get。ADT插件会在gen目录下自动生成一个RemoteService.java文件，该类中含有一个名为RemoteService.stub的内部类，该内部类中含有aidl文件接口的get方法。说明一：aidl文件的位置不固定，可以任意。然后定义自己的MyService类，在MyService类中自定义一个内部类去继承RemoteService.stub这个内部类，实现get方法。在onBind方法中返回这个内部类的对象，系统会自动将这个对象封装成IBinder对象，传递给他的调用者。
  其次需要在`AndroidManifest.xml`文件中配置MyService类，代码如下：
```xml
<!--注册服务-->
<serviceandroid:name=".MyService">
	<intent-filter>
		<!--指定调用AIDL服务的ID-->
		<actionandroid:name="net.blogjava.mobile.aidlservice.RemoteService"/>
	</intent-filter>
</service>
```
- 为什么要指定调用AIDL服务的ID,就是要告诉外界MyService这个类能够被别的进程访问，只要别的进程知道这个ID，正是有了这个ID,B工程才能找到A工程实现通信。说明：AIDL并不需要权限
- B工程：首先我们要将A工程中生成的RemoteService.java文件拷贝到B工程中，在bindService方法中绑定AIDL服务（emoteService的ID作为intent的action参数。）说明：如果我们单独将`RemoteService.aidl`文件放在一个包里，那个在我们将gen目录下的该包拷贝到B工程中。如果我们将`RemoteService.aidl`文件和我们的其他类存放在一起，那么我们在B工程中就要建立相应的包，以保证`RmoteService.java`文件的报名正确，我们不能修改`RemoteService.java`文件
```java
bindService(new Inten("net.blogjava.mobile.aidlservice.RemoteService"),serviceConnection,Context.BIND_AUTO_CREATE);
```
ServiceConnection的`onServiceConnected(ComponentNamename,IBinderservice)`方法中的service参数就是A工程中MyService类中继承了`RemoteService.stub`类的内部类的对象。


***使用AIDL完成远程Service方法调用下列说法正确的是？***
- AIDL的文件的内容类似java代码
- 创建一个Service，在Service的`onBind(Intent intent)`方法中返回实现AIDL接口的对象
- AIDL对应的接口的方法前面不能加访问权限修饰符


**Android 使用AIDL提供公开服务接口，使得不同进程间可以相互通信。**
- AIDL 对应的接口名称必须与AIDL文件名相同不然无法自动编译
- AIDL 对应的接口方法不能加访问权限修复
- 建立AIDL服务要比建立普通的服务复杂一些，具体步骤如下：
1. 在Eclipse Android工程的Java包目录中建立一个扩展名为aidl的文件。该文件的语法类似于Java代码，但会稍有不同。
2. 如果aidl文件的内容是正确的，ADT会自动生成一个Java接口文件（*.java）。
3. 建立一个服务类（Service的子类）。
4. 实现由aidl文件生成的Java接口。
5. 在AndroidManifest.xml文件中配置AIDL服务，尤其要注意的是，`<action>`标签中`android:name`的属性值就是客户端要引用该服务的ID，也就是Intent类的参数值。


***请简述JNI在Android中的调用过程？***
- 安装和下载Cygwin，下载AndroidNDK
- 在ndk项目中JNI接口的设计
- 使用C/C++实现本地方法
- JNI生成动态链接库.so文件
- 将动态链接库复制到java工程，在java工程中调用，运行java工程即可


***SIM卡的EF文件有何作用？***
- SIM卡的文件系统有自己规范，主要是为了和手机通讯，SIM本身可以有自己的操作系统
- EF就是作存储并和手机通讯用的


***什么是嵌入式实时操作系统？Android操作系统属于实时操作系统吗？***
- 嵌入式实时操作系统，是指当外界事件或数据产生时，能够接受并以足够快的速度予以处理，其处理的结果又能在规定的时间之内来控制生产过程或对处理系统作出快速响应，并控制所有实时任务协调一致运行的嵌入式操作系统。
- 实时系统，主要用于工业控制、军事设备、航空航天等领域对系统的响应时间有苛刻的要求。（又可分为软实时和硬实时两种）
- 而Android是基于Linux内核的，因此属于软实时。


***一条最长的短信息约占多少byte？***
- 中文70(包括标点)，英文160，160个字节。


***如何防止apk包被反编译？***
- 代码混淆：java代码混淆工具 `proguard`
- 二次打包


***关于ContenValues类说法正确的是？***
- 和`Hashtable`比较类似，也是负责存储一些名值对，但是他存储的名值对当中的名是String类型，而值都是基本类型


***关于`res/raw`目录说法正确的是？***
- 这里的文件是原封不动的存储到设备上不会转换为二进制的格式


***下列属于SAX解析xml文件的优点的是？***
- 不用事先调入整个文档，占用资源少


***DDMS和TraceView的区别？***
- DDMS是一个程序执行查看器，在里面可以看见线程和堆栈等信息，TraceView是程序性能分析器。


***什么是Context？***
- **它描述的是一个应用程序环境的消息，即上下文**。
- 该类是一个抽象（abstract class）类。Android提供了该抽象类的集体实现类。
- 通过它我们可以获取应用程序的资源和类，也包括一些应用级别操作。例如：启动一个Activity，发送广播，接收Intent信息等。
- 总结：**Context是一个抽象基类**，我们通过它访问当前包的资源(getResources, getAssets)和启动其他组件(Activity, Service, Broadcast)以及得到各种服务(getSystemService)，当然，通过Context提供了一个应用的运行环境，在Context定义了一套基本的功能接口，我们可以理解为一套规范，而Activity和Service是实现这套规范的子类，这么说也许并不准确，因为这套规范实际是被ContextImpl类统一实现的，Activity和Service只是继承并有选择性地重写了某些规范的实现。

Context相关类的继承关系：
![Alt text](./Image.png)
 通过上图可以看出，Activity类、Service类、Application类本质上都是Context的子类。


***Application、Activity、Service作为Context的区别?*** 
- 相同点：它们都简介继承了Context
- 不同点：首先看它们的继承关系，通过对比可以发现，Activity比Service和Application多了一层继承关系ContextTemeWrapper，这是因为Activity有主题概念，而Service没有界面服务，Application更是一个抽象的东西，它也是通过Activity类呈现的。Context的真正实现都在ContextImpl中，也就是说Context的大部分方法调用都会转到ContextImpl中，而三者的创建均在ActivityThread中完成，**Activity启动的核心过程是在ActivityThread中完成的**，这里要说明的是，**Application和Service的创建也是在ActivityThread中完成的**。


***一个应用程序中有多少Context？***
总Context实例个数 = Service个数 + Activity个数 + 1(Application对应的Context实例)


## 组件相关
### Intent
**Intent 可在同一个app中不同组件之间传递信息。**


***Intent传递数据时，下列的数据类型哪些可以被传递？***
- Serializable
- Charsequence
- Parcelable
- Bundle


***Android中下列属于Intent的作用的是？***
- 可以实现界面间的切换，可以包含动作和动作数据，连接四大组件的纽带


***关于Intent的说法，正确的是？***
- 可以用来激活一些组件
- 表示程序想做某事的意图
- 是一个简单的消息对象


***在Android中使用Menu时可能需要重写的方法有？***
- `onCreateOptionsMenu()`
- `onOptionsItemSelected()`


***页面上现有`ProgressBar`控件，请用书写线程以10秒的的时间完成其进度显示工作。***
```java
private Progress BarprogressBar = null;
progressBar = (ProgressBar)findViewById(R.id.progressBar);
Threadthread = new Thread(new Runnable(){
	@Override
	public void run() {
		intprogressBarMax = progressBar.getMax();
		try{
			while(progressBarMax != progressBar.getProgress()) {
				intstepProgress = progressBarMax/10;
				intcurrentprogress = progressBar.getProgress();
				progressBar.setProgress(currentprogress + stepProgress);
				Thread.sleep(1000);
			}
		} catch(InterruptedExceptione) {
			//TODOAuto-generatedcatchblock
			e.printStackTrace();
		}
	}
});

thread.start();
```

***下面是属于GLSurFaceView特性的是？***
-  管理一个surface，这个surface就是一块特殊的内存，能直接排版到android的视图view上。
-  管理一个EGL display，它能让opengl把内容渲染到上述的surface上。
-  让渲染器在独立的线程里运作，和UI线程分离。

    **GLSurfaceView**类，具有以下特点 ：
   1. 管理一个平面，这个平面是一个特殊的内存块，它可以和 android 视图系统混合 。
   2. 管理一个EGL 显示 ，它能够让 OpenGL 渲染到一个平面 。
   3. 接受一个用户提供的实际显示的Renderer 对象 。
   4. 使用一个专用线程去渲染从而和UI 线程解耦 。
   5. 支持on-demand 和连续的渲染。
   6. 可选的包, 追踪 和 / 或者错误检查这个渲染器的 OpenGL 调用 。


***Toask的显示时长***
- Toast的默认显示时间有两个，分别为 :`Toast.LENGTH_SHORT` `(2s)` 和 `Toast.LENGTH_LONG` `(3.5s)`


## UI相关

***下面的对自定style的方式正确的是***
```xml
<resources>
<stylename="myStyle">
<itemname="android:layout_width">fill_parent</item>
</style>
</resources>
```


***常用的五种布局***
- FrameLayout（框架布局）
- LinearLayout（线性布局）
- AbsoluteLayout（绝对布局）
- RelativeLayout（相对布局）
- TableLayout（表格布局）


***在 Android 中， 在屏幕密度为160时， 1pt 大概等于__sp？***
- 在Android中，`1pt`大概等于`2.22sp`以上供参考，与分辨率无关的度量单位可以解决这一问题。
- Android支持下列所有单位：
  - `px（像素）`：屏幕上的
  - `in（英寸）`：长度
  - `mm（毫米）`：长度
  - `pt（磅）`：1/72
  - `dp（与密度无关的像素）`：一种基于屏幕密度的抽象单位。在每英寸160点的显示器上，`1dp = 1px` 
  - `dip`：与dp相同，多用于android/ophone示例
  - `sp（与刻度无关的像素）`：与dp类似，但是可以根据用户的字体大小首选项进行缩放。

>分辨率：整个屏是多少点，比如800x480，它是对于软件来说的显示单位，以`px`为单位的点。 density(密度)值表示每英寸有多少个显示点，与分辨率是两个概念。

- apk的资源包中：
  - 当屏幕density=240时，使用`hdpi`标签的资源  
  - 当屏幕density=160时，使用`mdpi`标签的资源  
  - 当屏幕density=120时，使用`ldpi`标签的资源
- 一般android设置`长度和宽度多用dip`,设置`字体大小多用sp`. 
- 在屏幕密度为160时，`1dp=1px=1dip`  `1pt = 160/72 sp` `1pt = 1/72 英寸`.
- 当屏幕密度为240时，`1dp=1dip=1.5px`


***属于Android 的动画类型有？***
- Android的动画类型有：
  - Property Animation（属性动画）
  - View Animation（视图动画）
    - Tween animation（补间动画）
      - 这种实现方式可以使视图组件移动、放大、缩小以及产生透明度的变化。
> 是指通过指定View的初末状态和变化时间、方式，对View的内容完成一系列的图形变换来实现动画效果。

			- Alpha（渐变透明度动画效果）
			- Scale（渐变尺寸伸缩动画效果）
		- Frame animation（帧动画）
			- 传统的动画方法，通过顺序的播放排列好的图片来实现，类似电影。
			- Translate（画面转换位置移动动画效果）
			- Rotate（画面转移旋转动画效果）
	- Drawable Animation

5. XML布局文件下的组件ID是什么类型？
   XML布局文件下的组件ID为`int`类型



##内存相关

***嵌入式操作系统内存管理有哪几种？各有什么特征？***

页式，段式，段页，用到了MMU,虚拟空间等技术


***下列哪些语句关于内存回收的说明是正确的？***

内存回收程序负责释放无用内存





