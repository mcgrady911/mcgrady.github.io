# Fragment

[TOC]

[版权声明](http://blog.csdn.net/lmj623565791/article/details/37970961)

---
## 什么是Fragment
**使用Fragment可以让更加充分地利用屏幕空间**。
**为了让界面可以在屏幕上更好地展示**，Android在3.0版本引入了Fragment(碎片)功能，它非常类似于Activity，可以像Activity一样包含布局。

---
## Fragment生命周期
### Fragment与Activity生命周期的对比
![Fragment](./Image.png) ![Fragment Activity](./20140719225005356.png)

**Fragment必须是依存与Activity而存在的，因此Activity的生命周期直接影响到Fragment的生命周期。**

可以看到Fragment比Activity多了几个额外的回掉方法：
- `onAttach(Activity)`
  当Fragment与Activity发生关联时调用
- `onCreateView(LayoutInflater, ViewGroup, Bundle)`
  创建该Fragment的试图
- `onActivityCreated(Bundle)`
  当Activity的onCreate()方法返回时调用
- `onDestoryView()`
  与onDestoryView()相对应，当该Fragment的视图被移除时调用
- `onDetach()`
  与onAttach()相对应，当Fragment与Activity关联被取消时调用

>注意：除了onCreateView()，如果重写了其他的所有方法，必须调用父类对于该方法的实现。

---
## 如何静态和动态的使用Fragment

### 静态的使用Fragment
1. 继承Fragment，重写onCreateView()决定Fragment的布局

```java
public class TitleFragment extends Fragment {  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, 
	    ViewGroup container,Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.layout_fragment, container, false); 
        return view;  
    }  
}  
```
2. 在Activity布局中声明Fragment
   `activity_main.xml`：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
  
    <fragment  
        android:id="@+id/id_fragment_title"  
        android:name="com.zhy.zhy_fragments.TitleFragment"  
        android:layout_width="fill_parent"  
        android:layout_height="45dp" /> 

</RelativeLayout>  
```

### 动态的使用Fragment
1. 在Activity布局中声明Fragment
   `activity_main.xml`：
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
    
	<FrameLayout  
	        android:id="@+id/id_fragment_content"
	        android:layout_width="fill_parent"  
	        android:layout_height="45dp" />    
	        
</RelativeLayout> 
```
2. 在Activity类中使用Fragment
```java
public class MainActivity extends Activity {

	private ContentFragment fragmentContent;
	@Override  
    protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        setContentView(R.layout.activity_main);  
		
		// set defalut fragment
		FragmentManager fm = getFragmentManager();  
        FragmentTransaction transaction = fm.beginTransaction();  
        fragmentContent = new ContentFragment();  
        transaction.replace(R.id.id_fragment_content, fragmentContent );  
        transaction.commit();  
	} 
}
```

### 使用Fragment时的注意事项

#### State loss 错误
主要是因为`commit`方法一定要在`Activity.onSaveInstance()`之前调用。

#### 切换Fragment时，对用户数据的处理
>情景：在Fragment_A中的EditText填入了一些数据，然后切换到Fragment_B时

- 如果希望回到Fragment_A后还能看到填入的数据，则适合用`hide/show`。
  **希望保留用户操作的面板，可以使用`hide/show`** 
- 若不希望保留用户操作，可以使用`remove/add`，或者`replace`，两者效果相同。

##### remove和detach的区别
在不考虑会退栈的情况下，`remove`会销毁整个Fragment实例，而`detach`则只是销毁其视图结构（实例不会被销毁）。
- 如果当前Activity一直存在，在不希望保留用户操作的时候，可以优先使用detach。

---
## 如何管理Fragment退回栈
`FragmentTransaction.addToBackStack(String)`

---
## Fragment事务

---
## Fragment的一些特殊用途

---
## 没有布局的Fragment有何用处

---
## Fragment如何与Activity交互
### Fragment与Activity通信
**所有的Fragment都是依附于Activity**
- 如果Activity中包含自己管理的Fragment的引用，可以通过引用直接访问所有的Fragment的public方法
- 如果Activity中未保存任何Fragment的引用，没关系，每个Fragment都有一个唯一的`TAG`/`ID`，可以通过getFragmentManager`.findFragmentByTag()`/`.findFragmentById()`获取任何Fragment实例，然后进行操作。
- 在Fragment中可以通过`getActivity()`得到当前绑定的Activity实例，然后进行操作。

> 注意：如果在Fragment中需要Context，可以通过调用`getActivity()`，如果该Context在Activity被销毁后还存在，则使用`getActivity().getApplicationContext()`。

因为要考虑Fragment的重复使用，所以必须降低Fragment与Activity的耦合，而且**Fragment更不应该直接操作别的Fragment**，毕竟**Fragment操作应该由它的管理者Activity来决定**。

当屏幕发生旋转时，Activity发生重新启动，默认的Activity中的Fragment也会随着Activity重新创建，这样会导致本身存在的Fragment重新启动，然后当执行Activity.onCreate()时，又重新实例化一个新的Fragment。
解决方法：通过onCreate(Bundle savedInstanceState)判断，当前是否发生Activity的重新创建（默认到底savedInstanceState会存储一些数据，包括Fragment的实例）。
```java
public class MainActivity extends Activity {
	private Fragment mFragmentOne;
	 @Override  
    protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		if (savedInstanceState == null) {
			mFragmentOne = new FragmentOne();
			FragmentMTransaction tx = getFragmentManager().beginTransaction();
			tx.add(R.id.id_fragment_content, mFragmentOne, "ONE");
			tx.commit();
		}
	}
```

---
## 没有布局的Fragment的作用
没有布局文件的fragment实际上是为了保存，当activity重启时，保存大量数据准备

---
## Fragment如何创建对话框
[Android 官方推荐：DialogFragment创建对话框](http://blog.csdn.net/lmj623565791/article/details/37815413)

---
## Fragment如何与ActionBar集成



