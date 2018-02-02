# Android Interview Questions

[TOC]

## 四大组件
### Activity



#### 如何退出activity？ 如何安全退出调用多个activity的application？

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



#### startActivityForResult()，onActivityResult() 的作用？

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



#### 如果后台的Activity由于某原因被系统回收了，如何在被系统回收之前保存当前状态？

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



#### 横竖屏切换时，Activity的生命周期？

不设置Activity的`android:configChanges`时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次。

设置Activity的`android:configChanges="orientation"`时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次。

设置Activity的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新调用各个生命周期，只会执行`onConfigurationChanged`方法。



#### 如何将一个Activity设置成窗口的样式？

在`AndroidManifest.xml中`定义Activity的地方一句话
```xml
// 均可变成半透明
android:theme="@android:style/Theme.Dialog"
android:theme="@android:style/Theme.Translucent"
```


***当启动一个Activity并且新的Activity执行完后需要返回到启动它的Activity来执行的回调函数是？***
`startActivityResult()`




#### Activity一般会重载那几个方法用来维护其生命周期？

`onCreate()` `onStart()` `onDestory()` `onRestart()` `onResume()` `onPause()` `onStop()`



#### 对一些资源以及状态的操作保存，最好是保存在生命周期的哪个函数中进行？
`onPause()`

#### LaunchMode的使用场景
**standard 模式***
这是默认模式，每次激活Activity时都会创建Activity实例，并放入任务栈中。使用场景：大多数Activity。
** singleTop 模式**
如果在任务的栈顶正好存在该Activity的实例，就重用该实例( 会调用实例的 onNewIntent() )，否则就会创建新的实例并放入栈顶，即使栈中已经存在该Activity的实例，只要不在栈顶，都会创建新的实例。使用场景如新闻类或者阅读类App的内容页面。
**singleTask 模式**
如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的 onNewIntent() )。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移出栈。如果栈中不存在该实例，将会创建新的实例放入栈中。使用场景如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。
**singleInstance 模式**
在一个新栈中创建该Activity的实例，并让多个应用共享该栈中的该Activity实例。一旦该模式的Activity实例已经存在于某个栈中，任何应用再激活该Activity时都会重用该栈中的实例( 会调用实例的 onNewIntent() )。其效果相当于多个应用共享一个应用，不管谁激活该 Activity 都会进入同一个应用中。使用场景如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。


#### 有关Activity生命周期描述正确的是？

未设置Activity的`android:configChanges`属性，切换屏幕横纵方向时会重新调用`onCreate()`方法。

当再次启动某个`launchMode`设置为`singletask`的Activity，它的`onNewIntent()`方法会被触发。



### Service



#### Service是如何向Activity传递数据的？



#### 用什么方式使 Service 和Activity 之间传递长久保存的对象？



#### Service的两种启动方式之间的区别和使用场景？
startService()：onCreate()	onStartCommend()	onDestory()
bindService()：onCreate()	onBind()	onUnBind()	onDestory()




### Intent
#### 显式intent和隐式intent的区别
- 定义
  - 显式intent：对于明确指出了目标主键名称的intent。
  - 隐式intent：对于没有明确指出目标组件名称的intent。
- 区别：
  - 显式更多用于在应用程序内部传递消息。
  - 隐式intent用于在不同应用程序之间传递消息。

#### IntentService与Service有什么不同，区别在哪？



#### IntentService的使用场景与特点？



#### Service的启动方法有几种，分别有什么区别？



### BroadcastReceiver

#### 什么情景下用到广播？
Android中的广播事件有两种：
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
#### Android引入广播机制的用意？

从MVC的角度考虑(应用程序内)



#### 注册广播有几种方式，这些方式有何区别？
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



#### ContentProvider是如何实现数据共享的？

- 创建一个属于你自己的ContentProvider或者将你的数据添加到一个已经存在的Contentprovider中，前提是有相同数据类型并且有写入ContentProvider的权限。


#### 在多个应用中读取共享存储数据时，需要用到的query方法，是哪个对象的方法？

- `ContentResolver`
  - 运行在提供数据端，`ContentResolver`一个运行在调用端。
  - **在多个应用中读取共享存储数据，只能用`ContentResolver`读取数据**



## 数据库相关



### 如何将打开`resaw`目录中的数据库文件？

