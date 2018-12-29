# Android开发 实践笔记



1. CountDownTimer 总时间最好加上16ms,不然一开始显示有问题；

2. 新浪微博配置的时候最后一个参数要和开发平台保持一致；

 ```java
PlatformConfig.setSinaWeibo("134xxx0589", "f33d8cb4dfab8419xxxxxx1c53b333", "http://sns.whalecloud.com/sina2/callback");
 ```

1. Android 自带浏览器对 Adobe Flashplayer WebGL CSS63D 的不友好支持；最后采用的是腾讯x5内核；

2. Https 证书支持（浏览器获取证书方式）；

3. 一个类中内部类又调用其他内部类的，混淆的时候似乎会有问题；

   1. 抄 umeng 混淆指定的代码的时候写入了下面这两行

      ```cm
      -dontshrink
      -dontoptimize
      ```

   2. 如果你执行了7.1，请记得在混淆里面多配置

      ```cm
      -keep class extends com.umeng.socialize.net.base.SocializeReseponse { ;
      }
      ```

   > 看这个名字就知道是不去做资源压缩和代码优化。如果你在 release 的时候指定了如下,不用怀疑，这里不会移除你不用的资源和相关代码。平时最好养成良好的习惯，产品或UI改动了界面，不要的资源文件及时移除，不要指望最后发release包的时候什么不用资源都可以自动给你移除。

   ```cm
   shrinkResources true
     minifyEnabled true
   
     proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
   ```
   > 这里使用的是 proguard-android-optimize.txt ,官方这样解释的：
   > 要想做进一步的代码压缩，请尝试使用位于同一位置的 proguard-android-optimize.txt 文件。它包括相同的 ProGuard 规则，但还包括其他在字节码一级（方法内和方法间）执行分析的优化，以进一步减小 APK 大小和帮助提高其运行速度。
   > PS：
   > 开启这个优化后，打包会变得更慢，毕竟优化.移除资源需要遍历耗时的咯。
   > 说到这里你也要小心引入或者打开了 -dontobfuscate ，这个就是说不混淆了。所以最后你是不是发现混淆和不混淆怎么都一样了？！ 我在抄 zxing 的时候不小心引入了。

4. java.lang.IllegalStateException: The specified child already has a parent 4.15
   搞了一上午，最后才发现是 mTopicAdapter.setList(mItems) 调用多次造成的；

5. 自定义 RadioButton 没有效果，看看是不是构造方法attrs没有写对，以后记得要去看看继承的View的构造方法是怎么写的；

   ```java
   public ThemeRadioButton(Context context, AttributeSet attrs) {
     this(context, attrs, android.R.attr.radioButtonStyle);
   }
   ```
6. RecyclerView 或者 SrollView 里面存在会获取焦点的 View（比如说RecyclerView 里含有 WebView ,或者 ScrollView 里面还有 RecycerView ,父控件添加属性 `android:descendantFocusability=”blocksDescendants” `可以避免其自动滚动；

7. RadioGroup 只能使用单行，而且回调似乎有点儿问题 ；

8. VersionName 等参数为空的时候 bugly 会报错，无法统计；

9. Gson 可以直接设置使用序列化的注解，这样就可以直接混淆model了，另外 model 不写get 或者 set 的方法也挺好，即减少了方法数量，调用的时候也相对方便，当然判空还是必须的，毕竟这不是 Kotlin ,想想 Message 对应的使用。

10. 父控件要获取到点击事件，需要将 Button .RadioButton 等子控件 设置 clickable focusable 为 false;

11. mRecycler.computeVerticalScrollOffset() 可用于判断 RecyclerView 滚动的距离，而不是使用 getScrollY() ;最后补充，这个方法返回的值还是有问题的，值会很诡异的骤变一下，具体的没有去研究，另外通过设置 OnScrollListener 获取的 dy 或者dx 在子View是动态测量设置宽度或者高度时也是有问题的；

12. CardView 默认是带有背景色的，在特定情况下，会出现背景色覆盖不了的情况。另外 CardView 阴影效果实现机制不一样，这个导致在5.0前后是有差异性的;
13. 友盟的QQ分享  需要记得替换，不然回调异常；

14. WebView 如果没有显示具体内容，检查是否是布局错误。

15. elevation 只设置某一边的效果，可以通过 setOutlineProvider() 来确定。
```java
setOutlineProvider(new ViewOutlineProvider() {

      @Override
      public void getOutline(View view, Outline outline) {
          outline.setRect(0, 6, view.getWidth(), view.getHeight());
      }

  })
```

17. 动态测量文字的行数

```java
mDes.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {

              @Override
              public boolean onPreDraw() {
    
                  //这个回调会调用多次，获取完行数记得注销监听
 mDes.getViewTreeObserver().removeOnPreDrawListener(this);
    
                  int lineCount1 = mDes.getLineCount();
    
                  Log.e("TreeObserver", "onBind: " + lineCount1);
    
                  if (lineCount1 < 4 || isExpand) {
                      mArrowContainer.setVisibility(View.GONE);
                  } else {
                      mArrowContainer.setVisibility(View.VISIBLE);
                  }
    
                  return true;
              }
    
          });
```

18. 使用 ViewPager 填充布局不显示：container.addView(iv)忘记调用。

23. 使用 Rxjava 的 mergeDelayError()方法时需要订阅在主线程的话，.observeOn(AndroidSchedulers.mainThread(), true)需要使用这个方法。

