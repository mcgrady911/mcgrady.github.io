# Android View

[TOC]

## ImageView

**Scaletype** 属性
- CENTER：按图片的原来size居中显示，当图片长宽超过View的长宽，则截取图片的居中部分显示。
- `CENTER_TOP`：按比例扩大图片的size居中显示，使得图片长宽等于或大于View的长宽。
- `FIT_CENTER`：把图片按比例扩大或缩小到View的size，且居中显示。
- `FIT_END`：把图片按比例扩大或缩小到View的size，显示在View的下部分区域。
- `FIT_START`：把图片按比例扩大或缩小到View的size，显示在View的上部分区域。
- `FIT_XY`：不按比例缩放图片，将图片填满整个View。

*使用方法*：
```
// layout
android:scaleType="CENTER"

// class
imageView.setScaleType(ImageView.ScaleType.CENTER);
```

自定义View的实现方式大概可分为三种：
- 自绘控件
  - 这个View上所展现的内容全部都是我们自己绘制出来的（`onDraw()`）
- 组合控件
- 继承控件







