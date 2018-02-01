# Android 消息处理机制

[TOC]

**Android消息处理机制有三个核心类：Looper、Handler、Message**。

## Looper

Looper用来使一个普通线程变成Looper线程。所谓**Looper线程就是循环工作的线程**。
>在程序开发中（尤其是GUI开发中），我们经常会需要一个线程不断循环，一旦有新任务则执行，执行完继续等待下一个任务，这就是Looper线程。

```java
public class LooperThread extends Thread {
    @Override
    public void run() {
        // 将当前线程初始化为Looper线程
        Looper.prepare();
        
        // ...其他处理，如实例化handler
        
        // 开始循环处理消息队列
        Looper.loop();
    }
}
```
1. `Looper.perpare()`
   ![Alt text](./2011091315284989.png)

通过上图可以看到，现在线程中有一个Looper对象，它的内部维护了一个消息队列MQ。>注意：**一个Thread只能有一个Looper对象**
```java
public class Looper {
    // 每个线程中的Looper对象其实是一个ThreadLocal，即线程本地存储(TLS)对象
    private static final ThreadLocal sThreadLocal = new ThreadLocal();
    // Looper内的消息队列
    final MessageQueue mQueue;
    // 当前线程
    Thread mThread;
    // 。。。其他属性

    // 每个Looper对象中有它的消息队列，和它所属的线程
    private Looper() {
        mQueue = new MessageQueue();
        mRun = true;
        mThread = Thread.currentThread();
    }

    // 我们调用该方法会在调用线程的TLS中创建Looper对象
    public static final void prepare() {
        if (sThreadLocal.get() != null) {
            // 试图在有Looper的线程中再次创建Looper将抛出异常
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper());
    }
    // 其他方法
}
```
通过源码，`prepare()`背后的工作方式其核心就是将looper对象定义为[ThreadLocal](https://app.yinxiang.com/shard/s9/nl/718260/b3f37135-3052-40bb-86c3-c75000b4f12b)。

2. `Looper.loop()`
   ![Alt text](./2011091315590586.png)
   调用loop方法后，Looper线程就开始真正工作了，它不断从自己的MQ中取出队头的消息(也叫任务)执行。其源码分析如下：
```java
public static final void loop() {
	Looper me = myLooper();  //得到当前线程Looper
	MessageQueue queue = me.mQueue;  //得到当前looper的MQ

	// 这两行没看懂= = 不过不影响理解
	Binder.clearCallingIdentity();
	final long ident = Binder.clearCallingIdentity();
	// 开始循环
	while (true) {
		Message msg = queue.next(); // 取出message
		if (msg != null) {
			if (msg.target == null) {
				// message没有target为结束信号，退出循环
				return;
			}
		// 日志。。。
		if (me.mLogging!= null) me.mLogging.println(
			">>>>> Dispatching to " + msg.target + " "
			+ msg.callback + ": " + msg.what);
			// 非常重要！将真正的处理工作交给message的target，即后面要讲的handler
			msg.target.dispatchMessage(msg);
            // 还是日志。。。
            if (me.mLogging!= null) me.mLogging.println(
				"<<<<< Finished to    " + msg.target + " "
				+ msg.callback);
                
				// 下面没看懂，同样不影响理解
				final long newIdent = Binder.clearCallingIdentity();
				if (ident != newIdent) {
					Log.wtf("Looper", "Thread identity changed from 0x"
						+ Long.toHexString(ident) + " to 0x"
	                    + Long.toHexString(newIdent) + " while dispatching to "
	                    + msg.target.getClass().getName() + " "
	                    + msg.callback + " what=" + msg.what);
	            }
            // 回收message资源
            msg.recycle();
            }
        }
    }
```

除了`prepare()`和`loop()`方法，Looper类还提供了一些有用的方法，比如：
```java
public static final Looper myLooper() {
		// 获取当前looper对象
        // 在任意线程调用Looper.myLooper()返回的都是那个线程的looper
        return (Looper)sThreadLocal.get();
}

public Thread getThread() {
		// 获取looper对象所属线程
        return mThread;
}

public void quit() {
        // 创建一个空的message，它的target为NULL，表示结束循环消息
        Message msg = Message.obtain();
        // 发出消息
        mQueue.enqueueMessage(msg, 0);
}
```

总结：
1. **每个线程有且最多只有一个Looper对象，它是一个[ThreadLocal](https://app.yinxiang.com/shard/s9/nl/718260/b3f37135-3052-40bb-86c3-c75000b4f12b)**
2. Looper内部有一个消息队列，`loop()`方法调用后线程开始不断从队列中取出消息执行
3. Looper使一个线程百年城Looper线程

## Handler
Handler扮演了往MessageQueue上添加和处理消息的角色（只处理由自己发出的消息），即通知MessageQueue它要执行一个任务（sendMessage），并在`loop`到自己的时候执行该任务（handleMessage），整个过程是异步的。Handler创建时会关联一个Looper，默认的构造方法将关联当前线程的Looper。

```
public class handler {
    final MessageQueue mQueue;  // 关联的MQ
    final Looper mLooper;  // 关联的looper
    final Callback mCallback; 
    // 其他属性
    public Handler() {
        // 没看懂，直接略过，，，
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
        // 默认将关联当前线程的looper
        mLooper = Looper.myLooper();
        // looper不能为空，即该默认的构造方法只能在looper线程中使用
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        // 重要！！！直接把关联looper的MQ作为自己的MQ，因此它的消息将发送到关联looper的MQ上
        mQueue = mLooper.mQueue;
        mCallback = null;
    }
    
    // 其他方法
}
```

下面为之前的LooperThread类加入Handler：
```java
public class LooperThread extends Thread {
    private Handler handler1;
    private Handler handler2;
    @Override
    public void run() {
        // 将当前线程初始化为Looper线程
        Looper.prepare();
        
        // 实例化两个handler
        handler1 = new Handler();
        handler2 = new Handler();
        
        // 开始循环处理消息队列
        Looper.loop();
    }
}
```
加入handler后的效果如下图：
![Alt text](./2011091322483894.png)
可以看到，**一个线程可以有多个Handler，但只能有一个Looper**。

### Handler发送消息
声明了Handler之后，可以使用以下这些方法MessageQueue上发送消息：
-  `post(Runnable)`
-  `postAtTime(Runnable, long)`
-  `postDelayed(Runnable, long)`
-  `sendEmptyMessage(int)`
-  `sendMessage(Message)`
-  `sendMessageAtTime(Message, long)`
-  `sendMessageDelayed(Message, long)`

从以上方法可以看出，Handler能发出两种消息：
- Message对象，Handler发出的Message有如下特点：
1. message.target为该handler对象，这确保了looper执行到该message时能找到处理它的handler，即`loop()`方法中的关键代码：`msg.target.dispatchMessage(msg);`
2. post发出的message，其callback为Runnable对象
- Runnable对象，但其实post发出的Runnable对象最后都被封装成message对象，见源码：
```java
// 此方法用于向关联的MQ上发送Runnable对象，它的run方法将在handler关联的looper线程中执行
public final boolean post(Runnable r){
	// 注意getPostMessage(r)将runnable封装成message
	return  sendMessageDelayed(getPostMessage(r), 0);
}

private final Message getPostMessage(Runnable r) {
        Message m = Message.obtain();  //得到空的message
        m.callback = r;  //将runnable设为message的callback，
        return m;
}

public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
	boolean sent = false;
	MessageQueue queue = mQueue;
	if (queue != null) {
            msg.target = this;  // message的target必须设为该handler！
            sent = queue.enqueueMessage(msg, uptimeMillis);
	} else {
		RuntimeException e = new RuntimeException(
		this + " sendMessageAtTime() called with no mQueue");
		Log.w("Looper", e.getMessage(), e);
	}

	return sent;
}
```
### Handler处理消息
Handler处理消息的核心方法：
- dispatchMessage(Message msg)
- handleMessage(Message msg) `钩子方法`
```java
// 处理消息，该方法由looper调用
public void dispatchMessage(Message msg) {
	if (msg.callback != null) {
	// 如果message设置了callback，即runnable消息，处理callback！
	handleCallback(msg);
	} else {
		// 如果handler本身设置了callback，则执行callback
		if (mCallback != null) {
			/* 这种方法允许让activity等来实现Handler.Callback接口，避免了自己编写handler重写handleMessage方法。见http://alex-yang-xiansoftware-com.iteye.com/blog/850865 */
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
	// 如果message没有callback，则调用handler的钩子方法handleMessage
	handleMessage(msg);
	}
}
    
// 处理runnable消息
private final void handleCallback(Message message) {
        message.callback.run();  //直接调用run方法！
}
// 由子类实现的钩子方法
public void handleMessage(Message msg) {
}
```
**Handler的内部工作机制对开发者是透明的**

### Handler的用处
1. Handler可以在**任意线程发送消息**，这些消息会被添加到关联的MessageQueue上。
   ![Alt text](./2011091409532564.png)

2. Handler是在它**关联的looper线程中处理消息**的。
   ![Alt text](./2011091409541179.png)

这就解决了android最经典的不能在其他非主线程中更新UI的问题。**android的主线程也是一个looper线程**，我们在其中创建的handler默认将关联主线程MessageQueue。因此，利用handler的一个solution就是在activity中创建handler并将其引用传递给worker thread，worker thread执行完任务后使用handler发送消息通知activity更新UI。(过程如图)
![Alt text](./2011091323582123.png)

参考代码：
```java
public class TestDriverActivity extends Activity {
    private TextView textview;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        textview = (TextView) findViewById(R.id.textview);
        // 创建并启动工作线程
        Thread workerThread = new Thread(new SampleTask(new MyHandler()));
        workerThread.start();
    }
    
    public void appendText(String msg) {
        textview.setText(textview.getText() + "\n" + msg);
    }
    
    class MyHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            String result = msg.getData().getString("message");
            // 更新UI
            appendText(result);
        }
    }
}

public class SampleTask implements Runnable {
    private static final String TAG = SampleTask.class.getSimpleName();
    Handler handler;
    
    public SampleTask(Handler handler) {
        super();
        this.handler = handler;
    }

    @Override
    public void run() {
        try {  // 模拟执行某项任务，下载等
            Thread.sleep(5000);
            // 任务完成后通知activity更新UI
            Message msg = prepareMessage("task completed!");
            // message将被添加到主线程的MQ中
            handler.sendMessage(msg);
        } catch (InterruptedException e) {
            Log.d(TAG, "interrupted!");
        }

    }

    private Message prepareMessage(String str) {
        Message result = handler.obtainMessage();
        Bundle data = new Bundle();
        data.putString("message", str);
        result.setData(data);
        return result;
    }

}
```

### Message

在整个消息处理机制中，message又叫task，**封装了任务携带的信息和处理该任务的handler**。

使用Message中需要注意的几点：
1. 尽管Message有public的默认构造方法，但是你应该通过`Message.obtain()`来从消息池中获得空消息对象，**以节省资源**。
2. 如果你的message只需要携带简单的`int`信息，请优先使用`Message.arg1`和`Message.arg2`来传递信息，**这比用Bundle更省内存**。
3. 擅用`message.what`来标识信息，以便用不同方式处理message。




