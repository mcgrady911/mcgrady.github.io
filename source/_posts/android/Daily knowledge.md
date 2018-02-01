# Daily knowledge

1. 什么是类？什么是对象？类和对象之间的关系？

   > 类是对象的集合，是抽象的；对象是类的实例化，是具体的。

2. 简述C/C++的核心思想

   > http://www.cnblogs.com/lanxuezaipiao/p/4135105.html

3. 泛型的本质

   > 泛型的本质是参数化类型，也就是说所操作的数据类型被指定为某个参数类型。
   >
   > 这种参数类型可以在类、接口和方法中创建，分别称为泛型类、泛型接口、泛型方法。
   >
   > 优点：在编译的时候检查类型安全，并且素有的强制转换都是自动和隐式的，提高代码的重用率。
   >
   > 缺点：

4. 静态变量的使用场景

   > 1. 变量所包含的对象提及较大，占用内存较多。
   > 2. 变量所包含的对象生名周期较长。
   > 3. 变量所包含的对象数据稳定。
   > 4. 该类的对象实例有对该变量所包含的对象共享需求。

5. 类加载原理及类加载器

6. 对Clone的理解

7. HashMap的实现

8. Collection和Collections的区别

9. 数组浅析

10. 代码优化编程

11. 事件处理机制与"恋爱关系"

12. JNDI(Java命名与目录接口)

13. Comparable和Comparator实现对象比较

14. String、StringBuffer与StringBuilder之间的区别

   > String：字符串常量
   >
   > StringBuffer：字符串变量，线程安全
   >
   > StringBuilder：字符串变量，线程非安全
   >
   > 三者在执行速度方面的比较：StringBuilder > StringBuffer > String
   >
   > **总结**：
   >
   > 1. 操作少量的数据用 String
   > 2. 单线程操作字符串缓冲区下操作大量数据 StringBuilder
   > 3. 多线程操作字符串缓冲区下操作大量数据 StringBuffer

15. Heap和Stack的区别

16. 反射机制

---
1.  Synchronized的使用

    > synchronized代码块：
    >
    > synchronized方法：
    >
    > synchronized静态方法：
    >
    > synchronized类：
    >
    > synchronized()：
2.  android:background="?/attr/..."
3.  DrawerLayout
4.  @SuppressLint("...")
5.  @SuppressWarnings("deprecation")
6.  @TargetApi(Build.VERSION_CODES.HONEYCOMB)
7.  CharSequence
8.  attr
9.  dimen
10.  @Nullable
11.  <merge>
12.  android:shadowDx
13.  android:shadowDy
14.  android:shadowRadius
15.  layout_alignWithParentIfMissing

