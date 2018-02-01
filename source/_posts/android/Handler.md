# Handler
[TOC]

##  前言
Android的消息传递机制是另外一种形式的`事件处理`，这种机制主要是为了解决Android应用中多线程的问题，**在Android中不允许Activity新启动的线程访问该Activity里的UI组件，这样会导致新启动的线程无法改变UI组件的属性值**。但实际开发中，很多地方需要在工作线程中改变UI组件的属性值，比如下载网络图片、动画等等。本篇博客主要介绍Handler是如何发送与处理线程上传递来的消息，并讲解Message的几种传递数据的方式，最后均会以小Demo来演示。



## Handler
它直接继承自Object，**Handler允许发送和处理Message或者Runnable对象，并且会关联到主线程的MessageQueue中**。**每个Handler具有一个单独的线程，并且关联到一个消息队列的线程**，就是说一个Handler有一个固有的消息队列。当实例化一个Handler的时候，它就承载在一个线程和消息队列的线程，这个Handler可以把Message或Runnable压入到消息队列，并且从消息队列中取出Message或Runnable，进而操作它们。

### Handler主要有两个作用：
- 在工作线程中发送消息
- 在UI线程中获取、处理消息

上面介绍到Handler可以把一个Message对象或者Runnable对象压入到消息队列中，进而在UI线程中获取Message或者执行Runnable对象，所以Handler把压入消息队列有两大体系，Post和sendMessage：
- Post：允许把一个Runnable对象入队到消息队列中。它的方法有：`post(Runnable)`、`postAtTime(Runnable,long)`、`postDelayed(Runnable,long)`。
- sendMessage：允许把一个包含消息数据的Message对象压入到消息队列中。它的方法有：`sendEmptyMessage(int)`、`sendMessage(Message)`、`sendMessageAtTime(Message,long)`、`sendMessageDelayed(Message,long)`。

从上面的各种方法可以看出，不管是post还是sendMessage都具有多种方法，它们可以设定Runnable对象和Message对象被入队到消息队列中，是立即执行还是延迟执行。

### Handler的优缺点
- 优点：结构清晰, 功能定义明确. 对于多个后台任务时, 简单, 清晰
- 缺点：在单个后台异步处理时, 显得代码过多, 结构过于复杂(相对性)



## Post
对于Handler的Post方式来说，它会传递一个Runnable对象到消息队列中，在这个Runnable对象中，重写`run()`方法。一般在`run()`方法中写入需要在UI线程上的操作。

在Handler中，关于Post方式的方法有：
- `boolean post(Runnable r)`：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，立即执行。
- `boolean postAtTime(Runnable r,long uptimeMillis)`：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，在特定的时间执行。
- `boolean postDelayed(Runnable r,long delayMillis)`：把一个Runnable入队到消息队列中，UI线程从消息队列中取出这个对象后，延迟delayMills秒执行
- `void removeCallbacks(Runnable r)`：从消息队列中移除一个Runnable对象。

下面通过一个Demo，讲解如何通过Handler的post方式在新启动的线程中修改UI组件的属性：

```java
package com.bgxt.datatimepickerdemo;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class HandlerPostActivity1 extends Activity {
    private Button btnMes1,btnMes2;
    private TextView tvMessage;
    // 声明一个Handler对象
    private static Handler handler=new Handler();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.message_activity);        
        
        btnMes1=(Button)findViewById(R.id.btnMes1);
        btnMes2=(Button)findViewById(R.id.btnMes2);
        tvMessage=(TextView)findViewById(R.id.tvMessage);
        btnMes1.setOnClickListener(new View.OnClickListener() {
            
            @Override
            public void onClick(View v) {
                // 新启动一个子线程
                new Thread(new Runnable() {                    
                    @Override
                    public void run() {
                        // tvMessage.setText("...");
                        // 以上操作会报错，无法再子线程中访问UI组件，UI组件的属性必须在UI线程中访问
                        // 使用post方式修改UI组件tvMessage的Text属性
                        handler.post(new Runnable() {                    
                            @Override
                            public void run() {
                                tvMessage.setText("使用Handler.post在工作线程中发送一段执行到消息队列中，在主线程中执行。");                        
                            }
                        });                                
                    }
                }).start();
            }
        });
        
        btnMes2.setOnClickListener(new View.OnClickListener() {
            
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {                    
                    @Override
                    public void run() {
                        // 使用postDelayed方式修改UI组件tvMessage的Text属性值
                        // 并且延迟3S执行
                        handler.postDelayed(new Runnable() {
                            
                            @Override
                            public void run() {
                                tvMessage.setText("使用Handler.postDelayed在工作线程中发送一段执行到消息队列中，在主线程中延迟3S执行。");    
                                
                            }
                        }, 3000);                        
                    }
                }).start();
                
            }
        });
    }
}
```
效果演示：
![Alt text](./18181647-20bb326b9ef448b19355012b2067e644.png)