- 在Android中不能直接打开`resaw`目录中的数据库文件，而需要在程序第一次启动时将该文件复制到手机内存或SD卡的某个目录中，然后再打开该数据库文件。
- 复制的基本方法是使用`getResources().openRawResource`方法获得`resaw`目录中资源的`InputStream`对象，然后将该`InputStream`对象中的数据写入其他的目录中相应文件中。
- 在AndroidSDK中可以使用`SQLiteDatabase.openOrCreateDatabase`方法来打开任意目录中的SQLite数据库文件。


### 什么时候用到数据库？

- 本地数据库SQLite主要作用是存储和保留网络数据，保证无网络数据或网络异常下能够使用，同时减少网络流量的损耗。
- 如果数据是已知的、静态的，可以在本地SQLite中存储、读取。这样不会因网络问题而降低效率和成功率。
- 如果数据未知、有实时的变化或者有与其他用户交互、共享的数据必然需要后台服务器数据。



### Android数据存储的方式有哪几类？

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




### 如何将SQLite数据库（`dictionary.db`文件）与apk文件一起发布？

- 可以将dictionary.db文件复制到Eclipse Android工程中的`resaw`目录中。所有在resaw目录中的文件不会被压缩，这样可以直接提取该目录中的文件。




### 请继承`SQLiteOpenHelper`实现：

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



### 在Android中使用`SQLiteOpenHelper`这个辅助类时，可以生成一个数据库，并可以对数据库版本进行管理的方法可以是？

- `getWriteableDatabase()`
- `getReadableDatabase()`




## 进程相关



### AsyncTask 的实现原理和使用的优缺点？



### Handler 异步实现的原理和使用的优缺点？



### Handler 的机制原理？

- **Andriod提供了Handler和Looper来满足线程间的通信**。
- Handler：构造Handler对象来与Looper沟通，以便push新消息到MessageQueue里;或者接收Looper从MessageQueue取出所送来的消息。
  - Looper：用来管理特定线程内对象之间的消息交换(MessageExchange)。一个线程可以产生一个Looper对象，由它来管理此线程里的MessageQueue(消息队列)。
  - MessageQueue(消息队列)：用来存放线程放入的消息。


### 在单线程模型中Message、Handler、MessageQueue、Looper之间的关系？

- 简单地说：Handler获取当前线程中的Looper对象，而Looper用来存放MessageQueue中取出的Message，再由Handler进行Message的分发和处理，按照先进先出执行。
- MessageQueue（消息队列）：用来存放通过Handler发布的消息，通常附属于某一个创建它的线程，可以通过`Looper.myQueue()`得到当前线程的消息队列。
- Handler：可以发布和处理一个Message或者操作一个Runnable，通过Handler发布消息，Message将只会发送到与它关联的MessageQueue，然而也只能处理该MessageQueue中的消息
- Looper：是Handler和消息队列之间的通讯桥梁，程序组件首先通过Handler把消息传递给Looper，然后Looper把Message放入MessageQueue。
- Message：消息的类型，在Handler类中的`handleMessage`方法中获取单个消息进行处理、
- 在单线程模型下，为了解决线程通信问题，Android设计了MessageQueue，线程间可以通过该MessageQueue并结合Handler和Looper组件进行信息交换。

### 什么是线程池，作用是什么？

### Android系统对对象提供了资源池有哪些？

- `Message`
  - 提供了消息池，有静态方法Obtain从消息池中取对象
- `AsyncTask`
  - 是线程池改造的，池里 默认提供（核数+1）个线程进行并发操作，最大支持（核数  * 2 + 1）个线程，超过后会丢弃其他任务



### 为满足线程间通信，Android提供了？

- `Handler`和`Looper`

### Android跨进程通讯的方式？
1. 访问其他应用程序的Activity
```java
Intent callIntent = new Intent(Intent.ACTION_CALL, Uri.parse("tel:12345678"));
startActivity(callIntent);
```

2. Content Provider

>如：访问系统相册

3. Broadcast

>如：显示系统时间

4. AIDL


### 遇到下列哪种情况时需要把进程移到前台？

- 进程正在运行一个与用户交互的Activity ，它的onResume()方法被调用
- 进程有一正在运行的BroadcastReceiver，它的onReceive()方法正在执行
- 进程有一个Service，并且在Service的某个回调函数（onCreate()、onStart()、或onDestroy()）内有正在执行的代码
- 进程有一个Service，该Service对应的Activity正在与用户交互


### 我们都知道Hanlder是线程与Activity通信的桥梁，如果线程处理不当，你的机器就会变得越慢，那么线程销毁的方法是？



### 下面哪种进程最重要，最后被销毁？

- 服务进程
- 后台进程
- 可见进程
- `前台进程`
  重要性依次是：前台进程--->可见进程--->服务进 --->后台进程和空进程
  所以销毁的顺序是逆方向。


