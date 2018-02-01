

# Activity横竖屏切换时的生命周期总结

[TOC]

---

## 横竖屏切换时Activity生命周期的变化
曾经遇到过一个面试题，让你写出横屏切换竖屏Activity的生命周期。现在给大家分析一下他切换时具体的生命周期是怎么样的：

1. 新建一个Activity，并把各个生命周期打印出来

2. 运行Activity，得到如下信息
```
onCreate-->
onStart-->
onResume-->
```

3. 按crtl+f12切换成横屏时
```
onSaveInstanceState-->
onPause-->
onStop-->
onDestroy-->
onCreate-->
onStart-->
onRestoreInstanceState-->
onResume-->
```

4. 再按crtl+f12切换成竖屏时，发现打印了两次相同的log
```
onSaveInstanceState-->
onPause-->
onStop-->
onDestroy-->
onCreate-->
onStart-->
onRestoreInstanceState-->
onResume-->
onSaveInstanceState-->
onPause-->
onStop-->
onDestroy-->
onCreate-->
onStart-->
onRestoreInstanceState-->
onResume-->
```

5. 修改AndroidManifest.xml，把该Activity添加 `android:configChanges="orientation"`，执行步骤3
```
onSaveInstanceState-->
onPause-->
onStop-->
onDestroy-->
onCreate-->
onStart-->
onRestoreInstanceState-->
onResume-->
```

6. 再执行步骤4，发现不会再打印相同信息，但多打印了一行`onConfigChanged`
```
onSaveInstanceState-->
onPause-->
onStop-->
onDestroy-->
onCreate-->
onStart-->
onRestoreInstanceState-->
onResume-->
onConfigurationChanged-->
```

7. 把步骤5的`android:configChanges="orientation"` 
   改成 `android:configChanges="orientation|keyboardHidden"`，执行步骤3，就只打印`onConfigChanged`
```
onConfigurationChanged-->
```

8. 执行步骤4
```
onConfigurationChanged-->
onConfigurationChanged-->
```

### 总结

1. 不设置Activity的`android:configChanges`时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

2. 设置Activity的`android:configChanges="orientation"`时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

3. 设置Activity的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新调用各个生命周期，只会执行`onConfigurationChanged`方法


>补充一点，当前Activity产生事件弹出Toast和AlertDialog的时候Activity的生命周期不会有改变

Activity运行时按下HOME键(跟被完全覆盖是一样的)：

```prolog
onSaveInstanceState-->onPause-->onStop       
onRestart-->onStart-->onResume
```

Activity未被完全覆盖只是失去焦点：
```prolog
onPause--->onResume
```

---
## 纠正Activity横竖屏切换的生命周期的错误

- 不设置`android:configChanges`时：
  - 切屏会重新调用各个生命周期(详细说明见Activity生命周期(一)),但不管是切横屏，还是竖屏，都是一次。
- 设置**android:configChages=”orientation”**时：
  - 结果和不设置一样，仍然是重新调用，而且横竖屏都是一次。
- 设置为`android:configChanges=”orientation|keyboardHidden”`时：
1. `android:targetSdkVersion` <= 12时，不会重新创建 
2. `android:targetSdkVersion` > 12时，和不设置一样，重新创建。 
3. 该点是网上获得的资料，没测试。在4.0以下的是不重建，而4.0以上的则为1，2所叙述（本测试机器为4.3）。
- 设置Activity的`android:configChanges=”orientation|keyboardHidden|screenSize”`时：
  - 不重新创建Activity。

最后补充一点说明，重新创建是指，当前你启动了一个Actvity，
```prolog
onCreate–>onStart–>onResume
```
此时切换屏幕时，会销毁当前Activity，重新生成一个。 
```prolog
onPause–>onStop–>onDestory–>onCreate–>onStart–>onResume
```

### 总结

在现在android普遍都是>4.0的版本下，以前的结论基本上都是错误的。出现这种情况，是因为很多人都是盲目的复制别人的结论、文章，而重来不会自己验证下而导致的。 