>注意：对于Post方式而言，它其中Runnable对象的`run()`方法的代码，均执行在UI线程上，所以对于这段代码而言，不能执行在UI线程上的操作，一样无法使用post方式执行，比如说访问网络，下面提供一个例子，使用post方式从互联网上获取一张图片，并且显示在ImageView中。

```java
 package com.bgxt.datatimepickerdemo;

import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;

import android.app.Activity;
import android.app.ProgressDialog;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;

public class HandlerPostActivity2 extends Activity {
    private Button btnDown;
    private ImageView ivImage;
    private static String image_path = "http://ww4.sinaimg.cn/bmiddle/786013a5jw1e7akotp4bcj20c80i3aao.jpg";
    private ProgressDialog dialog;
    // 一个静态的Handler，Handler建议声明为静态的
    private static  Handler handler=new Handler();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.asynctask_activity);
        
        btnDown = (Button) findViewById(R.id.btnDown);
        ivImage = (ImageView) findViewById(R.id.ivSinaImage);

        dialog = new ProgressDialog(this);
        dialog.setTitle("提示");
        dialog.setMessage("正在下载，请稍后...");
        dialog.setCancelable(false);
        
        btnDown.setOnClickListener(new View.OnClickListener() {            
            @Override
            public void onClick(View v) {
                // 开启一个子线程，用于下载图片
                new Thread(new MyThread()).start();
                // 显示对话框
                dialog.show();
            }
        });
    }
    
    public class MyThread implements Runnable {

        @Override
        public void run() {
            // 下载一个图片
            HttpClient httpClient = new DefaultHttpClient();
            HttpGet httpGet = new HttpGet(image_path);
            HttpResponse httpResponse = null;
            try {
                httpResponse = httpClient.execute(httpGet);
                if (httpResponse.getStatusLine().getStatusCode() == 200) {
                    byte[] data = EntityUtils.toByteArray(httpResponse
                            .getEntity());
                    // 得到一个Bitmap对象，并且为了使其在post内部可以访问，必须声明为final
                    final Bitmap bmp=BitmapFactory.decodeByteArray(data, 0, data.length);
                    handler.post(new Runnable() {                        
                        @Override
                        public void run() {
                            // 在Post中操作UI组件ImageView
                            ivImage.setImageBitmap(bmp);
                        }
                    });
                    // 隐藏对话框
                    dialog.dismiss();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
}
```
效果演示：
![Alt text](./18181733-3101871caea1429094c9740fdf5dd63e.png) ![Alt text](./18181759-82e2ddb4ade84d9eafb26507bef8fc27.png) ![Alt text](./18181819-e75f3b605d5c43159324a7d521346dd7.png)



## Message
>Handler如果使用sendMessage的方式把消息入队到消息队列中，需要传递一个Message对象，而在Handler中，需要重写handleMessage()方法，用于获取工作线程传递过来的消息，此方法运行在UI线程上。下面先介绍一下Message。

**Message是一个final类，所以不可被继承**。Message封装了线程中传递的消息，如果对于一般的数据，Message提供了`getData()`和`setData()`方法来获取与设置数据，其中操作的数据是一个Bundle对象，这个Bundle对象提供一系列的`getXxx()`和`setXxx()`方法用于传递基本数据类型的键值对，对于基本数据类型，使用起来很简单，这里不再详细讲解。而对于复杂的数据类型，如一个对象的传递就要相对复杂一些。

