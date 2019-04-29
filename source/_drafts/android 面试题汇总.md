# Interview questions summary



## Android相关

### 四大组件
##### 1. 请描述四大组件的生命周期和简单用法？
```java
activity
onCreate onStart onResume onPause onStop onDestory
Intent intent = new Intent(context, YouActivity.class);
context.startActivity(intent);

service
静态 onCreate onStartCommand onDestory
动态 onCreate onBind onUnbind onDestory
<service android:name=".YouService"></service>
//or
Intent intent = new Intent(context, YouService.class);
context.startService(intent);
context.stopService(intent);

broadcast
动态注册跟随activity,静态注册跟随app生命周期，且不需手动注销
<reciver android:name=".YouRevice"></reciver>
//or
Intent intent = new Intent("android.intent.action.xxx");
conetxt.sendBroadcast(intent);
//or
context.registerBroadcast(YourBroadcastReciver, intentFilter);
context.unreigsterBroadcast(YourBroadcastReciver);

contentProvider
跟随进程的生命周期
<application>
  <provider android:name="com.path.YouProvider">
</application>
Uri uri = Uri.pase("content://xxx");
ContentResoler resoler = new ContentResoler();
resoler.insert(uri, contentValue);
```
#### Activity
##### 1. Activity生命周期，有哪些启动模式，以及应用场景？
##### 2. 写一个SingTop，有哪三个条件？
##### 3. Activity启动模式的应用场景？
- standard：创建一个新的Activity
- singleTop：栈顶不是该类型的Activity，创建一个行的Activity。否则，onNewIntent。
- singleTask：回退栈中没有该类型的Activity，创建Activity，否则，onNewIntent + ClearTop。
- singleInstance：回退栈中只有这个Activity，没有其他Activity。
##### 4. 两个Activity之间跳转时必然会执行的时哪几个方法？
##### 5. 请描述横竖屏切换时Activity的生命周期变化？
##### 6. 如何保存Activity的状态？


#### Service
##### 1.  service有几种启动方式？分别有什么区别？
##### 2. 描述service的生命周期？
##### 3. service是否在主线程中执行？
##### 4. service里可以弹toast吗？
##### 5. service里可以执行耗时操作吗？

#### Broadcast
##### 1. 注册广播的方式有几种？各有什么优缺点？

#### ContentProvider
##### 1. 如何实现数据共享？
##### 2. ContentProvider、ContentResolver、ConentObserver之间的关系

### View

#### 1. Activity、Window、View 三者的差别?
Activity 控制单元，Window 承载模型，View 显示视图

Activity在onCreate中调用attach方法，在attach方法中创建一个window对象。window对象创建时并没有创建DocerView对象，而是当用户在Activity中调用setContentView方法之后，接着转到window的setContentView，这时会检查DecorView是否存在，如果不存在则创建DecorView对象，然后把用户自己的View添加到DecorView中。

#### 2. PopupWindow 和 Dialog的区别
**不同点：**
1. Popupwindow在显示之前一定要设置宽高，Dialog无此限制。
2. Popupwindow默认不会响应物理键盘的back，除非显示设置了popup.setFocusable(true)；而在点击back的时候，Dialog会消失。
3. Popupwindow不会给页面其他的部分添加蒙层，而Dialog会。
4. Popupwindow没有标题，Dialog默认有标题，可以通过`dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);`取消标题

AlertDialog是**非阻塞式**对话框，AlertDialog弹出时，后台还可以做其他事情；而PopupWindow是**阻塞式**对话框，PopupWindow弹出直至PopupWindow推出前，程序会一直等待，等待dismiss方式执行后，程序才会想下执行。
这两种区别的表现是：

**共同点：**

1. 二者显示的时候都要设置Gravity，如不设置，Dialog默认是Gravity.CENTER
2. 二者都有默认的背景，都可以通过`setBackgroundDrawable(new ColorDrawable(android.R.color.transparent));`去掉。

#### 3. 自定义View的基本流程？
1. 覆写onDraw、onMeasure、onLayout
2. 覆写dispatchTouchEvent、onTouchEvent
3. 覆写自定义属性，编写attr.xml，然后在代码中通过TypedArray等类获取自定义属性值
4. 处理滑动冲突、像素转换等问题
   Other
5. 编写attr.xml文件
6. 在layout布局文件中引用，同时引用命名空间
7. 在自定义控件中进行读取（构造方法拿到attr.xml文件值）
8. 覆写onMeasure
9. 覆写onLayout

