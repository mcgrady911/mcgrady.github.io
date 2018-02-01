#Android Basics

[TOC]

## 四大组件
### Activity

#### 什么是Activity？
**Activity 是 Android 的应用程序组件，用于处理用户操作**。
Activity提供用户一个交互的接口，Activity本身没有界面，所以Activity创建了一个窗口，开发人员可以通过 setContentView(view) 接口把 UI 放到 Activity 创建的窗口上。

#### Activity生命周期
![](http://img.blog.csdn.net/20130828141902812)

| Activity生命周期回调方法 | 方法描述                                     |
| :--------------: | ---------------------------------------- |
|    onStart()     | **用户可见不可交互** 的状态，表示活动将被展现给用户。            |
|   onRestart()    | 处于停止状态的活动需要再次展现给用户的时候，触发该方法。             |
|    onResume()    | **用户可交互**状态，当活动和用户发送交互的时候，触发该方法。         |
|    onPause()     | 当一个正在前台运行的活动因其他活动需要前台运行而转入后台运行时，触发该方法。(这时需要将活动的状态持久化，比如正在编辑的数据库记录等) |
|     onStop()     | 当活动不需要展示给用户的时候，触发该方法。（如果内存吃紧，系统会直接结束这个活动，而不会触发 `onStop()`方法） |
|   onDestroy()    | 当活动销毁时，触发该方法。如果内存吃紧，系统会直接结束这个活动，而不会触发`onDestroy()`方法。 |

#### Activity 生命周期的3个嵌套循环
 ![09155920_UEQi](C:\Users\Rx6\Desktop\09155920_UEQi.jpg)

>在Activity生命周期之中，系统调用了App生命周期中的回调方法集，这些生命周期回调方法就像个金字塔。
>当系统创建了一个新的Activity实例，回调方法一级一级的从塔底向塔顶移动，当位于金字塔顶部的时候,这个Activity就位于用户前台，用户此时就可以与Activity互动了。
>当用户要离开Activity的时候，系统调用另外一串方法，使Activity的状态从塔顶移动到塔底。在有些情况下，Activity只是完成部分的状态迁移并且等待用户的指令，并重新回到塔顶的状态。

>根据Activity复杂度的不同，你或许不用实现所有的生命周期方法。可是，**理解每个生命周期回调函数的意义从而确保你的应用按照用户的期望正确的动作则非常重要**。正确的实现生命周期的回调方法，从而使应用正确的动作，主要有以下几点需要注意：
1. 确保应用在用户使用你的时候可以接电话或者切换到其他应用而不崩溃。
2. 确保你的应用在用户不使用的时候不消耗系统资源。
3. 确保用户在从其他的应用切换回你的应用的时候能够继续之前的工作
   在用户屏幕切换或者其他动作的时候不崩溃或者丢失用户数据


>**恰当的停止和重启你的Activity是其生命周期中非常重要的过程**，他们保证你的应用对用户来说一致都是可用的，并且不会丢失他们的数据，你需要：
1. 当用户从你的应用切换到其他的应用，你的Activity能够正确的停止；当用户从跟其他的应用切换回你的应用，你的应用可以正确的重启。
2. 当用户在你的Activity中启动了另外一个新的Activity，当前Activity在新的Activity创建之后停止；当用户点击返回按钮，原先的Activity可以正确重启。
3. 当用户在使用你的应用的时候接到一个电话，你的Activity可以正确的停止。

#### Activity的4种状态
##### Active/Running
当Activity运行在屏幕前台(处于当前任务活动栈的最上面),此时它获取了焦点能响应用户的操作,属于运行状态，同一个时刻只会有一个Activity 处于活动(Active)或运行(Running)状态

##### Paused
当Activity失去焦点但仍对用户可见(如在它之上有另一个透明的Activity或Toast、AlertDialog等弹出窗口时)它处于暂停状态。暂停的Activity仍然是存活状态(它保留着所有的状态和成员信息并保持和窗口管理器的连接),但是当系统内存极小时可以被系统杀掉

##### Stopped
完全被另一个Activity遮挡时处于停止状态,它仍然保留着所有的状态和成员信息。只是对用户不可见,当其他地方需要内存时它往往被系统杀掉

##### Dead
Activity 尚未被启动、已经被手动终止，或已经被系统回收时处于非活动的状态，要手动终止Activity，可以在程序中调用"finish"方法。如果是（按根据内存不足时的回收规则）被系统回收，可能是因为内存不足了，内存不足时，Dalvak 虚拟机会根据其内存回收规则来回收内存：
1. 先回收与其他Activity 或Service/Intent Receiver 无关的进程(即优先回收独立的Activity)因此建议,我们的一些(耗时)后台操作，最好是作成Service的形式
2. 不可见(处于Stopped状态的)Activity
3. Service进程(除非真的没有内存可用时会被销毁)
4. 非活动的可见的(Paused状态的)Activity
5. 当前正在运行（Active/Running状态的）Activity

#### Activity的4种启动模式
##### standard(标准模式)
**在这个模式下，每次激活Activity时都会创建Activity实例，并放入任务栈中。因此，可有多个相同的实例，也允许多个相同的Activity叠加。**



##### singleTop
**可有多个实例，但不允许多个相同的Activity叠加。**

即：当Activity位于栈顶时，再通过Intent转跳到本身Activity，则不会创建一个新的实例压入栈中。

>Activity1为`standard`模式，Activity2为`singleTop`模式

打开顺序：
```flow
a1=>end: Activity1
a2=>end: Activity2
a3=>end: Activity2
a1(right)->a2(right)->a3(right)
```
栈压入顺序：

```flow
a1=>end: Activity1
a2=>end: Activity2 
a3=>end: Activity2.onNewIntent()

a1(right)->a2(right)->a3(right)
```
打开顺序：
```flow
a1=>end: Activity1
a2=>end: Activity2
a11=>end: Activity1
a22=>end: Activity2

a1(right)->a2(right)->a11(right)->a22(right)
```
栈压入顺序：
>与打开顺序相同



##### singleTask

**在Task栈中将有且只有一个该Activity的实例。**

即：若Activity不存在，则会在当前Task创建一个新的实例，若存在，则会把Task中在其之上的求他Activity Destory掉并调用它的`onNewIntent()`方法。

例：若有三个Activity1、Activity2（singleTask）、Activity3，可相互启动，那么无论这个程序如何启动Activity，但Activity2有且只有一个，并且这三个Activity都在同一个Task里。



##### singleInstance

**有且只有一个实例，并且这个实例将独立运行在Task中，不允许有别的Activity存在**。

例：有三个Activity1、Activity2、Activity3，可相互启动，其中Activity2为`singeIntent`模式，假设程序从Activity开始运行，Activity1的`TaskID`为200，那么当Activity1启动Activity2时，Activity2强会新启动一个Task，及Activity2与Activity1不在同一个Task中运行。

假设Activity2的`TaskID`为201，再从Activity2启动Activity3时，Activity3的`TaskID`为200，也就是说它被压到了Activity1启动的任务栈中。

若在Other应用中打开Activity2，假设Other的`TaskID`为200，那么打开Activity，将会新建一个`TaskID`运行，假设`TaskID`为201，如果这是再从Activity2启动Activity1或者Activity3，则会在创建一个Task。因此，若操作步骤为`Other-->Activity2-->Activity1`，这个过程将设计到三个Task。

#### Activity横竖屏切换时的生命周期变化
1. 运行Activity
```
onCreate
onStart
onResume
```

2. 切换横屏
```
onSaveInstanceState
onPause
onStop
onDestroy
onCreate
onStart
onRestoreInstanceState
onResume
```

3. 切回竖屏（打印了两次相同的log）
```
onSaveInstanceSate
onPause
onStop
onDestory
onCreate
onStart
onRestoreInstanceState
onResume
onSaveInstanceState
onPause
onStop
onDestroy
onCreate
onStart
onRestoreInstanceState
onResume
```

4. 修改AndroidManifest.xml，添加 `android:configChanges="orientation"`，执行步骤3
```
onSaveInstanceState
onPause
onStop
onDestroy
onCreate
onStart
onRestoreInstanceState
onResume
```
5. 再执行步骤4
```
onSaveInstanceState
onPause
onStop
onDestroy
onCreate
onStart
onRestoreInstanceState
onResume
onConfigurationChanged
```
6. 修改`android:configChanges="orientation"`为`android:configChanges="orientation|keyboardHidden`，执行步骤3
```
onConfigurationChanged
```
7. 执行步骤4
```
onConfigurationChanged
onConfigurationChanged
```
**总结：**
- 不设置Activity的`android:configChanges`时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次
- 设置Activity的`android:configChanges="orientation"`时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次
- 设置Activity的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新调用各个生命周期，只会执行`onConfigurationChanged`方法

>补充：当前Activity产生事件弹出Toast和AlertDialog的时候Activity的生命周期不会有改变
>Activity运行时按下HOME键(跟被完全覆盖一样)
```
onSaveInstanceState--->onPause--->onStop       
onRestart--->onStart--->onResume
```
>Activity未被完全覆盖只是失去焦点：
```
onPause--->onResume
```

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
#### Service的生命周期
`onCreate()`该方法在服务被创建时调用，该方法只会被调用一次，无论调用多少次`startService()`或`bindService()`方法，服务也只被创建一次。

`onDestroy()`该方法在服务被终止时调用。

#### Service的启动方式
#### Service的应用场景？
#### Service进程的优先级？

### BroadcastReceiver
#### BroadcastReceiver 的生命周期
#### BroadcastReceiver 的应用场景


### ContentProvider



---

**五种布局： FrameLayout 、 LinearLayout 、 AbsoluteLayout 、 RelativeLayout 、 TableLayout 全都继承自ViewGroup，各自特点及绘制效率对比。**

*   FrameLayout(框架布局)

      此布局是五中布局中最简单的布局，Android中并没有对child view的摆布进行控制，这个布局中所有的控件都会默认出现在视图的左上角，我们可以使用``android:layout_margin``，``android:layout_gravity``等属性去控制子控件相对布局的位置。

*   LinearLayout(线性布局)

                                                                                                            一行只控制一个控件的线性布局，所以当有很多控件需要在一个界面中列出时，可以用LinearLayout布局。
                                                                                                            此布局有一个需要格外注意的属性:``android:orientation=“horizontal|vertical``。

    * 当`android:orientation="horizontal`时，*说明你希望将水平方向的布局交给**LinearLayout** *，其子元素的`android:layout_gravity="right|left"` 等控制水平方向的gravity值都是被忽略的，*此时**LinearLayout**中的子元素都是默认的按照水平从左向右来排*，我们可以用`android:layout_gravity="top|bottom"`等gravity值来控制垂直展示。
    * 反之，可以知道 当`android:orientation="vertical`时，**LinearLayout**对其子元素展示上的的处理方式。

*   AbsoluteLayout(绝对布局)

                                                                                                            可以放置多个控件，并且可以自己定义控件的x,y位置

*   RelativeLayout(相对布局)

                                                                                                            这个布局也是相对自由的布局，Android 对该布局的child view的 水平layout& 垂直layout做了解析，由此我们可以FrameLayout的基础上使用标签或者Java代码对垂直方向 以及 水平方向 布局中的views任意的控制.

    * 相关属性：

    ```

    　　android:layout_centerInParent="true|false"
    　　android:layout_centerHorizontal="true|false"
    　　android:layout_alignParentRight="true|false"
    　　
    ```

*   TableLayout(表格布局)

                                                                                                            将子元素的位置分配到行或列中，一个TableLayout由许多的TableRow组成


---

* 启动Activity:
  onCreate()—>onStart()—>onResume()，Activity进入运行状态。
* Activity退居后台:
  当前Activity转到新的Activity界面或按Home键回到主屏：
  onPause()—>onStop()，进入停滞状态。
* Activity返回前台:
  onRestart()—>**onStart()**—>onResume()，再次回到运行状态。
* Activity退居后台，且系统内存不足，
  系统会杀死这个后台状态的Activity（此时这个Activity引用仍然处在任务栈中，只是这个时候引用指向的对象已经为null），若再次回到这个Activity,则会走onCreate()–>onStart()—>onResume()(将重新走一次Activity的初始化生命周期)
* 锁屏：`onPause()->onStop()`
* 解锁：`onStart()->onResume()`



**通过Acitivty的xml标签来改变任务栈的默认行为**

*   使用``android:launchMode="standard|singleInstance|singleTask|singleTop"``来控制Acivity任务栈。

      **任务栈**是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其``onDestory()``方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

    * standard : 标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(*onCreate()->onStart()->onResume()*)都会执行。
    - singleTop : 栈顶复用模式.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的``onNewIntent()``方法会被回调.如果Activity已经存在但是不在栈顶,那么作用于*standard模式*一样.
    - singleTask: 栈内复用模式.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么这个Activity就会被调到栈顶,``onNewIntent()``,并且singleTask会清理在当前Activity上面的所有Activity.(clear top)
    - singleInstance : 加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了

Activity的堆栈管理以ActivityRecord为单位,所有的ActivityRecord都放在一个List里面.可以认为一个ActivityRecord就是一个Activity栈

---

**Activity缓存方法。**

有a、b两个Activity，当从a进入b之后一段时间，可能系统会把a回收，这时候按back，执行的不是a的onRestart而是onCreate方法，a被重新创建一次，这是a中的临时数据和状态可能就丢失了。

可以用Activity中的onSaveInstanceState()回调方法保存临时数据和状态，这个方法一定会在活动被回收之前调用。方法中有一个Bundle参数，putString()、putInt()等方法需要传入两个参数，一个键一个值。数据保存之后会在onCreate中恢复，onCreate也有一个Bundle类型的参数。

示例代码：

```
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

一、onSaveInstanceState (Bundle outState)

当某个activity变得“容易”被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。

注意上面的双引号，何为“容易”？言下之意就是该activity还没有被销毁，而仅仅是一种可能性。这种可能性有哪些？通过重写一个activity的所有生命周期的onXXX方法，包括onSaveInstanceState和onRestoreInstanceState方法，我们可以清楚地知道当某个activity（假定为activity A）显示在当前task的最上层时，其onSaveInstanceState方法会在什么时候被执行，有这么几种情况：

1、当用户按下HOME键时。

这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则

2、长按HOME键，选择运行其他的程序时。

3、按下电源按键（关闭屏幕显示）时。

4、从activity A中启动一个新的activity时。

5、屏幕方向切换时，例如从竖屏切换到横屏时。（如果不指定configchange属性）
在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行

总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据（当然你不保存那就随便你了）。另外，需要注意的几点：

1.布局中的每一个View默认实现了onSaveInstanceState()方法，这样的话，这个UI的任何改变都会自动的存储和在activity重新创建的时候自动的恢复。但是这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，将不会存储它的状态。

2.由于默认的onSaveInstanceState()方法的实现帮助UI存储它的状态，所以如果你需要覆盖这个方法去存储额外的状态信息时，你应该在执行任何代码之前都调用父类的onSaveInstanceState()方法（super.onSaveInstanceState()）。
既然有现成的可用，那么我们到底还要不要自己实现onSaveInstanceState()?这得看情况了，如果你自己的派生类中有变量影响到UI，或你程序的行为，当然就要把这个变量也保存了，那么就需要自己实现，否则就不需要。

3.由于onSaveInstanceState()方法调用的不确定性，你应该只使用这个方法去记录activity的瞬间状态（UI的状态）。不应该用这个方法去存储持久化数据。当用户离开这个activity的时候应该在onPause()方法中存储持久化数据（例如应该被存储到数据库中的数据）。

4.onSaveInstanceState()如果被调用，这个方法会在onStop()前被触发，但系统并不保证是否在onPause()之前或者之后触发。


二、onRestoreInstanceState (Bundle outState)

至于onRestoreInstanceState方法，需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，（本人注：我昨晚调试时就发现原来不一定成对被调用的！）

onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行

另外，onRestoreInstanceState的bundle参数也会传递到onCreate方法中，你也可以选择在onCreate方法中做数据还原。
还有onRestoreInstanceState在onstart之后执行。
至于这两个函数的使用，给出示范代码（留意自定义代码在调用super的前或后）：

```
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

---



**Fragment的生命周期和activity如何的一个关系**

这我们引用本知识库里的一张图片：
![Mou icon](https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/FlowchartDiagram.jpg?raw=true)


**为什么在Service中创建子线程而不是Activity中**

这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。


**Intent的使用方法，可以传递哪些数据类型。**

通过查询Intent/Bundle的API文档，我们可以获知，Intent/Bundle支持传递基本类型的数据和基本类型的数组数据，以及String/CharSequence类型的数据和String/CharSequence类型的数组数据。而对于其它类型的数据貌似无能为力，其实不然，我们可以在Intent/Bundle的API中看到Intent/Bundle还可以传递Parcelable（包裹化，邮包）和Serializable（序列化）类型的数据，以及它们的数组/列表数据。

所以要让非基本类型和非String/CharSequence类型的数据通过Intent/Bundle来进行传输，我们就需要在数据类型中实现Parcelable接口或是Serializable接口。

[http://blog.csdn.net/kkk0526/article/details/7214247](http://blog.csdn.net/kkk0526/article/details/7214247)

**Fragment生命周期**

![](http://7xntdm.com1.z0.glb.clouddn.com/fragment_lifecycle.png)

注意和Activity的相比的区别,按照执行顺序

* onAttach(),onDetach()
* onCreateView(),onDestroyView()

---

**Service的两种启动方法，有什么区别**
1.在Context中通过``public boolean bindService(Intent service,ServiceConnection conn,int flags)`` 方法来进行Service与Context的关联并启动，并且Service的生命周期依附于Context(**不求同时同分同秒生！但求同时同分同秒屎！！**)。

2.通过`` public ComponentName startService(Intent service)``方法去启动一个Service，此时Service的生命周期与启动它的Context无关。

3.要注意的是，whatever，**都需要在xml里注册你的Service**，就像这样:
```
<service
        android:name=".packnameName.youServiceName"
        android:enabled="true" />
```

**广播(Boardcast Receiver)的两种动态注册和静态注册有什么区别。**

* 静态注册：在AndroidManifest.xml文件中进行注册，当App退出后，Receiver仍然可以接收到广播并且进行相应的处理
* 动态注册：在代码中动态注册，当App退出后，也就没办法再接受广播了

---


**ContentProvider使用方法**

[http://blog.csdn.net/juetion/article/details/17481039](http://blog.csdn.net/juetion/article/details/17481039)

---

**目前能否保证service不被杀死**

**Service设置成START_STICKY**

* kill 后会被重启（等待5秒左右），重传Intent，保持与重启前一样

**提升service优先级**
* 在AndroidManifest.xml文件中对于intent-filter可以通过``android:priority = "1000"``这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，**同时适用于广播**。
* 【结论】目前看来，priority这个属性貌似只适用于broadcast，对于Service来说可能无效

**提升service进程优先级**
* Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收
* 当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以在startForeground()使用startForeground()将service放到前台状态。这样在低内存时被kill的几率会低一些。
* 【结论】如果在极度极度低内存的压力下，该service还是会被kill掉，并且不一定会restart()

**onDestroy方法里重启service**
* service +broadcast  方式，就是当service走ondestory()的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
* 也可以直接在onDestroy()里startService
* 【结论】当使用类似口口管家等第三方应用或是在setting里-应用-强制停止时，APP进程可能就直接被干掉了，onDestroy方法都进不来，所以还是无法保证

**监听系统广播判断Service状态**
* 通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限
* 【结论】这也能算是一种措施，不过感觉监听多了会导致Service很混乱，带来诸多不便

**在JNI层,用C代码fork一个进程出来**

* 这样产生的进程,会被系统认为是两个不同的进程.但是Android5.0之后可能不行

**root之后放到system/app变成系统级应用**

**大招: 放一个像素在前台(手机QQ)**


**动画有哪两类，各有什么特点？三种动画的区别**
* tween 补间动画。通过指定View的初末状态和变化时间、方式，对View的内容完成一系列的图形变换来实现动画效果。
  Alpha
  Scale
  Translate
  Rotate。

* frame 帧动画
  AnimationDrawable 控制
  animation-list xml布局

* PropertyAnimation 属性动画


**Android的数据存储形式。**
* SQLite：SQLite是一个轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。
  Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api

* SharedPreference： 除SQLite数据库外，另一种常用的数据存储方式，其本质就是一个xml文件，常用于存储较简单的参数设置。

* File： 即常说的文件（I/O）存储方法，常用语存储大数量的数据，但是缺点是更新数据将是一件困难的事情。

* ContentProvider: Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其又是必不可少的一种存储方式。例如音频，视频，图片和通讯录，一般都可以采用此种方式进行存储。每个Content Provider都会对外提供一个公共的URI（包装成Uri对象），如果应用程序有数据需要共享时，就需要使用Content Provider为这些数据定义一个URI，然后其他的应用程序就通过Content Provider传入这个URI来对数据进行操作。

---

**Sqlite的基本操作。**


[http://blog.csdn.net/zgljl2012/article/details/44769043](http://blog.csdn.net/zgljl2012/article/details/44769043)

---

**如何判断应用被强杀**

在Applicatio中定义一个static常量，赋值为－1，在欢迎界面改为0，如果被强杀，application重新初始化，在父类Activity判断该常量的值。

**应用被强杀如何解决**

如果在每一个Activity的onCreate里判断是否被强杀，冗余了，封装到Activity的父类中，如果被强杀，跳转回主界面，如果没有被强杀，执行Activity的初始化操作，给主界面传递intent参数，主界面会调用onNewIntent方法，在onNewIntent跳转到欢迎页面，重新来一遍流程。

**Json有什么优劣势。**

**怎样退出终止App**

**Asset目录与res目录的区别。**
res 目录下面有很多文件，例如 drawable,mipmap,raw 等。res 下面除了 raw 文件不会被压缩外，其余文件都会被压缩。同时 res目录下的文件可以通过R 文件访问。Asset 也是用来存储资源，但是 asset 文件内容只能通过路径或者 AssetManager 读取。 [官方文档](https://developer.android.com/studio/projects/index.html)

**Android怎么加速启动Activity。**
分两种情况，启动应用 和 普通Activity
启动应用 ：Application 的构造方法，onCreate 方法中不要进行耗时操作，数据预读取(例如 init 数据) 放在异步中操作
启动普通的Activity：A 启动B 时不要在 A 的 onPause 中执行耗时操作。因为 B 的 onResume 方法必须等待 A 的 onPause 执行完成后才能运行

**Android内存优化方法：ListView优化，及时关闭资源，图片缓存等等。**

**Android中弱引用与软引用的应用场景。**

**Bitmap的四种属性，与每种属性队形的大小。**


**View与View Group分类。自定义View过程：onMeasure()、onLayout()、onDraw()。**

如何自定义控件：

1. 自定义属性的声明和获取

    * 分析需要的自定义属性
    * 在res/values/attrs.xml定义声明
    * 在layout文件中进行使用
    * 在View的构造方法中进行获取       
2. 测量onMeasure
3. 布局onLayout(ViewGroup)
4. 绘制onDraw
5. onTouchEvent
6. onInterceptTouchEvent(ViewGroup)
7. 状态的恢复与保存


**Android长连接，怎么处理心跳机制。**

---

**View树绘制流程**

---

**下拉刷新实现原理**

---

**你用过什么框架，是否看过源码，是否知道底层原理。**

Retrofit

EventBus

glide

---

**Android5.0、6.0新特性。**

Android5.0新特性：

* MaterialDesign设计风格
* 支持多种设备
* 支持64位ART虚拟机

Android6.0新特性

* 大量漂亮流畅的动画
* 支持快速充电的切换
* 支持文件夹拖拽应用
* 相机新增专业模式

Android7.0新特性

* 分屏多任务
* 增强的Java8语言模式
* 夜间模式

---


**Context区别**

* Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper
* 每一个Activity和Service以及Application的Context都是一个新的ContextImpl对象
* getApplication()用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法，getApplicationContext()比getApplication()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。
* Activity在创建的时候会new一个ContextImpl对象并在attach方法中关联它，Application和Service也差不多。ContextWrapper的方法内部都是转调ContextImpl的方法
* 创建对话框传入Application的Context是不可以的
* 尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存
* Context的数量等于Activity的个数 + Service的个数 + 1，这个1为Application

---

**IntentService的使用场景与特点。**

>IntentService是Service的子类，是一个异步的，会自动停止的服务，很好解决了传统的Service中处理完耗时操作忘记停止并销毁Service的问题

优点：

* 一方面不需要自己去new Thread
* 另一方面不需要考虑在什么时候关闭该Service

onStartCommand中回调了onStart，onStart中通过mServiceHandler发送消息到该handler的handleMessage中去。最后handleMessage中回调onHandleIntent(intent)。

---


**图片缓存**

查看每个应用程序最高可用内存：

```
    int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);  
    Log.d("TAG", "Max memory is " + maxMemory + "KB");  
```

---

**Gradle**

构建工具、Groovy语法、Java

Jar包里面只有代码，aar里面不光有代码还包括代码还包括资源文件，比如 drawable 文件，xml 资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度

---

**你是如何自学Android**

首先是看书和看视频敲代码，然后看大牛的博客，做一些项目，向github提交代码，觉得自己API掌握的不错之后，开始看进阶的书，以及看源码，看完源码学习到一些思想，开始自己造轮子，开始想代码的提升，比如设计模式，架构，重构等。

---

### ANR(Application Not Responding) 应用程序无响应
在 Android 上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应 ANR。用户可以选择“等待”而让程序继续运行，也可以选择“强制关闭”。默认情况下，在 android 中 Activity 的最长执行时间是 5 秒，BroadcastReceiver 的最长执行时间则是 10 秒。
- 三种常见类型：
  - **KeyDispatchTimeout (5s) `主要类型`**
    按键或触摸事件在特定时间内无响应
  - **BroadcastTimeout (10s)**
    BroadcastReceiver 在特定时间内无法处理完成
  - **ServiceTimeout (20s) `小概率类型`**
    Service 在特定的时间内无法处理完成

### 常见的Java Exception（RuntimeException）异常
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




## Fragment

为了同时适配手机和平板、UI和逻辑的共享。

###介绍
* Fragment也会被加入回退栈中。
* Fragment拥有自己的生命周期和接受、处理用户的事件
* 可以动态的添加、替换和移除某个Fragment

###生命周期
* 必须依存于Activity

![Mou icon](http://img.blog.csdn.net/20140719225005356?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* Fragment依附于Activity的生命状态
* ![Mou icon](https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/FlowchartDiagram.jpg?raw=true)


###Fragment生命周期方法含义：
* `public void onAttach(Context context)`
  * onAttach方法会在Fragment于窗口关联后立刻调用。从该方法开始，就可以通过Fragment.getActivity方法获取与Fragment关联的窗口对象，但因为Fragment的控件未初始化，所以不能够操作控件。

* `public void onCreate(Bundle savedInstanceState)`
  * 在调用完onAttach执行完之后立刻调用onCreate方法，可以在Bundle对象中获取一些在Activity中传过来的数据。通常会在该方法中读取保存的状态，获取或初始化一些数据。在该方法中不要进行耗时操作，不然窗口不会显示。

* `public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState)`
  * 该方法是Fragment很重要的一个生命周期方法，因为会在该方法中创建在Fragment显示的View，其中inflater是用来装载布局文件的，container是`<fragment>`标签的父标签对应对象，savedInstanceState参数可以获取Fragment保存的状态，如果未保存那么就为null。

* `public void onViewCreated(View view,Bundle savedInstanceState)`
  * Android在创建完Fragment中的View对象之后，会立刻回调该方法。其种view参数就是onCreateView中返回的view，而bundle对象用于一般用途。

* `public void onActivityCreated(Bundle savedInstanceState)`
  * 在Activity的onCreate方法执行完之后，Android系统会立刻调用该方法，表示窗口已经初始化完成，从这一个时候开始，就可以在Fragment中使用getActivity().findViewById(Id);来操控Activity中的view了。

* `public void onStart()`
  * 这个没啥可讲的，但有一个细节需要知道，当系统调用该方法的时候，fragment已经显示在ui上，但还不能进行互动，因为onResume方法还没执行完。

* `public void onResume()`
  * 该方法为fragment从创建到显示Android系统调用的最后一个生命周期方法，调用完该方法时候，fragment就可以与用户互动了。

* `public void onPause()`
  * fragment由活跃状态变成非活跃状态执行的第一个回调方法，通常可以在这个方法中保存一些需要临时暂停的工作。如保存音乐播放进度，然后在onResume中恢复音乐播放进度。

* `public void onStop()`
  * 当onStop返回的时候，fragment将从屏幕上消失。

* `public void onDestoryView()`
  * 该方法的调用意味着在 `onCreateView` 中创建的视图都将被移除。

* `public void onDestroy()`
  * Android在Fragment不再使用时会调用该方法，要注意的是~这时Fragment还和Activity藕断丝连！并且可以获得Fragment对象，但无法对获得的Fragment进行任何操作（呵~呵呵~我已经不听你的了）。

* `public void onDetach()`
  * 为Fragment生命周期中的最后一个方法，当该方法执行完后，Fragment与Activity不再有关联。

###Fragment比Activity多了几个额外的生命周期回调方法：
* onAttach(Activity):当Fragment和Activity发生关联时使用
* onCreateView(LayoutInflater, ViewGroup, Bundle):创建该Fragment的视图
* onActivityCreate(Bundle):当Activity的onCreate方法返回时调用
* onDestoryView():与onCreateView相对应，当该Fragment的视图被移除时调用
* onDetach():与onAttach相对应，当Fragment与Activity关联被取消时调用

>注意：除了onCreateView，其他的所有方法如果你重写了，必须调用父类对于该方法的实现

###Fragment与Activity之间的交互
* Fragment与Activity之间的交互可以通过`Fragment.setArguments(Bundle args)`以及`Fragment.getArguments()`来实现。

###Fragment状态的持久化。
由于Activity会经常性的发生配置变化，所以依附它的Fragment就有需要将其状态保存起来问题。下面有两个常用的方法去将Fragment的状态持久化。

* 方法一：
  * 可以通过`protected void onSaveInstanceState(Bundle outState)`,`protected void onRestoreInstanceState(Bundle savedInstanceState)` 状态保存和恢复的方法将状态持久化。
* 方法二(更方便,让Android自动帮我们保存Fragment状态)：
  * 我们只需要将Fragment在Activity中作为一个变量整个保存，只要保存了Fragment，那么Fragment的状态就得到保存了，所以呢.....
    * `FragmentManager.putFragment(Bundle bundle, String key, Fragment fragment)` 是在Activity中保存Fragment的方法。
    * `FragmentManager.getFragment(Bundle bundle, String key)` 是在Activity中获取所保存的Frament的方法。
  * 很显然，key就传入Fragment的id，fragment就是你要保存状态的fragment，但，我们注意到上面的两个方法，第一个参数都是Bundle，这就意味着*FragmentManager*是通过Bundle去保存Fragment的。但是，这个方法仅仅能够保存Fragment中的控件状态，比如说EditText中用户已经输入的文字（*注意！在这里，控件需要设置一个id，否则Android将不会为我们保存控件的状态*），而Fragment中需要持久化的变量依然会丢失，但依然有解决办法，就是利用方法一！
  * 下面给出状态持久化的事例代码：
    ```		
    	 /** Activity中的代码 **/
    	 FragmentB fragmentB;

        @Override
        protected void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.fragment_activity);
            if( savedInstanceState != null ){
                fragmentB = (FragmentB) getSupportFragmentManager().getFragment(savedInstanceState,"fragmentB");
            }
            init();
        }

        @Override
        protected void onSaveInstanceState(Bundle outState) {
            if( fragmentB != null ){
                getSupportFragmentManager().putFragment(outState,"fragmentB",fragmentB);
            }

            super.onSaveInstanceState(outState);
        }

        /** Fragment中保存变量的代码 **/

        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            AppLog.e("onCreateView");
            if ( null != savedInstanceState ){
                String savedString = savedInstanceState.getString("string");
                //得到保存下来的string
            }
            View root = inflater.inflate(R.layout.fragment_a,null);
            return root;
        }

    	@Override
        public void onSaveInstanceState(Bundle outState) {
            outState.putString("string","anAngryAnt");
            super.onSaveInstanceState(outState);
        }

    ```

###静态的使用Fragment
1. 继承Fragment，重写onCreateView决定Fragment的布局
2. 在Activity中声明此Fragment,就和普通的View一样

###Fragment常用的API
* android.support.v4.app.Fragment 主要用于定义Fragment
* android.support.v4.app.FragmentManager 主要用于在Activity中操作Fragment，可以使用FragmentManager.findFragmenById，FragmentManager.findFragmentByTag等方法去找到一个Fragment
* android.support.v4.app.FragmentTransaction 保证一些列Fragment操作的原子性，熟悉事务这个词
* 主要的操作都是FragmentTransaction的方法
  (一般我们为了向下兼容，都使用support.v4包里面的Fragment)
  <code>

  getFragmentManager() // Fragment若使用的是support.v4包中的，那就使用getSupportFragmentManager代替

</code>

* 主要的操作都是FragmentTransaction的方法

```
	
	FragmentTransaction transaction = fm.benginTransatcion();//开启一个事务
	transaction.add() 
	//往Activity中添加一个Fragment

	transaction.remove() 
	//从Activity中移除一个Fragment，如果被移除的Fragment没有添加到回退栈（回退栈后面会详细说），这个Fragment实例将会被销毁。

	transaction.replace()
	//使用另一个Fragment替换当前的，实际上就是remove()然后add()的合体~

	transaction.hide()
	//隐藏当前的Fragment，仅仅是设为不可见，并不会销毁

	transaction.show()
	//显示之前隐藏的Fragment

	detach()
	//当fragment被加入到回退栈的时候，该方法与*remove()*的作用是相同的，
	//反之，该方法只是将fragment从视图中移除，
	//之后仍然可以通过*attach()*方法重新使用fragment，
	//而调用了*remove()*方法之后，
	//不仅将Fragment从视图中移除，fragment还将不再可用。

	attach()
	//重建view视图，附加到UI上并显示。

	transatcion.commit()
	//提交一个事务

```

###管理Fragment回退栈
* 跟踪回退栈状态
  * 我们通过实现*``OnBackStackChangedListener``*接口来实现回退栈状态跟踪，具体如下
   ```
   public class XXX implements FragmentManager.OnBackStackChangedListener 

   /** 实现接口所要实现的方法 **/

  @Override
    public void onBackStackChanged() {
        //do whatevery you want
    }

    /** 设置回退栈监听接口 **／
    getSupportFragmentManager().addOnBackStackChangedListener(this);
   ```


 	```
​	
* 管理回退栈
  * ``FragmentTransaction.addToBackStack(String)`` *--将一个刚刚添加的Fragment加入到回退栈中*
  * ``getSupportFragmentManager().getBackStackEntryCount()`` *－获取回退栈中实体数量*
  * ``getSupportFragmentManager().popBackStack(String name, int flags)`` *－根据name立刻弹出栈顶的fragment*
  * ``getSupportFragmentManager().popBackStack(int id, int flags)`` *－根据id立刻弹出栈顶的fragment*

##Android性能优化

###合理管理内存

####节制的使用Service
如果应用程序需要使用Service来执行后台任务的话，只有当任务正在执行的时候才应该让Service运行起来。当启动一个Service时，系统会倾向于将这个Service所依赖的进程进行保留，系统可以在LRUcache当中缓存的进程数量也会减少，导致切换程序的时候耗费更多性能。我们可以使用IntentService，当后台任务执行结束后会自动停止，避免了Service的内存泄漏。

####当界面不可见时释放内存
当用户打开了另外一个程序，我们的程序界面已经不可见的时候，我们应当将所有和界面相关的资源进行释放。重写Activity的onTrimMemory()方法，然后在这个方法中监听TRIM_MEMORY_UI_HIDDEN这个级别，一旦触发说明用户离开了程序，此时就可以进行资源释放操作了。

####当内存紧张时释放内存
onTrimMemory()方法还有很多种其他类型的回调，可以在手机内存降低的时候及时通知我们，我们应该根据回调中传入的级别来去决定如何释放应用程序的资源。

####避免在Bitmap上浪费内存
读取一个Bitmap图片的时候，千万不要去加载不需要的分辨率。可以压缩图片等操作。

####是有优化过的数据集合
Android提供了一系列优化过后的数据集合工具类，如SparseArray、SparseBooleanArray、LongSparseArray，使用这些API可以让我们的程序更加高效。HashMap工具类会相对比较低效，因为它需要为每一个键值对都提供一个对象入口，而SparseArray就避免掉了基本数据类型转换成对象数据类型的时间。

####知晓内存的开支情况
* 使用枚举通常会比使用静态常量消耗两倍以上的内存，尽可能不使用枚举
* 任何一个Java类，包括匿名类、内部类，都要占用大概500字节的内存空间
* 任何一个类的实例要消耗12-16字节的内存开支，因此频繁创建实例也是会在一定程序上影响内存的
* 使用HashMap时，即使你只设置了一个基本数据类型的键，比如说int，但是也会按照对象的大小来分配内存，大概是32字节，而不是4字节，因此最好使用优化后的数据集合

####谨慎使用抽象编程
在Android使用抽象编程会带来额外的内存开支，因为抽象的编程方法需要编写额外的代码，虽然这些代码根本执行不到，但是也要映射到内存中，不仅占用了更多的内存，在执行效率上也会有所降低。所以需要合理的使用抽象编程。

####尽量避免使用依赖注入框架
使用依赖注入框架貌似看上去把findViewById()这一类的繁琐操作去掉了，但是这些框架为了要搜寻代码中的注解，通常都需要经历较长的初始化过程，并且将一些你用不到的对象也一并加载到内存中。这些用不到的对象会一直站用着内存空间，可能很久之后才会得到释放，所以可能多敲几行代码是更好的选择。

####使用多个进程
谨慎使用，多数应用程序不该在多个进程中运行的，一旦使用不当，它甚至会增加额外的内存而不是帮我们节省内存。这个技巧比较适用于哪些需要在后台去完成一项独立的任务，和前台是完全可以区分开的场景。比如音乐播放，关闭软件，已经完全由Service来控制音乐播放了，系统仍然会将许多UI方面的内存进行保留。在这种场景下就非常适合使用两个进程，一个用于UI展示，另一个用于在后台持续的播放音乐。关于实现多进程，只需要在Manifast文件的应用程序组件声明一个android:process属性就可以了。进程名可以自定义，但是之前要加个冒号，表示该进程是一个当前应用程序的私有进程。

###分析内存的使用情况

系统不可能将所有的内存都分配给我们的应用程序，每个程序都会有可使用的内存上限，被称为堆大小。不同的手机堆大小不同，如下代码可以获得堆大小：

```
ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
int heapSize = manager.getMemoryClass();
```
结果以MB为单位进行返回，我们开发时应用程序的内存不能超过这个限制，否则会出现OOM。

####Android的GC操作
Android系统会在适当的时机触发GC操作，一旦进行GC操作，就会将一些不再使用的对象进行回收。GC操作会从一个叫做Roots的对象开始检查，所有它可以访问到的对象就说明还在使用当中，应该进行保留，而其他的对系那个就表示已经不再被使用了。

####Android中内存泄漏
Android中的垃圾回收机制并不能防止内存泄漏的出现导致内存泄漏最主要的原因就是某些长存对象持有了一些其它应该被回收的对象的引用，导致垃圾回收器无法去回收掉这些对象，也就是出现内存泄漏了。比如说像Activity这样的系统组件，它又会包含很多的控件甚至是图片，如果它无法被垃圾回收器回收掉的话，那就算是比较严重的内存泄漏情况了。
举个例子，在MainActivity中定义一个内部类，实例化内部类对象，在内部类新建一个线程执行死循环，会导致内部类资源无法释放，MainActivity的控件和资源无法释放，导致OOM,可借助一系列工具，比如LeakCanary。

###高性能编码优化

都是一些微优化，在性能方面看不出有什么显著的提升的。使用合适的算法和数据结构是优化程序性能的最主要手段。

####避免创建不必要的对象
不必要的对象我们应该避免创建：

* 如果有需要拼接的字符串，那么可以优先考虑使用StringBuffer或者StringBuilder来进行拼接，而不是加号连接符，因为使用加号连接符会创建多余的对象，拼接的字符串越长，加号连接符的性能越低。
* 在没有特殊原因的情况下，尽量使用基本数据类型来代替封装数据类型，int比Integer要更加有效，其它数据类型也是一样。
* 当一个方法的返回值是String的时候，通常需要去判断一下这个String的作用是什么，如果明确知道调用方会将返回的String再进行拼接操作的话，可以考虑返回一个StringBuffer对象来代替，因为这样可以将一个对象的引用进行返回，而返回String的话就是创建了一个短生命周期的临时对象。
* 基本数据类型的数组也要优于对象数据类型的数组。另外两个平行的数组要比一个封装好的对象数组更加高效，举个例子，Foo[]和Bar[]这样的数组，使用起来要比Custom(Foo,Bar)[]这样的一个数组高效的多。

尽可能地少创建临时对象，越少的对象意味着越少的GC操作。

####静态优于抽象
如果你并不需要访问一个对系那个中的某些字段，只是想调用它的某些方法来去完成一项通用的功能，那么可以将这个方法设置成静态方法，调用速度提升15%-20%，同时也不用为了调用这个方法去专门创建对象了，也不用担心调用这个方法后是否会改变对象的状态(静态方法无法访问非静态字段)。

####对常量使用static final修饰符
```
static int intVal = 42;  
static String strVal = "Hello, world!";  
```
编译器会为上面的代码生成一个初始方法，称为<clinit>方法，该方法会在定义类第一次被使用的时候调用。这个方法会将42的值赋值到intVal当中，从字符串常量表中提取一个引用赋值到strVal上。当赋值完成后，我们就可以通过字段搜寻的方式去访问具体的值了。

final进行优化:

```
static final int intVal = 42;  
static final String strVal = "Hello, world!";  
```

这样，定义类就不需要<clinit>方法了，因为所有的常量都会在dex文件的初始化器当中进行初始化。当我们调用intVal时可以直接指向42的值，而调用strVal会用一种相对轻量级的字符串常量方式，而不是字段搜寻的方式。

这种优化方式只对基本数据类型以及String类型的常量有效，对于其他数据类型的常量是无效的。

####使用增强型for循环语法

```
static class Counter {  
    int mCount;  
}  
  
Counter[] mArray = ...  
  
public void zero() {  
    int sum = 0;  
    for (int i = 0; i < mArray.length; ++i) {  
        sum += mArray[i].mCount;  
    }  
}  
  
public void one() {  
    int sum = 0;  
    Counter[] localArray = mArray;  
    int len = localArray.length;  
    for (int i = 0; i < len; ++i) {  
        sum += localArray[i].mCount;  
    }  
}  
  
public void two() {  
    int sum = 0;  
    for (Counter a : mArray) {  
        sum += a.mCount;  
    }  
}  
```

zero()最慢，每次都要计算mArray的长度，one()相对快得多，two()fangfa在没有JIT(Just In Time Compiler)的设备上是运行最快的，而在有JIT的设备上运行效率和one()方法不相上下，需要注意这种写法需要JDK1.5之后才支持。

Tips:ArrayList手写的循环比增强型for循环更快，其他的集合没有这种情况。因此默认情况下使用增强型for循环，而遍历ArrayList使用传统的循环方式。

####多使用系统封装好的API

系统提供不了的Api完成不了我们需要的功能才应该自己去写，因为使用系统的Api很多时候比我们自己写的代码要快得多，它们的很多功能都是通过底层的汇编模式执行的。
举个例子，实现数组拷贝的功能，使用循环的方式来对数组中的每一个元素一一进行赋值当然可行，但是直接使用系统中提供的System.arraycopy()方法会让执行效率快9倍以上。

####避免在内部调用Getters/Setters方法

面向对象中封装的思想是不要把类内部的字段暴露给外部，而是提供特定的方法来允许外部操作相应类的内部字段。但在Android中，字段搜寻比方法调用效率高得多，我们直接访问某个字段可能要比通过getters方法来去访问这个字段快3到7倍。但是编写代码还是要按照面向对象思维的，我们应该在能优化的地方进行优化，比如避免在内部调用getters/setters方法。

###布局优化技巧

####重用布局文件
**<include>**

<include>标签可以允许在一个布局当中引入另一个布局，那么比如说我们程序的所有界面都有一个公共的部分，这个时候最好的做法就是将这个公共的部分提取到一个独立的布局中，然后每个界面的布局文件当中来引用这个公共的布局。

Tips:如果我们要在<include>标签中覆写layout属性，必须要将layout_width和layout_height这两个属性也进行覆写，否则覆写xiaoguo将不会生效。

**<merge>**

<merge>标签是作为<include>标签的一种辅助扩展来使用的，它的主要作用是为了防止在引用布局文件时引用文件时产生多余的布局嵌套。布局嵌套越多，解析起来就越耗时，性能就越差。因此编写布局文件时应该让嵌套的层数越少越好。

举例：比如在LinearLayout里边使用一个<include>布局。<include>里边又有一个LinearLayout，那么其实就存在了多余的布局嵌套，使用merge可以解决这个问题。

####仅在需要时才加载布局

某个布局当中的元素不是一起显示出来的，普通情况下只显示部分常用的元素，而那些不常用的元素只有在用户进行特定操作时才会显示出来。

举例：填信息时不是需要全部填的，有一个添加更多字段的选项，当用户需要添加其他信息的时候，才将另外的元素显示到界面上。用VISIBLE性能表现一般，可以用ViewStub。ViewStub也是View的一种，但是没有大小，没有绘制功能，也不参与布局，资源消耗非常低，可以认为完全不影响性能。

```
<ViewStub   
        android:id="@+id/view_stub"  
        android:layout="@layout/profile_extra"  
        android:layout_width="match_parent"  
        android:layout_height="wrap_content"  
        />  
```

```
public void onMoreClick() {  
    ViewStub viewStub = (ViewStub) findViewById(R.id.view_stub);  
    if (viewStub != null) {  
        View inflatedView = viewStub.inflate();  
        editExtra1 = (EditText) inflatedView.findViewById(R.id.edit_extra1);  
        editExtra2 = (EditText) inflatedView.findViewById(R.id.edit_extra2);  
        editExtra3 = (EditText) inflatedView.findViewById(R.id.edit_extra3);  
    }  
}  
```

>tips：ViewStub所加载的布局是不可以使用<merge>标签的，因此这有可能导致加载出来出来的布局存在着多余的嵌套结构。