**在Bundle中提供了两个方法，专门用来传递对象**，但是这两个方法也有相应的限制，需要实现特定的接口，当然，一些Android自带的类，其实已经实现了这两个接口中的某一个，可以直接使用。方法如下：
- `putParcelable(String key,Parcelable value)`：需要传递的对象类实现Parcelable接口。
- `pubSerializable(String key,Serializable value)`：需要传递的对象类实现Serializable接口。

还有另外一种方式在Message中传递对象，那就是使用Message自带的obj属性传值，它是一个Object类型，所以可以传递任意类型的对象，Message自带的有如下几个属性：
- `int arg1`：参数一，用于传递不复杂的数据，复杂数据使用setData()传递。
- `int arg2`：参数二，用于传递不复杂的数据，复杂数据使用setData()传递。
- `Object obj`：传递一个任意的对象。
- `int what`：定义的消息码，一般用于设定消息的标志。

对于Message对象，一般并不推荐直接使用它的构造方法得到，而是**建议通过使用`Message.obtain()`这个静态的方法或者`Handler.obtainMessage()`获取**。`Message.obtain()`会从消息池中获取一个Message对象，如果消息池中是空的，才会使用构造方法实例化一个新Message，这样有利于消息资源的利用。并不需要担心消息池中的消息过多，它是有上限的，上限为10个。`Handler.obtainMessage()`具有多个重载方法，如果查看源码，会发现其实`Handler.obtainMessage()`在内部也是调用的`Message.obtain()`。

既然**Message是在线程间传递消息**，那么先以一个Demo讲解一下Message的使用，还是常规的从互联网上下载一张图片的Demo，下载后使用ImageView控件展示：