24. 使用 elevation 之后，这个布局上层的的控件将变得不可见。不知道这个是不是一个Bug。反正我被坑了。

25. Glide 默认使用的是 `DecodeFormat.PREFER_RGB_565` 的图片编码格式，如果有透明度，或者加载出现误差，应当要切换到 `DecodeFormat.PREFER_ARGB_8888`，具体设置方法可以看之前写的文章。

26. Glide 和 CircleImageView配合使用的时候，不能设置渐变动画。

27. 友盟分享或者就是QQ的分享，需要有外设读写权限，不然分享失败，而且，这个失败异常回调不在主线程。

28. 集成 Google Analytic 的时候，没法使用最新的版本，提示信息是version conflict ，然后呢，APP就莫名的崩溃，`Method 'void android.support.v4.c.d.<init>()' is inaccessible to class 'com.google.firebase.iid.zzg'`，说到底，这个还是集成版本太旧的问题，如何解决呢？似乎就是在 app/build.gradle 中添加 `apply plugin: 'com.google.gms.google-services'` 要加在最后面。链接

29. RecyclerView 更新数据如果有动画的话，那么应该先清除所有的数据，再添加新的数据，不然动画效果和以前的列表会同时出现，特别诡异。

30. Http Header默认的参数要确定好，原谅我们没有测试。

31. RecyclerView 的 LayoutParams 会是 MarginLayoutParams，如果重设一个 ViewGroup.LayoutParams将会报错，在 onBindViewHolder() 的时候，都会有LayoutParams，不用手动再去添加。

32. TextView 指定最大行数 应该用的都多，如果高度又需要固定，可以使用LinearLayout的weight指定，但是weight是不建议嵌套使用的，其实这里可以考虑使用 minLine 来限定最小高度，或者使用 minHeight 直接指定具体值。

33. 关于系统权限弹框，联想手机上面会影响到宿主 Activity 的状态，会回调 onSaveInstanceState() 的方法，如果在这之后需要弹出DialogFragment或者 执行FragmentTransaction 的 commit，将会抛出can not perform this action after onsaveinstancestate的异常。解决方法直接调用 commitAllowingStateLoss()。

34. gradle 不是正式版本的当然是只能测试，有些机型默认不支持安装，可以使用 adb install -t xxxxx.xxx 安装。

35. RecyclerView 发生： java.lang.IllegalArgumentException: Scrapped or attachedviews may not be recycled.最后发现就是自己在填充 ViewHolder 的时候，attachParent 一不小心设置为 true 了。

36. 水平方向的 ProgressBar，最好重设一下 progressdrawable， 不同设备上面的 drawable 样式不一致。给人的感觉就是有的有 padding 有的没有padding 。

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

  <item android:id="@android:id/background">
      <color android:color="@color/gray_trans"/>
      <shape>
          <corners android:radius="5dp"/>
      </shape>
  </item>


  <item android:id="@android:id/secondaryProgress">
      <clip>
          <shape >
              <corners android:radius="5dip"/>
              <gradient
                  android:angle="270"
                  android:centerColor="#80ffb600"
                  android:centerY="0.75"
                  android:endColor="#a0ffcb00"
                  android:startColor="#80ffd300"
                  />
          </shape>
      </clip>
  </item>
    
  <item android:id="@android:id/progress">
      <clip>
          <shape>
              <corners android:radius="5dip"/>
              <gradient
                  android:angle="270"
                  android:centerColor="#ffffb600"
                  android:centerY="0.75"
                  android:endColor="#ffffcb00"
                  android:startColor="#ffffd300"
                  />
          </shape>
      </clip>
  </item>
    
</layer-list>
```

33. Activity 的软件盘模式还是不要忽略，避免输入法在不该出现的地方自己出现。

38. RecyclerView 的 Adapter 可以设置独立 id 的模式，第一步，关联 RecycerView 之前设置
    adapter?.setHasStableIds(true) 时机不对会抛异常,第二步， adapter 的 getItemId() 返回独立的 id 。这个方案可以实现在调用 notifyDataSetChanged 实现刷新数据时可以按指定的 id 复用已存在的 hoder ，进而实现 item 的局部更新（比如说，仅刷新 item 里面的时间 ）。该方案不会影响 RecyclerView 的缓存策略。即你设了独立 id 的 holder ，如有必要，还是会优先被复用。

39. 接 37，RecyclerView 若要实现某种 type 类型的 Holder 完全独立，不被复用，最简单的是让 holder 的 setIsRecyclable()可以设置为 false，但是这个会导致一直创建.绑定 holder，数据无法做到复用。最优方案是考虑给这种类型每个 holder 都设置不同的 type，保证唯一性，这样就可以实现创建一次，holder 不被复用，数据可复用。这样的确违背了 RecyclerView 的初衷，因此该方案仅适合 item 数量少，要求独立不被复用的情况。

40. 多个 Fragment 实现懒加载，使用 ViewPager 管理 Fragment 的时候，可以通过 setUserVisibleHint() 获取到状态；如果直接 add().hide() 添加的 Fragment ，只能通过onHiddenChanged()来确定，第一个可见的 Fragment 第一次不会回调该方法。另外Activity 的 onResume()回调是所有状态的 Fragment ，所以要在里面检测是否是可见的 Fragment 。

41. CarView 设置 clipChildren = false 无效。