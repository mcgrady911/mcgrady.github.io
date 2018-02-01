---
title: Android error collect
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [android, tips]
categories: android
---



### .9出错解决方案

**AS中要求.9图片四条边都绘制**
可通过以下方法取消AS对.9图片的安全检查

```java
// app/build.gradle

buildToolsVersion 1.0

// 取消掉系统对.9图片的检查
aaptOptions.cruncherEnabled = false
aaptOptions.useNewCruncher = false
```

### 使用泛型
```java
// DON'T
List sort(List input) {
  [...]
}
// DO
<T> List<T> sort(List<T> input) {
  [...]
}
```