```java
package com.bgxt.datatimepickerdemo;

import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;

import android.app.Activity;
import android.app.ProgressDialog;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;

public class HandlerMessageActivity1 extends Activity {
    private Button btnDown;
    private ImageView ivImage;
    private static String image_path = "http://ww4.sinaimg.cn/bmiddle/786013a5jw1e7akotp4bcj20c80i3aao.jpg";
    private ProgressDialog dialog;
    private static int IS_FINISH = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.asynctask_activity);

        btnDown = (Button) findViewById(R.id.btnDown);
        ivImage = (ImageView) findViewById(R.id.ivSinaImage);

        dialog = new ProgressDialog(this);
        dialog.setTitle("提示信息");
        dialog.setMessage("正在下载，请稍后...");
        dialog.setCancelable(false);

        btnDown.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                    new Thread(new MyThread()).start();
                    dialog.show();
            }
        });
    }

    private  Handler handler = new Handler() {
        // 在Handler中获取消息，重写handleMessage()方法
        @Override
        public void handleMessage(Message msg) {            
            // 判断消息码是否为1
            if(msg.what==IS_FINISH){
                byte[] data=(byte[])msg.obj;
                Bitmap bmp=BitmapFactory.decodeByteArray(data, 0, data.length);
                ivImage.setImageBitmap(bmp);
                dialog.dismiss();
            }
        }
    };

    public class MyThread implements Runnable {

        @Override
        public void run() {
            HttpClient httpClient = new DefaultHttpClient();
            HttpGet httpGet = new HttpGet(image_path);
            HttpResponse httpResponse = null;
            try {
                httpResponse = httpClient.execute(httpGet);
                if (httpResponse.getStatusLine().getStatusCode() == 200) {
                    byte[] data = EntityUtils.toByteArray(httpResponse
                            .getEntity());
                    // 获取一个Message对象，设置what为1
                    Message msg = Message.obtain();
                    msg.obj = data;
                    msg.what = IS_FINISH;
                    // 发送这个消息到消息队列中
                    handler.sendMessage(msg);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
效果演示：
![Alt text](./18181733-3101871caea1429094c9740fdf5dd63e.png) ![Alt text](./18181759-82e2ddb4ade84d9eafb26507bef8fc27.png) ![Alt text](./18181819-e75f3b605d5c43159324a7d521346dd7.png)


**`Message.obtain()`方法具有多个重载方法**，大致可以分为为两类：
- 第一类：是无需传递Handler对象，对于这类的方法，当填充好消息后，需要调用Handler.sendMessage()方法来发送消息到消息队列中。
- 第二类：需要传递一个Handler对象，这类方法可以直接使用`Message.sendToTarget()`方法发送消息到消息队列中，这是因为在Message对象中有一个私有的Handler类型的属性Target，当时obtain方法传递进一个Handler对象的时候，会给Target属性赋值，当调用sendToTarget()方法的时候，实际在它内部还是调用的`Target.sendMessage()`方法。

在Handler中，也定义了一些发送空消息的方法，如：`sendEmptyMessage(int what)`、`sendEmptyMessageDelayed(int what,long delayMillis)`，看似这些方法没有使用Message就可以发送一个消息，但是如果查看源码就会发现，其实内部也是从`Message.obtain()`方法中获取一个Message对象，然后给属性赋值，最后使用`sendMessage()`发送消息到消息队列中。

**Handler中，与Message发送消息相关的方法有**：
- `Message obtainMessage()`：获取一个Message对象。
- `boolean sendMessage()`：发送一个Message对象到消息队列中，并在UI线程取到消息后，立即执行。
- `boolean sendMessageDelayed()`：发送一个Message对象到消息队列中，在UI线程取到消息后，延迟执行。
- `boolean sendEmptyMessage(int what)`：发送一个空的Message对象到队列中，并在UI线程取到消息后，立即执行。
- `boolean sendEmptyMessageDelayed(int what,long delayMillis)`：发送一个空Message对象到消息队列中，在UI线程取到消息后，延迟执行。
- `void removeMessage()`：从消息队列中移除一个未响应的消息。

下面通过一个小Demo演示一下各种发送Message的方式：

```java
package com.bgxt.datatimepickerdemo;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class HandlerMessageActivity2 extends Activity {
    private Button btn1, btn2, btn3, btn4,btn5;
    private static TextView tvMes;
    private static Handler handler = new Handler() {
        @Override
        public void handleMessage(android.os.Message msg) {
            if (msg.what == 3||msg.what==5) {
                tvMes.setText("what=" + msg.what + "，这是一个空消息");
            } else {
                tvMes.setText("what=" + msg.what + "," + msg.obj.toString());
            }

        };
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // TODO Auto-generated method stub
        super.onCreate(savedInstanceState);
        setContentView(R.layout.message_activity2);
        tvMes = (TextView) findViewById(R.id.tvMes);
        btn1 = (Button) findViewById(R.id.btnMessage1);
        btn2 = (Button) findViewById(R.id.btnMessage2);
        btn3 = (Button) findViewById(R.id.btnMessage3);
        btn4 = (Button) findViewById(R.id.btnMessage4);
        btn5 = (Button) findViewById(R.id.btnMessage5);

        btn1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 使用Message.Obtain+Hander.sendMessage()发送消息
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message msg = Message.obtain();
                        msg.what = 1;
                        msg.obj = "使用Message.Obtain+Hander.sendMessage()发送消息";
                        handler.sendMessage(msg);
                    }
                }).start();
            }
        });

        btn2.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                // 使用Message.sendToTarget发送消息
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message msg = Message.obtain(handler);
                        msg.what = 2;
                        msg.obj = "使用Message.sendToTarget发送消息";
                        msg.sendToTarget();
                    }
                }).start();
            }
        });

        btn3.setOnClickListener(new View.OnClickListener() {
            // 发送一个延迟消息
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        handler.sendEmptyMessage(3);
                    }
                }).start();
            }
        });

        btn4.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message msg = Message.obtain();
                        msg.what =4;
                        msg.obj = "使用Message.Obtain+Hander.sendMessage()发送延迟消息";
                        handler.sendMessageDelayed(msg, 3000);
                    }
                }).start();
            }
        });
        
        btn5.setOnClickListener(new View.OnClickListener() {
            // 发送一个延迟的空消息
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        handler.sendEmptyMessageDelayed(5, 3000);
                    }
                }).start();
            }
        });
    }
}
```
效果演示：
![Alt text](./18213015-a9a6620367c24f2f8411f7d60276b7b4.png) ![Alt text](./18213040-831ecc2f470f4841834847f7996311ef.png) ![Alt text](./18213101-03c2e04a92fc48188a08ed9db3ed0304.png) ![Alt text](./18213122-b11621d064e44c0ebf4dbfd1741a0560.png) ![Alt text](./18213146-911350280fd84a5780c556903aab8045.png)