#### 4. getWidth()和getMeasureWidth()的区别？



## 序列化

### Parcelable
Parcelable 对象用来在进程间、Activity 间传递数据，保存实例状态也是用它，Bundle 是它的一个实现，最好只用它存储和传递少量数据，别超过 50k，否则既可能影响性能又可能导致崩溃

#### LayoutInflater
LayoutInflater只做一件事，就是把xml表述的layout转化为View。



#### 1. SpannableString 与 SpannableStringBuilder的区别？
SpannableString 长度不可变
SpannableStringBuilder 长度可变

#### 2. 布局文件中`@`和`?`的区别？
`@` 标记是引用一个实际的值（color, string, dimension...）。
`?` 标记是引用一个`style attribute`，其值取决于当前使用的主题。


## View 相关

### ListView

#### 1. ListView和RecyclerView的区别？

#### 2. 如何优化ListView？
- 复用convertView，使用ViewHolder
- item中有图片时，异步加载，适当对图片进行压缩
- 监听滚动事件，滑动时不加载图片
- 实现上拉加载更多，进行分页加载

### RecyclerView
#### 1. RecyclerView的拖拽怎么实现的？
#### 2. RecylcerView和ListView之间有什么区别？

### 事件分发机制

![viewgroup_touch](E:\Blog\res\viewgroup_touch.png)

| Touch 事件相关方法                                   | 方法功能 | ViewGroup | Activity |
| :--------------------------------------------------- | -------- | --------- | -------- |
| public boolean dispatchTouchEvent(MotionEvent ev)    | 事件分发 | Yes       | Yes      |
| public boolean onInterceptTouchEvent(MotionEvent ev) | 事件拦截 | Yes       | No       |
| public boolean onTouchEvent(MotionEvent ev)          | 事件响应 | Yes       | Yes      |

#### onTouch和onTouchEvent有什么区别，又该如何使用？

这两个方法都是在View的dispatchTouchEvent中调用的，onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回ture将事件消费掉，onTouchEvent将不再执行。

- android事件分发是先传递到ViewGroup，再由ViewGroup传递到View。
- ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouch方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。
- 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。

#### Touch事件的传递机制
- 所有Touch事件都被封装成MotionEvent对象，包括Touch的位置、时间、历史记录以及第几个手指（多点触控）等。
- 事件类型分为：
  - ACTION_DOWN
  - ACTION_UP
  - ACTION_MOVE
  - ACTION_POINTER_DOWN
  - ACTION_POINTER_UP
  - ACTION_POINTER_CANCEL
- 每个事件都是以ACTION_DOWN开始，以ACTION_UP结束。

1. 事件从Activity的dispatchTouchEvent开始传递，只要onInterceptTouchEvent不进行拦截，从最上层的View(ViewGroup)开始一直往下（子View）传递。子View可以通过onTouchEvent对事件进行处理。
2. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的onTouchEvent。
3. 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来。
4. OnTouchListener优先于onTouchEvent执行。


#### 你用过MD，你知道怎么定义一个Behavior吗？

15. 说说三级缓存、Handler机制 ？

16. 什么是类？什么是对象？类和对象之间的关系？
   类是对象的集合，是抽象的；对象是类的实例化，是具体的。

17. 简述C/C++的核心思想
   http://www.cnblogs.com/lanxuezaipiao/p/4135105.html


## Java相关

### 类的加载过程
   ![classload](E:\Blog\res\classload.jpg)