### AndroidDvm的进程和Linux的进程，应用程序的进程是否为同一个概念？

- DVM指`Dalvik`虚拟机，每一个Android应用程序都在它自己的进程中运行，都拥有一个独立的`Dalvik`虚拟机实例。而每一个DVM都是在Linux中的一个进程，所以说可以认为是同一个概念。


### 谈谈Android的IPC（进程间通信）机制

- IPC是内部进程通信的简称，是共享"命名管道"的资源。
- 是为了让Activity和Service之间可以随时的进行交互，故在Android中该机制，只适用于Activity和Service之间的通信，类似于远程方法调用，类似于C/S模式的访问。通过定义AIDL接口文件来定义IPC接口。Servier端实现IPC接口，Client端调用IPC接口本地代理。



### Android 数据持久化主要有几种？

1. 保存到Shared Preferences
2. 保存到手机内存
3. 保存到SDCard中
4. 保存到SQlite数据库中



## 系统相关



### 简述Android应用程序结构是哪些？*

- 应用运行时抛出了OutOfMemoryError
- 应用运行时抛出了RuntimeException
- 一般像空指针啊，可以看起`logcat`，然后对应到程序中来解决错误


### 如何避免应用闪退？


### ANR
#### 什么是ANR？
**ANR排错的三种类型**：
- KeyDispathTimeOut (5s)：主要是类型按键或触摸事件在特定时间内无响应

- BrodcastTimeOut (10s) ：BroadcastReceiver在特定时间内无法处理完成

- ServiceTimeOut (20s) ：小概率事件 Service在特定的时间内无法处理完成

  ​

#### 哪些操作会导致ANR？ 如何避免？
**在主线程执行以下操作**：
1. 高耗时操作，如图像变换
2. 磁盘读写，数据库读写操作
3. 大量的创建新对象

**避免**：
1. UI线程尽量只做跟UI相关的工作
2. 耗时操作（比如数据库操作，I/O，链接网络或者别的有可能阻塞UI线程的操作）把它们放在单独的线程处理
3. 尽量用Handler来处理UIThread和别的Thread之前的交互

**解决逻辑**：
1. 使用AsyncTask
   1. 在doInBackground()方法中执行耗时操作
   2. 在onPostExecuted()更新UI
2. 使用Handler实现一部任务
   1. 在子线程中处理耗时操作
   2. 处理完成之后，通过Handler.sendMessage()传递处理结果
   3. 在Handler的handlerMessage()方法中更新UI
   4. 或者使用Handler.post()方法将消息放到Looper中

**如何排查**：
1. 首先分析log
2. 从trace.txt文件查看调用stack，adb pull data/anr/traces.txt ./mytraces.txt
3. LeakCanary


#### 下列哪些情况下系统会弹出Froce Close（强制关闭）对话框？如何避免？能否捕获导致其的异常？
**什么时候会出现**：
1. Error
2. OOM（OutOfMemoryError），内存溢出
3. StackOverFlowError
4. RuntimeException，比如空指针异常

**避免**：
1. 注意内存的使用和管理
2. 使用Thread.UncaughtExceptionHandler接口

### Android本身的api并未声明会抛出异常，则其在运行时有无可能抛出Runtime异常，你遇到过吗?诺有的话会导致什么问题?如何解决?

- 比如nullpointerException
- 比如`textview.setText()`时，`textview`没有初始化。会导致程序无法正常运行出现ForceClose。打开控制台查看logcat信息找出异常信息并修改程序。


### Android程序运行时权限与文件系统权限的区别。运行时权限？

- Dalvik(android授权)
- 文件系统linux内核授权


### 系统上安装了多种浏览器，能否指定某浏览器访问指定页面？请说明原由。

通过直接发送Uri把参数带过去，或者通过manifest里的intentfilter里的data属性。



### AIDL的全称是什么？如何工作？能处理哪些类型的数据？

