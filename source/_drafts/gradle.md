

## build.gradle

### compileSdkVersion
**指明用哪个 Android SDK 版本编译你的应用**，强烈推荐总是使用最新的 SDK 进行编译（默认就是最新的）。

修改 compileSdkVersion 不会改变运行时的行为，当你修改了 `compileSdkVersion` 的时候，可能会出现新的编译警告、编译错误，但新的 **`compileSdkVersion` 不会被包含到 APK 中：它纯粹只是在编译的时候使用**。
>注意：
>如果使用 `Support Library` ，那么使用最新发布的 `Support Library` 就需要使用最新的 SDK 编译。



### minSdkVersion

**指明应用程序运行所需的最小API level**。如果不指明的话，默认是1。也就是说该应用兼容所有的android版本。

我们应该总是声明这个属性

1. 如果系统的API level低于`android:minSdkVersion`设定的值，那么android系统会阻止用户安装这个应用
2. 如果指明了这个属性，并且在项目中使用了高于这个API level的API， 那么会在编译时报错。(下面会讲解解决方法)


> 注意：
>
> 你所使用的库，如 `Support Library` 或 `Google Play services`，可能有他们自己的 `minSdkVersion`。你的应用设置的 `minSdkVersion` 必需大于等于这些库的 `minSdkVersion` 。



### targetSdkVersion

**targetSdkVersion 是 Android 提供向前兼容的主要依据**
将 target 更新为最新的 SDK 是所有应用都应该优先处理的事情。但这不意味着你一定要使用所有新引入的功能，也不意味着你可以不做任何测试就盲目地更新` targetSdkVersion `，**请一定在更新 `targetSdkVersion` 之前做测试**！

如果平台的API Level高于你的应用程序中的`targetSdkVersion`属性指定的值，系统会开启兼容行为来确保你的应用程序继续以期望的形式来运行。你可以通过指定`targetSdkVersion`来匹配运行程序的平台的 API level来禁用这种兼容性行为。



> 举例：
>
> 设置API level为11或更高，当你的应用运行在Android3.0或更高的系统上时，系统会为你的应用使用新的默认主题（Holo主题），并且当运行在大屏幕的设备上时会禁用屏幕兼容模式（screen compatibility mode），因为支持了 API level 11就暗示了支持大屏幕。



Android 高版本API方法在低版本系统上的兼容性处理

1. 用`@TargeApi($API_LEVEL)`使可以编译通过, 不建议使用`@SuppressLint("NewApi")`
   区别：

   - ` @SuppressLint("NewApi"）`屏蔽一切新api中才能使用的方法报的`android lint`错误

   - `@TargetApi()` 只屏蔽某一新API中才能使用的方法报的`android lint`错误
     举个例子，某个方法中使用了API9新加入的方法，而项目设置的`android:minSdkVersion=8`，此时在方法上加`@SuppressLint("NewApi"）`
     和`@TargetApi(Build.VERSION_CODES.GINGERBREAD)`都可以，以上是通用的情况。
     而当你在此方法中又引用了一个`API11`才加入的方法时，`@TargetApi(Build.VERSION_CODES.GINGERBREAD)`注解的方法又报错了，而
     `@SuppressLint("NewApi"）`不会报错，这就是区别。

2. 判断运行时版本，在低版本系统不调用此方法，同时为了保证功能的完整性，需要提供低版本功能实现。
   例如：

```java
public class MainActivity extends AppCompatActivity {
   private  AlertDialog.Builder builder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //如果API level大于11 大于11的时候能够指定主体
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB) {
             builder = new AlertDialog.Builder(this,
                    AlertDialog.THEME_HOLO_LIGHT);
        }else {
             builder = new AlertDialog.Builder(this);
        }
        
        builder.setItems(new String[] { "拍照","选择" },
                new DialogInterface.OnClickListener() {

                    @Override
                    public void onClick(DialogInterface dialog, int which) {

                    }
                });
        builder.setTitle("选择照片");
        builder.create().show();
    }
}
```



### 配置依赖包下载地址

用阿国内阿里云的依赖下载地址替换Google依赖包下载地址

```gr
repositories {
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
    ...
}

allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        ...
    }
} 
```

