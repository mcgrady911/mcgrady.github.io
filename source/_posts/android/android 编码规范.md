---
title: android 编码规范
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [android, tips]
categories: android
---

## 摘要

- 把密码和敏感数据放在`gradle.properties`中，并在版本控制中忽略掉


## 命名规范


### 资源命名
#### 布局文件
| **类型**      | **前缀**             |
| ----------- | ------------------ |
| Activity    | activity_          |
| Fragment    | fragment_          |
| Dialog      | dialog_            |
| PopupWindow | popup_             |
| Menu        | menu_              |
| Adapter     | layout_item_/item_ |

#### 图片资源
| **前缀**    | **类型**      |
| --------- | ----------- |
| bg_xxx    | 背景图片        |
| btn_xxx   | Button      |
| ic_xxx    | Icon        |
| bg_描述_状态  | 不同状态的背景     |
| btn_描述_状态 | 不同状态的Button |
| chx_描述_状态 | 选择框         |

#### 其他资源
| **类型**  | **命名**        | **示例**            |
| ------- | ------------- | ----------------- |
| strings | 模块名_逻辑名称      | main_menu_about   |
| colors  | 模块名_颜色名称      | main_info_bg_gray |
| style   | 模块名_逻辑名称（驼峰法） | main_tabBottom    |
| dimen   | 模块名_逻辑名称      | main_menu_padding |


### 布局使用
- 合理布局，注意布局嵌套层次，有效运用`<merge>`、`<ViewStub>`、`<include>`标签




### 注释规范

#### 类注释

每个类完成后应该有作者姓名和联系方式的注释，对自己的代码负责

> 具体的注释风格可以在AS中自己配制:
>
> Settings → Editor → File and Code Templates → Includes → File Header

```java
/**
 * <pre>
 *     author : developer
 *     e-mail : xxx@xx
 *     time   : 2017/08/13
 *     desc   : xxxx描述
 *     version: 1.0
 * </pre>
 */
public class MainActivity {
       ...
}
```



### 代码规范



**使用单例可以减轻加载的负担、缩短加载的时间、提高加载的效率**，但并不是所有地方都适用于单例，简单来说，单例主要适用于一下三个方面：
1. 控制资源的使用，通过线程同步来控制资源的并发访问
2. 控制实例的产生，以达到节约资源的目的
3. 控制数据的共享，在不建立关联的条件下，让多个不相关的进程或线程之间实现通信



把一个基本数据类型转为字符串，基础数据类型`.toString()`最快，·String.valueOf(数据)·其次，`数据 + ""`最慢。



**循环内不要不断创建对象**

例如：

```
for (int i = 0; i <= count; i++) {
  Object obj = new Object();
}
```

如果count的值很大，消耗的内存就会更大，建议改为：

```
Object obj = null;
for (int i = 0; i <= count; i++) {
	obj = new Object();
}
```

这样的话，内存中只有一份Object对象引用，每次new Object()的时候，Object对象引用指向不同的Object罢了，但是内存中只有一份，这样就大大节省了内存空间。



**for 和 foreach的使用场景**

实现RandomAccess接口的类实例比如ArrayList，假如是随机访问的，使用普通`for`循环效率将高于使用`foreach`，反过来如果是顺序访问的，则使用Iterator会效率更高（`foreach`的底层实现就是迭代器）。



**ArrayList 和 LinkedList的使用场景**

顺序插入和随机访问比较多的场景使用ArrayList，元素删除和中间插入比较多的场景使用LinkedList，

详细可查看ArrayList和LinkedList的原理。



**不要让public方法中有太多的形参**

如果给这些方法太多形参的话主要有两点坏处：

1. 违反了面向对象的编程思想 ，Java讲求一切都是对象，太多的形参，和面向对象的变成思想并不契合。
2. 参数太多势必导致方法调用的出错概率增加。



字符串变量和字符串常量equals的时候将字符串常量写在前面

```java
String str = "123";
if ("123".equals(str)) {
  ...
}
```

这么做主要是可以避免空指针异常



**公用集合类不使用的数据一定要及时`remove`掉**

如果一个集合类是公用的（也就是说不是方法里面的属性），那么这个集合里面的元素是不会自动释放的，因为始终有引用指向它们。所以，如果公用集合里面的某些数据不使用而不去remove掉它们，那么将会造成这个公用集合不断增大，使得系统有内存泄露的隐患。



**使用最有效率的方式去遍历Map**

```java
public static void main(String[] args) {
  HashMap<String, String> hm = new HashMap<String, String>();
  hm.put("111", "222");
  
  Set<Map.Entry<String, String>> entrySet = hm.entrySet();
  Iterator<Map.Entry<String, String>> iter = entrySet.iterator(); 
  
  while (iter.hasNext()) {
    Map.Entry<String, String> entry = iter.next();
    System.out.println(entry.getKey() + "\t" + entry.getValue());
  }
}
```

如果只是想遍历下这个Map的key值，那用`Set<String> keySet = hm.keySet()`会比较合适。



**尽量采用懒加载的策略，即在需要的时候才创建的原则**

**尽量避免随意或过度的使用静态变量**