#### 1. 加载 loading
加载阶段主要完成[三件](http://www.chinabyte.com/keyword/%E4%B8%89%E4%BB%B6/)事，即通过一个类的全限定名来获取定义此类的二进制字节流，将这个字节流所代表的静态[存储](http://storage.chinabyte.com/)结构转化为方法区的运行时数据结构，在Java堆中生成一个代表此类的Class对象，作为访问方法区这些数据的入口。这个加载过程主要就是靠类加载器实现的，这个过程可以由用户自定义类的加载过程。

#### 2. 验证 verification
这个阶段目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。主要包括四种验证：

#### 3. 文件格式验证：
- 基于字节流验证，验证字节流是否符合Class文件格式的规范，并且能被当前虚拟机
- 元数据验证：基于方法区的存储结构验证，对字节码描述信息进行语义
- 字节码验证：基于方法区的存储结构验证，进行数据流和控制流的
- 符号引用验证：基于方法区的存储结构验证，发生在解析中，是否可以将符号引用成功解析为直接引用。

#### 4. 准备 preparation
仅仅为类变量(即static修饰的字段变量)分配内存并且设置该类变量的初始值即零值，这里不包含用final修饰的static，因为final在编译的时候就会分配了，同时这里也不会为实例变量分配初始化。类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。

#### 5. 解析 resolution
解析主要就是将常量池中的符号引用替换为直接引用的过程。符号引用就是一组符号来描述目标，可以是任何字面量，而直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。有类或接口的解析，字段解析，类方法解析，[接口方法](http://www.chinabyte.com/keyword/%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95/)解析。

这里要注意如果有一个同名字段同时出现在一个类的接口和父类中，那么编译器一般都会拒绝编译。

#### 6. 初始化 initialization
初始化阶段依旧是初始化类变量和其他资源，这里将执行用户的static字段和静态语句块的赋值操作。这个过程就是执行类构造器方法的过程。

方法是由编译器收集类中所有类变量的赋值动作和静态语句块的语句生成的，类构造器方法与实例构造器方法不同，这里面不用显示的调用父类的方法，父类的方法会自动先执行于子类的方法。即父类定义的静态语句块和静态字段都要优先子类的变量赋值操作。

### HashMap

#### HashMap的实现原理

### 泛型

#### 泛型的本质
**泛型的本质是参数化类型**，也就是说所操作的数据类型被指定为某个参数类型。
这种参数类型可以在类、接口和方法中创建，分别称为泛型类、泛型接口、泛型方法。

#### 泛型的优缺点
优点：在编译的时候检查类型安全，并且素有的强制转换都是自动和隐式的，提高代码的重用率。
缺点：



#### 1. String、StringBuffer、StringBuilder之间的区别?

- String：字符串常量
- StringBuffer：字符串变量，线程安全
- StringBuilder：字符串变量，线程非安全

三者在执行速度方面的比较：StringBuilder > StringBuffer > String

**总结**
1. 操作少量的数据用 String
2. 单线程操作字符串缓冲区下操作大量数据 StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据 StringBuffer



### Heap和Stack的区别



## 进程相关

### 进程间通信 IPC(Inter-Process Communication)

#### Serializable 接口

#### Parcelable 接口

#### Binder

### 线程池
#### 1. 那你说说线程池的四种初始化吧？

### AsyncTask
#### 1. AsyncTask内部维护了一个线程池，是串行还是并行，怎么维护的？
#### 2. 你用过AsyncTask吗？那你跟我说说AsyncTask的内部实现原理？

### Android中进程的优先级
前台进程->可见进程->服务进程->后台进程->空进程

### 应用无响应 ANR(Android Not Responding)
#### 1. ANR产生的原因?
1. 主线程中做了非常耗时的操作 
2. 在BroadcastReceiver里做耗时的操作或计算 
3. CPU使用过高 
4. 发生了死锁 等等。

#### 2. 如何定位和分析ANR？
一般情况下，如果有ANR发生，系统都会记录在`/data/anr/traces.txt`文件

使用adb命令查看分析ANR产生原因（需要root）
```shell
adb shell
cd data/anr
cat traces.txt
```
#### 3. 如何避免ANR ？
1. 运行在主线程里的任何方法都尽可能少做事情。特别是，Activity应该在它的关键生命周期方法（如onCreate()和onResume()）里尽可能少的去做创建操作。（可以采用重新开启子线程的方式，然后使用Handler+Message的方式做一些操作，比如更新主线程中的ui等）
2. 别在BroadcastReceiver里做耗时的操作


### 线程安全
**线程安全是指多线程访问统一代码，不会产生不确定的结果。**




## 网络相关

#### 1. 请描述一次网络请求过程？
1. 域名解析
2. TCP三次握手
3. 建立连接后发起Http请求
4. 服务器响应请求
5. 浏览器解析Html代码，同时请求资源
6. 浏览器渲染
7. TCP四次挥手 


## 性能优化相关
### 内存泄漏（OOM）
#### 1. 描述下内存泄漏的场景和解决方法？
#### 2. 如何避免OOM？

### JNI



## 其他

### Gradle
### Git
### 反射机制


## 设计模式



## 算法

1. 一个按升序排列好的数组int[] arry = {-5,-1,0,5,9,11,13,15,22,35,46},输入一个x，int x = 31，在数据中找出和为x的两个数，例如 9 + 22 = 31，要求算法的时间复杂度为O(n)。
2. 如何向一个数据库具有int类型A，B，C，D四列的表中随机插入10000条数据？如何按升序取出A列中前10个数？