- Android Interface Define Language
- 当A进程要去调用B进程中的service时，并实现通信，通常是通过AIDL来操作的
- A工程：首先我们在net.blogjava.mobile.aidlservice包中创建一个RemoteService.aidl文件，在里面我们自定义一个接口，含有方法get。ADT插件会在gen目录下自动生成一个RemoteService.java文件，该类中含有一个名为RemoteService.stub的内部类，该内部类中含有aidl文件接口的get方法。说明一：aidl文件的位置不固定，可以任意。然后定义自己的MyService类，在MyService类中自定义一个内部类去继承RemoteService.stub这个内部类，实现get方法。在onBind方法中返回这个内部类的对象，系统会自动将这个对象封装成IBinder对象，传递给他的调用者。
  其次需要在`AndroidManifest.xml`文件中配置MyService类，代码如下：
  ​```xml
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
​```java
bindService(new Inten("net.blogjava.mobile.aidlservice.RemoteService"),serviceConnection,Context.BIND_AUTO_CREATE);
```
ServiceConnection的`onServiceConnected(ComponentNamename,IBinderservice)`方法中的service参数就是A工程中MyService类中继承了`RemoteService.stub`类的内部类的对象。



### Android 使用AIDL提供公开服务接口，使得不同进程间可以相互通信。

- AIDL 对应的接口名称必须与AIDL文件名相同不然无法自动编译
- AIDL 对应的接口方法不能加访问权限修复
- 建立AIDL服务要比建立普通的服务复杂一些，具体步骤如下：
1. 在Eclipse Android工程的Java包目录中建立一个扩展名为aidl的文件。该文件的语法类似于Java代码，但会稍有不同。
2. 如果aidl文件的内容是正确的，ADT会自动生成一个Java接口文件（*.java）。
3. 建立一个服务类（Service的子类）。
4. 实现由aidl文件生成的Java接口。
5. 在AndroidManifest.xml文件中配置AIDL服务，尤其要注意的是，`<action>`标签中`android:name`的属性值就是客户端要引用该服务的ID，也就是Intent类的参数值。



### 请简述JNI在Android中的调用过程？

- 安装和下载Cygwin，下载AndroidNDK
- 在ndk项目中JNI接口的设计
- 使用C/C++实现本地方法
- JNI生成动态链接库.so文件
- 将动态链接库复制到java工程，在java工程中调用，运行java工程即可


### SIM卡的EF文件有何作用？

- SIM卡的文件系统有自己规范，主要是为了和手机通讯，SIM本身可以有自己的操作系统
- EF就是作存储并和手机通讯用的


### 什么是嵌入式实时操作系统？Android操作系统属于实时操作系统吗？

- 嵌入式实时操作系统，是指当外界事件或数据产生时，能够接受并以足够快的速度予以处理，其处理的结果又能在规定的时间之内来控制生产过程或对处理系统作出快速响应，并控制所有实时任务协调一致运行的嵌入式操作系统。
- 实时系统，主要用于工业控制、军事设备、航空航天等领域对系统的响应时间有苛刻的要求。（又可分为软实时和硬实时两种）
- 而Android是基于Linux内核的，因此属于软实时。


### 一条最长的短信息约占多少byte？

- 中文70(包括标点)，英文160，160个字节。


### 如何防止apk包被反编译？

- 代码混淆：java代码混淆工具 `proguard`
- 二次打包


###  DDMS和TraceView的区别？

- DDMS是一个程序执行查看器，在里面可以看见线程和堆栈等信息，TraceView是程序性能分析器。
### 什么是Context？
- **它描述的是一个应用程序环境的消息，即上下文**。
- 该类是一个抽象（abstract class）类。Android提供了该抽象类的集体实现类。
- 通过它我们可以获取应用程序的资源和类，也包括一些应用级别操作。例如：启动一个Activity，发送广播，接收Intent信息等。
- 总结：**Context是一个抽象基类**，我们通过它访问当前包的资源(getResources, getAssets)和启动其他组件(Activity, Service, Broadcast)以及得到各种服务(getSystemService)，当然，通过Context提供了一个应用的运行环境，在Context定义了一套基本的功能接口，我们可以理解为一套规范，而Activity和Service是实现这套规范的子类，这么说也许并不准确，因为这套规范实际是被ContextImpl类统一实现的，Activity和Service只是继承并有选择性地重写了某些规范的实现。

Context相关类的继承关系：
![Alt text](./Image.png)
 通过上图可以看出，Activity类、Service类、Application类本质上都是Context的子类。



### Application、Activity、Service作为Context的区别?

- 相同点：它们都简介继承了Context
- 不同点：首先看它们的继承关系，通过对比可以发现，Activity比Service和Application多了一层继承关系ContextTemeWrapper，这是因为Activity有主题概念，而Service没有界面服务，Application更是一个抽象的东西，它也是通过Activity类呈现的。Context的真正实现都在ContextImpl中，也就是说Context的大部分方法调用都会转到ContextImpl中，而三者的创建均在ActivityThread中完成，**Activity启动的核心过程是在ActivityThread中完成的**，这里要说明的是，**Application和Service的创建也是在ActivityThread中完成的**。
### 一个应用程序中有多少Context？
总Context实例个数 = Service个数 + Activity个数 + 1(Application对应的Context实例)



### Intent传递数据时，下列的数据类型哪些可以被传递？

- Serializable
- Charsequence
- Parcelable
- Bundle



### 在 Android 中， 在屏幕密度为160时， 1pt 大概等于__sp？***

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




### 页面上现有`ProgressBar`控件，请用书写线程以10秒的的时间完成其进度显示工作。

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


### 在Android中使用Menu时可能需要重写的方法有？

- `onCreateOptionsMenu()`
- `onOptionsItemSelected()`




### 简要解释一下Activity、Intent、IntentFilter、Service、BroadcastReceiver？

- Activity呈现了一个用户可以操作的可视化用户界面
- Service不包含可见的用户界面，而是在后台无限地运行，可以连接到一个正在运行的服务中，连接后，可以通过服务中暴露出来的借口与其进行通信
- BroadcastReceiver是一个接收广播消息并作出回应的component，broadcastreceiver没有界面
- Intent：ContentPprovider在接收到ContentResolver的请求时被激活
- Activity、Sservice和Broadcastreceiver是被称为Intent的异步消息激活的。
- 对于Activity和Service来说，Intent指定了请求的操作名称和待操作数据的URI
- Intent对象可以显式的指定一个目标component。如果这样的话，android会找到这个component(基于manifest文件中的声明)并激活它。但如果一个目标不是显式指定的，android必须找到响应intent的最佳component。
- 它是通过将Intent对象和目标的intentfilter相比较来完成这一工作的。一个component的intentfilter告诉android该component能处理的intent。intentfilter也是在manifest文件中声明的。


### Binder的理解


## View
### View的绘制流程

![20150529090922419](d:\Users\mcgrady\Documents\Markdown\Blog\Interview\20150529090922419.jpg)

#### onMeasure 过程
![整个View树的源码measure流程图](d:\Users\mcgrady\Documents\Markdown\Blog\Interview\20150529163050000.jpg)

measure过程主要就是从顶层父View向子View递归用view.measure方法（measure中又回调onMeasure方法）的过程。

- MeasureSpec（View的内部类）测量规格为int型，值由高2位规格模式specMode和低30位具体尺寸specSize组成。其中specMode只有三种值：
  - MeasureSpec.EXACTLY	// 确定模式
    - MeasureSpec.AT_MOST// 最多模式
    - MeasureSpec.UNSPECIFIED// 未指定模式
- View的measure方法是final的，不允许重载，View子类只能重载onMeasure来完成自己的测量逻辑。
- 最顶层DecorView测量measureChild，measureChild和measureChildWithMargins方法，简化了父子View的尺寸计算。
- 只要是ViewGroup的子类就必须要求LayoutParams继承子MarginLayoutParams，否则无法使用layout_margin参数。
- View的布局大小由父View和子View共同决定。
- 使用View的getMeasureWidth()和getMeasureHeight()方法

#### onLayout 过程

#### onDraw 过程
#### ListView的优化
#### View的状态与重绘
#### View的事件分发机制
#### View的事件冲突处理

### Activity View Window 三者之间的关系
Activity（控制单元），Window（承载模型），View（显示视图）

Activity在onCreate()中调用attrch()方法，attrch()方法中创建了一个window(PhoneWindow)对象；当用户调用setContentView()方法，实际是window的setContentView()，这是window检查DecorView是否存在，不存在则创建DecorView对象，然后把用户自己的View添加到DecorView中;当前用户添加的View的事件监听是由window的WindowManagerService来接收消息，并且回调给Activity函数。

#### LayoutInflater
LayoutInflater只做一件事，就是把xml表述的layout转化成View。


### 自定义View

## Android系统的原理和机制

### Touch事件机制

### Android 动画
#### 属于Android 的动画类型有？
- Frame animation（逐帧动画）
  将多张图片组合起来进行播放，很多loading动画使用这种方式。

- Tween animation（补间动画）
  是对某个View进行一些列的动画操作，包括：
  - Alpha（渐变透明度动画效果）
  - Scale（渐变尺寸伸缩动画效果）
  - Translate（画面转换位置移动动画效果）
  - Rotate（画面转移旋转动画效果）

- Property Animation（属性动画）
  不仅仅是一种视觉动画，而是一种不断地对值进行操作的机制，并赋值到指定对象的指定属性上，可以是任意对象的任意属性。


