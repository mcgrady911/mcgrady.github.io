---
title: 自定义view
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [android]
categories: android
---





# 自定义View



## Attribute

指定UI的某种风格样式，比如`android:layout_width`就是一种attr。



| **类型**    | **作用**                                   |
| --------- | ---------------------------------------- |
| reference | 引用某一资源ID，例如：`@mipmap/xxx`                |
| dimension | 尺寸值，可以为`wrap_content` `match_parent` `xxdp` |
| color     | 颜色值，例如：`#000000`                         |
| boolean   | 布尔值                                      |
| integer   | 整型                                       |
| float     | 浮点型                                      |
| fraction  | 百分数                                      |
| string    | 字符串类型                                    |
| enum      | 枚举类型，各个取值互斥                              |
| flag      | 位或运算，各个取值可用`|`连接，例如：`android:windowSoftInputMode = "stateUnspecified | stateUnchanged　|　stateHidden"` |

### 自定义attr

- 在`res/values`目录下新建一个`attrs.xml`，定义属性组即可。

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <declare-styleable name="MyStyle">    
      <attr name="text" format="string" />        
      <attr name="weight" format="float" />
      <attr name="pic" format="reference" />
  </declare-styleable>
</resources>
```

- 使用自定义attr

```java
public CustomView(Context context, AttributeSet attrs) {
  super(context, attrs);
  TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.MyStyle);
  String name = ta.getString(R.styleable.MyStyle_text, "mcgrady");
   . . .
}
```

## Style

### 自定义Style


## Theme

### 自定义Theme
