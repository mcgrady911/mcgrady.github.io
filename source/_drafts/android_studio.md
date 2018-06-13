# Android Studio 3.0

## 依赖变化

### compile（api）
这种是我们最常用的方式，使用该方式依赖的库将会参与编译和打包。

### provided（compileOnly）
只在编译时有效，不会参与打包，可以在自己的`moudle`中使用该方式依赖。比如`com.android.support`，`gson`这些使用者常用的库，避免冲突。

### apk（runtimeOnly）
只在生成apk的时候参与打包，编译时不会参与，很少用。

### testCompile（testImplementation）

### testCompile 
只在单元测试代码的编译以及最终打包测试apk时有效。

### debugCompile（debugImplementation）

### debugCompile 
只在debug模式的编译和最终的debug apk打包时有效。

### releaseCompile（releaseImplementation）