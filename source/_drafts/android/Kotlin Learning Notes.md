#  Kotlin Learning Notes



##  变量、函数和类型

### 1. 变量

#### 1-1. Kotlin 的空安全设计

1. 在Kotlin里面，所有的变量默认不允许为空。

2. 对于一些可空的变量，可以在类型右边加上`?`号解除非空限制：`var name: String? = null`；在Kotlin中这种类型称作**可空类型**。

3. 对于空引用的调用会导致空指针异常，所以Kotlin在可空变量直接调用时IDE会报错，即使做了非空判断也不能保证调用的时候就是非空，因为多线程情况下，其它线程可能变更为空，例如：

   ```kotlin
   var view: View? = null
   view.setBackgroundColor(Color.RED)
   //Error: Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type View?
   
   if (view != null) {
     view.setBackgroundColor(Color.RED)
     //Error: Smart cast to 'View' is impossible, because 'view' is a mutable property that could have been changed by this time
   }
   ```

4. `non-null asserted call`断言式调用，意思是告诉IDE，这里的变量一定是非空，不需要编译器做检查，一旦启用非空断言，享受不到Kotlin的控安全设计的好处（编译时检查，而不是运行时抛异常），实际上和Java就没什么两样。

   ```kotlin
   view!!.setBackgroundColor(Color.RED)
   ```



#### 1-2. 延迟初始化

```kotlin
lateinit var view: View
```

`lateinit`：意思是告诉编译器变量没法第一时间初始化，作用就是让IDE不要对这个变量检查初始化和报错。

当编译器不检查初始化值，在未初始化调用该变量也会报错：

```kotlin
lateint var view: View
view.setBackgroundColor(Color.RED)
//Error UninitializedPropertyAccessException: lateinit property view has not been initialized
```



#### 1-3. val 和 var

> var 是 variable 的缩写，val 是 value的缩写

`val` 是Kotlin在Java的「变量」之外增加的一种变量类型：**只读变量**；它只能赋值一次，不能修改。而 `var` 是一种**可读可写变量**。

> 那有没有方法修改 `val` 类型的变量呢？答案是**有**！见：[2-3. getter/setter 函数](#2-3)



### 2. 函数

> 对于任何编程语言来讲，变量是用来存储数据的，而函数是用来处理数据的。

#### 2-1. 函数的声明

函数参数也可以有可空的控制，根据前面说的空安全设计，在传递时需要注意：

```kotlin
var name: String? = "mcgrady"
fun main(args: Array<String>) {
  
  //可空变量传给不可空参数时，编译报错
  //Error: Type mismatch: inferred type is String? but String was expected
  call(name)
}

fun call(name: String) {
  println("name is $name")
}
```



#### 2-2. 可见性

#### <a id="2-3">2-3. getter/setter 函数</a>

Kotlin的getter/setter和Java的一些区别：

- getter/setter 函数有专门的关键字`get`和`set`
- gtter/setter 函数位于var所声明的变量下面
- setter 的函数参数是value

除此之外还多了个叫`field`的东西，这个东西叫做「**Backing Field**」

```kotlin
class Kotlin {
  var name = "kotlin"
}
```

在编译后的字节码大致等价于这样的Java代码

```java
public final class Kotlin {
  @NotNull
  private String name = "kotlin";
  
  @NotNull
  public final String getName() {
    return this.name;
  }
  
  public final void setName(@NotNull String name) {
    this.name = name;
  }
}
```

  上面的`String name`就是Kotlin自动创建的一个`Java field`。这个`field`对编码的人不可见，但会自动应用于getter/setter，因此命名为「Backing Field」。

  对于Kotlin的语法来讲，Kotlin的`field`和Java的`field`不同一个概念，在Kotlin里，它相当于每个`var`内部的一个变量。

  前面讲过`val`是只读变量，也就是说不能调用setter函数，因此，`val`声明的变量不能重写setter函数的，但它可以重写getter函数：

```kotlin
val name = "mcgrady"
get() {
  return field + "NB"
}
```

所以，`val`所声明的只读变量，在取值的时候仍然可能被修改，这也是和Java里的`final`不同之处。

> 关于「钩子」的作用，出了修改取值和复制，也可以加一些自己的逻辑，就像我们在Activity的声明周期函数里做的事情一样。



#### 2-4. 函数简化

```kotlin
fun area(width: Int, height: Int): Int {
  retrun width * height
}
//省略了 {} 和 return，使用 = 符号连接返回值
fun area(width: Int, height: Int): Int = width * height
//通过「类型推断」特性，省略了返回值类型
fun area(width: Int, height: Int) = width * height

fun sayHi(name: String) {
  println("Hi " + name)
}
//函数没有返回值的情况下，可以理解为返回值是 Unit。可以简化成如下
fun sayHi(name: String) = println("Hi " + name)
//当调用函数不传参时，可以使用参数默认值
fun sayHi(name: String = "world") = println("Hi " + name)

fun clockIn(name: String = "mcgrady", time: Int) {
  ...
}
clockIn(20200524)
// 这时想使用默认值进行调用，IDE 会报以下两个错误
// Error: The integer literal does not conform to the expected type String
// Error: No value passed for parameter 'time'

//需要通过「命名参数」解决这个问题
clockIn(time = 20200524)
```

#### 2-5. 本地函数（嵌套函数）

```kotlin
fun login(user: String, password: String, illegalStr: String) {
  //验证 user 是否为空
  if (user.isEmpty()) {
    throw IllegalArgumentException(illegalStr)
  }
  
  if (password.isEmpty()) {
    throw IllegalArgumentException(illegalStr)
  }
}

//嵌套函数方式改进
fun login(user: String, password: String, illegalStr: String) {
  fun validate(value: String) {
    if (value.isEmpty()) {
      throw IllegalArgumentException(illegalStr)
    }
  }
  
  validate(user, illegalStr)
  validate(password, illegalStr)
}

//lambda表达式配合Kotlin内置 require 函数改进
fun login(user: String, password: String) {
  require(user.isNotEmpty()) {illegalStr}
  require(password.isNotEmpty()) {illegalStr}
}
```



#### 2-6. 扩展函数





### 3. 类型

#### 3-1. 基本类型

- Kotlin里的`int`和Java里的`int`、`Integer`不同，主要是在装箱方面不同：

  - Java里的`int`是unbox的，而`Integer`是box的

  - Kotlin里，`int`是否装箱根据场合来定

    ```kotlin
    var a: Int = 1	//unbox
    var b: Int? = 2	//box
    var list: List<Int> = listOf(1,2)	//box
    ```

- 对比Java里的基本类型，Kotlin里满足以下条件之一就不装箱：

  - 不可空类型
  - 使用`IntArray`、`FloatArray`等

#### 3-2. 字符串

- 字符串拼接

  ```kotlin
  val name = "world"
  println("Hi " + name)
  //等价于
  println("Hi $name")
  //除了变量，$后还可以跟表达式，需要用 {} 包裹
  println("the name is $name, length = ${name.length}")
  ```

- raw string（原生字符串）

  ```kotlin
  val name = "world"
  val myName = "kotlin"
  
  val text = """
  Hi $name!
  My name is $myName.
  """
  println(text)
  
  //输出结果
  Hi world!
  My name is kotlin.
  ```

- 字符串模板

  ```kotlin
  val strings = arrayListOf("abc", "def", "ghi")
  println("strings is $strings")
  println("strings is ${strings[0]}")
  println("strings is ${ if (string.size > 0) strings[0] else "null"}")
  ```

  

#### 3-3. 类和对象

##### 3-3-1. 类的可见性

Kotlin的类默认是``public`的，所以Java中的`public`在Kotlin中可以省略。

##### 3-3-2. 类的继承

继承的写法，Java里用`extends`，而Kotlin里用`:`，但`:`不仅可以表示继承，还可以表示Java中的`implement`。

Kotlin里的类默认是`final`的，可以通过`open`来解除`final`限制

```kotlin
open class MainActivity : AppCompatActivity() {
}
class NewActivity : MainActivity() {
}
```

##### 3-3-3. 构造方法

```kotlin
/******************** 默认的构造函数 ********************/
class MainActivity : AppCompatActivity() {
}
//等价于
class MainActivity constructor() : AppCompatActivity() {
}

/******************** 主构造器 ********************/
class User {
  val id: Int
  val name: String
  
  constructor(id: Int, name: String) {
    this.id = id
    this.name = name
  }
}
//等价于
class User constructor(id: Int, name: String) {
  var id: id = id
  var name: String = name
}
//通常，主构造函数中的 constructor 关键字可以省略
class User(name: String) {
  var name: String = name
}

/******************** 次构造器 ********************/
class User constructor(var name: String) {
  constructor(name: String, id: Int) : this(name) {   
  }
  
  constructor(name: String, id: Int, age: Int) : this(name, id) {
  }
}
```

- Kotlin把构造器单独用`constructor`关键字和其它`fun`做区分。

- Java中构造器和类同名，Kotlin中使用`constructor`表示。

- 我们知道主构造器 `constructor` 关键字通常情况下可以省略，但某些场景是不可省略的，例如在主构造器上使用「可见性修饰符」或「注解」

  ```kotlin
  class User private constructor(name: String) {
    //主构造器被修饰为私有的，外部无法调用该构造器
  }
  ```

##### 3-3-4. override函数

- Java里@`Override`是注解的形式

- Kotlin里`override`是关键字的形式

- Kotlin省略了`protected`关键字，也就是说，**Kotlin里`override`函数的可见性是继承自父类的**。

- 若要关闭`override`的遗传性，可以通过`final`关闭

  ```kotlin
  open class MainActivity : AppCompatActivity() {
    final override fun onCreate(savedInstanceState: Bundle?) {
    }
  }
  ```

##### 3-3-5. init代码块

- Java

```java
public class User {
  {
    //初始化代码块，先于下面的构造器执行
  }
  
  public User() {
  }
}
```

- Kotlin

```kotlin
class User {
  init{
    //初始化代码块，先于下面的构造器执行
  }
  
  constructor() {
    
  }
}
```

##### 3-3-6. object

Kotlin里的`object`首字母是小写，做为关键字使用，区别于Java里`Object`，Java里的`Object`在Kotlin中变成了`Any`，作为所有类的基础；而Kotlin里的`object`的作用是：**创建一个类，并且创建一个类的对象**。

```kotlin
object Sample {
  val name = "mcgrady"
}

//通过类名可直接访问，类似Java中的单例
Sample.name
```

所以，在Kotlin中创建单例不用像Java中那么复杂，只需要把`class`换成`object`。

> 1. 这种通过`object`实现的单例是一个饿汉式的单例，并且实现了线程安全。
> 2. 用`object`修饰的对象汇总的变量和函数都是静态的。

##### 3-3-7. 匿名类

- Java

  ```java
  View.OnClickListener listener = new View.OnClickListener() {
    @Override
    public void onClick(View v) {
  
    }
  };
  ```

- Kotlin

  ```kotlin
  val listener = object : View.OnClickListener {
    override fun onClick(v: View?) {
  
    }
  }
  ```

对比Java，Kotlin只不过把`new`换成了`object`，这里的`new`和`object`修饰的都是接口和抽象类。

##### 3-3-8. companion object 伴生对象

表示修饰的对象和外部类绑定；等价于Java静态变量和方法的写法。

作用：用 `object` 修饰的对象中的变量和函数都是静态的，但有时候，我们只想让类中的一部分函数和变量是静态，这时可以用半生对象实现：

```kotlin
class A {
  companion object {
    var c: Int = 0
  }
}

//调用
A.c
```



##### 3-3-9. object、compantion object、top-level 对比

- 工具类功能，直接创建文件，写`top-level`「顶层」函数。
- 继承类或实现接口，用`object`或`compantion object`。



#### 3-4. 类型的判断和强转

- 类型判断，Java中用`instanceof`，Kotlin中用`is`

  ```java
  void main() {
    Activity activity = new MainActivity();
    if (activity instanceof MainActivity) {
      ((MainActivity) activity).action();
    }
  }
  ```

  ```kotlin
  fun main() {
    //Kotlin中实例化省略了new关键字
    var activity: Activity = MainActivity()
    //1. Kotlin中的强转由于类型推断被省略了
    if(activity is MainActivity) {
      activity.action()
    }
    
    //2. 如果强转类型安全，想忽略类型判断的可以使用以下写法
    (activity as MainActivity).action()
    
    //3. 如果如果强转类型非安全，想忽略类型判断的可以使用以下写法
    (activity as? MainActivity)?.action()
  }
  ```



## 常量

```kotlin
class Sample1 {
  companion object {
    const val CONST_NUMBER = 1
  }
}

object Sample2 {
  const val CONST_NUMBER = 1
}

const val CONST_SECOND_NUMBER = 1

```

- Kotlin的常量必须声明在`object`、`compantion object`、`top-level`中，因为常量是静态的。

- Kotlin新增了修饰常量的`const`关键字。

- Kotlin中只有基本类型和`String`类型可以声明成常量。
  原因是Kotlin中的常量是指「compile-time constant 编译时常量」，意思是在编译期间就知道这个东西在每处调用的实际值，因此可以在编译期间直接把这个值硬编码到调用的地方。

  而其它的类型变量，可以通过调用对象的方法或变量改变对象内部的值，这样内部发生变化后就不是常量了。



## 数组和集合

### 1. 数组

声明一个数组

- Java中的写法

  ```java
  String[] strs = {"a", "b", "c"};
  ```

- Kotlin中的写法

  ```kotlin
  val strs: Array<String> = arrayOf("a", "b", "c")
  ```



### 2. 集合

Kotlin和Java一样有三种集合类型：`List`、`Set`、`Map`

- `List`：以固定的顺序村粗一组元素，元素可以重复

  - Java中创建一个列表

    ```java
    List<String> strList = new ArrayList<>();
    strList.add("a");
    strList.add("b");
    strList.add("c");
    ```

  - Kotlin中创建一个列表

    ```kotlin
    val strList = listOf("a", "b", "c")
    ```

  可以看到Kotlin中创建一个`List`特别简单，而且Kotlin中的`List`多了一个特性：支持`covariant（协变）`，可以把子类的`List`赋值给父类的List变量：

  ```kotlin
  val strList: List<String> listOf("a", "b", "c")
  val anys: List<String> = strList
  ```

  

- `Set`：存储一组互不相等的元素，通常没有固定顺序

  - Java中创建一个`Set`

    ```java
    Set<String> strSet = new HashSet<>();
    strSet.add("a");
    strSet.add("b");
    strSet.add("c");
    ```

  - Kotlin中创建一个`Set`

    ```kotlin
    val strSet = setOf("a", "b", "c")
    ```

  类是`List`，`Set`同样具有`covariant（协变）`特性。
  
- `Map`：存储键值对的数据集合，键互不相等，但不同的键可以对应相同的值

  - Java中创建一个`Map`

    ```java
    Map<String, Integer> map = new HashMap<>();
    map.put("key1", 1);
    map.put("key2", 2);
    map.put("key3", 3);
    ```

  - Kotlin中创建一个`Map`

    ```kotlin
    val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3)
    ```

    `to`表示「键」和「值」关联，叫做「中辍表达式」。

    

  - Kotlin中`Map`的取值和修改

    ```kotlin
    val map = mutableMapOf("key1" to 1, "key2" to 2)
    
    val value1 = map.get("key1")
    val value2 = map["key2"]
    
    map.put("key1", 1)
    map["key2"] = 1
    可变集合/不可变集合
    ```

- 可变集合/不可变集合
  上面修改`Map`值得例子中，创建函数的是`mutableMapOf()`而不是`mapOf()`，因为Kotlin中分为两种类型：只读和可变，只有`mutableMapOf()`创建的`Map`才能修改。

  - 这里的**只读**有两层意思: 集合的`size`不可变，集合中的元素值不可变
  - 已创建的只读集合，可以通过`toMutable*()`转换可变集合，这里返回的是一个新建的集合，原有集合保持只读属性。

- Sequence 
  Kotlin引入了一个新的容器类型`Sequence`，它和`Iterable`一样用来遍历一组数据并可以对每个元素进行特定的处理。

  ```kotlin
  //创建一组元素，类似listOf()
  sequenceOf("a", "b", "c")
  
  //使用Iterable创建一组元素，这里的list实现了Iterable接口
  val list = listOf("a", "b", "c")
  list.asSequence()
  
  //使用lambda表达式创建一组元素
  val sequence = generateSequence(0) {it + 1}
  ```
  
  序列 Sequence 又被称作「惰性集合操作」，在定义执行流程阶段不会被执行，只有使用的时候才执行
  
  ```kotlin
  val sequence = sequenceOf(1,2,3,4)
  val result: Sequence<Int> = sequence.map { i ->
  	println("Map $i")
  	i * 2
  }.filter { i ->
  		println("Filter $i")
      i % 3 == 0
  }
  
  println(result.first())	//被执行的地方
  ```
  
  `Sequence` 的优点：
  
  - 一旦满足遍历退出的条件，可省略后续不必要的遍历过程
  - 像`List` 这种实现 Iterable 接口的集合，每调用一次函数就会创建一个新的Iterable，若下一个函数再局域新的 Iterable 执行，每次函数调用产生临时的 Iterable 会导致额外的内存消耗，而 Sequence 在整个流程中只有一个。
  
  因此，`Sequence` 这种数据类型可以在数据量较大或者量未知的时候，作为流式处理的解决方案。

### 3. 数组和集合的操作符

```kotlin
val intArrayOf(1, 2, 3)
val strList = listOf("a", "b", "c")
```

#### 3-1. forEach 遍历操作

```kotlin
intArray.forEach { i -> 
  print(i + " ")
}
```

#### 3-2. filter 过滤操作

```kotlin
val newList: List = intArray.filter { i ->
	i != i 	// 过滤数组中等于 1 的元素
}
```

#### 3-3. map 遍历并执行指定表达式操作

```kotlin
val newList: List = intArray.map { i ->
	i + 1
}
```

#### 3-4. flatMap 遍历并为每个元素创建新的集合，最后合并到一个集合中

```kotlin
intArray.flatMap { i ->
	listOf("${i + 1}", "a")  //生成新的集合
}
```



### 4. Range 区间类型

```kotlin
val range: IntRange = 0..100	//表示0到100范围，包括100 [0, 100]

val range: IntRange = 0 until 100 //表示0到100范围，不包括100 [0, 100)
```

- `in` 关键字

  ```kotlin
  val range = 0..100
  for (i in range) {
    print("$i, ")
  }
  ```

- `step` 关键字

  ```kotlin
  val range = 0..100
  for (i in range step 2) {
    print("$i, ")
  }
  ```

- `downTo` 关键字

  ```kotlin
  for (i in 4 downTo 1) {
    print("$i, ") 	//输出：4，3，2，1
  }
  ```

  

## 可见性修饰符

Kotlin中有四种可见性修饰符：

- `public`：公开，可见性最大
- `private`：私有，可见性最小，根据声明位置不同可分为**类中可见**和**文件中可见**
- `protected`：保护，相当于`private` + 子类可见
- `internal`：内部，仅对module内可见

相比Java少了一个`default`「包内可见」修饰符，多了一个`internal`「module内可见」修饰符。

### public

Java中默认修饰符是`default`，表示包内可见，只有在同一个package内可引用，package外引用，需要在class前加上`public`修饰符。

Kotlin中默认修饰符是`public`。

### internal

`internal`表示修饰的类、函数仅对module内可见，这里的module具体指的是一组共同编译的kotlin文件，常见的形式有：

- Android Studio 里的 module
- Maven project

>鉴于Android sdk官方sdk中，只对sdk内可见的Javadoc的`@hide`注释，这种特性的不严格性，可以通过反射访问到限制的方法，Kotlin引进了一个更为严格的可见性修饰符：`internal`。
>
>Java中的`default`「包内可见」在Kotlin中被弃用，与它更接近的是`internal`「module内可见」。
>
>- Kotlin鼓励创建 top-level 函数和属性，一个源文件可以包含多个类，使得Kotlin的源码结构更加扁平化，包结构不再像Java中那么重要。
>- 为了代码的解耦和可维护性，module越来越多、越来越小，使得`internal`「module内可见」已经可以满足代码封装的需求。

### protected

- Java中`protected`表示包内可见 + 子类可见
- Kotlin中`protected`表示`private ` + 子类可见

相比Java的可进行着眼于package，Kotlin更关心的是module。

### private

- Java中`private`表示类中可见，作为内部类时对外部类「可见」。
- Kotlin中的 `private` 表示类中或所在文件内可见，作为内部类时对外部类「不可见」。



## 条件控制

### 1. if/else

```kotlin
val max = if (a > b) a else b
//含代码块逻辑的方式
val max = if (a > b) {
  println("max: a")
  a
} else {
  println("max: b")
  b
}
```

### 2. when

```kotlin
when(x) {
  1 -> {println("1")}
  2 -> {println("2")}
  
  else -> {println("else")}
}

//多种情况执行同一份代码时
when(x) {
  1, 2 -> println("x == 1 or x == 2")
  else -> println("else")
}

//使用 in 检查是否在一个区间或者集合中
when(x) {
  in 1..10 -> println("x 在区间 1..10 中")
  int listOf(1,2) -> println("x 在集合中")
  !in 10..20 -> println("x 不在区间 10..20 中")
  else -> println("不在任何区间上")
}

//使用 is 进行特定类型检查
val isString = when(x) {
  is String -> true
  else -> false
}

//当分支的判断条件为表达式时，优先执行为 true 分支代码块，还可以省略 when后面的参数
when {
  str1.contains("a") -> println("字符串 str1 包含 a")
  str2.length == 3 -> println("字符串 str2 的长度为 3")
}
```

- 相对与Java的 `switch` ，省略了 `case` 和 `break` 
- 相对与Java的 `default` ，Kotlin中使用 `else`

### 3. for

```kotlin
val array = intArrayOf(1, 2, 3, 4)
for (item in array) {
  ...
}

for(item in 0..10) {
  println(i)
}
```



### 4. try-catch

```kotlin
val a: Int? = try { paseInt(input)} catch(e: NumberFormatException) { null }
```



### 5. `.`  和 `?:`

空安全调用 `?.` ，在对象非空时会执行后面的调用，对象为空时返回 null，如果这是讲该表达式复制给一个不可为空的变量：

```kotlin
val str: String? = "Hello"
val length: Int = str?.length
//Error: Type mismatch. Required:Int. Found:Int?
```

报错的原因是 `str` 为 null 时没有值可以返回给 `length` ，这时可以使用Kotlin的 Elvis 操作符 `?:` 解决 ：

```kotlin
val str: String? = "Hello"
val length: Int = str?.length ?: -1

fun validate(user: User) {
  if (user.id == null) {
    return
  }
  val id = user.id
}
// Elvis 操作符进行优化
fun validate(user: User) {
  val id = user.id ?: return
}
```



### 6. `==` 和 `===`

- ==：对基本数据类型以及 `String` 等类型进行内容比较，相当于Java中的 `equals`
- ===：对引用的内存地址进行比较，相当于Java中的 ==



## 泛型

把具体的类型泛化，编码的时候用符号来指代类型，在使用的时候再确定它的类型。

### `in` 和 `out`

```kotlin
//生产者
class Producer<T> {
  fun produce(): T {
    ...
  }
}
val producer: Producer<out TextView> = Producer<Button>()
val textView: TextView = producer.produce()	//相当于 List.get

//消费者
class Consumer<T> {
  fun consume(t: T) {
    ...
  }
}

val consumer: Consumer<in Button> = Consumer<TextView>()
consumer.consume(Button(context))	//相当于 List.add
```

- 使用关键字 `out` 来支持协变，等同于Java中的商界通配符 `? extends`
- 使用关键字 `in` 来支持逆变，等同于Java中的下界通配符 `? super`

### `*` 号

Java中单个 `?` 号也能作为泛型通配符，相当于 `? extends Object` ，在Kotlin中等效的写法：`*` 号，相当于 `out Any`。

```kotlin
var list: List<*>
```

### `where` 关键字

设置多个边界可以使用 `where` 关键字

```kotlin
class Monster<T> where T: Animal, T: Food {
  
}

//当Monster本身还有继承的时候
class Monster<T> : MonsterParent<T> where T: Animal {
  
}
```

### `reified` 关键字

由于Java中的泛型存在类型擦除的情况，任何在运行时需要知道泛型确切类型信息的操作都没法用了。

比如不能检查一个对象是否为泛型类型 `T` 的实例：

```java
<T> void printIfTypeMatch(Object item) {
  if (item instanceOf T) {
    System.out.println(item);
  }
}
//Error: illegal generic type for instanceof
```

这个问题在Java中通常是额外传递一个 `Class<T>` 类型的参数，然后通过 `Class#isInstance` 方法来检查：

```java
<T> void printIfTypeMatch(Object item, CLass<T> type) {
  if (type.isInstance(item)) {
    System.out.println(item);
  }
}
```

Kotlin中还有一个更方便的做法：使用关键字 `reified` 配合 `inline` 解决：

```kotlin
inline fun <reified T> printIfTypeMatch(item: Any) {
  if (item is T) {
    println(item)
  }
}
```



## Higher-Order Functions（高阶函数）

在Kotlin里可以将函数作为函数的参数进行传递使用：

```kotlin
fun a(funParam: (Int) -> (String)): String {
  return funParam(1)
}

fun b(param: Int): String {
  return param.toString()
}

//调用
a(::b)
val d = ::b
```

### `::method` 语法糖

在Kotlin中，一个函数名左边加上双冒号，表示是一个**函数类型的对象**（或者说是一个指向对象的引用），但这个对象不是函数本身，而是一个和这个函数有相同功能的对象。这种写法也叫「Function Reference」函数引用。

实际上对一个函数类型的对象加括号、参数，他真正调用的是这个对象的 `invoke()` 函数：

```kotlin
fun b(param: Int): String {
  return param.toString()
}

b(1)	//实际上调用 b.invoke(1)
(::b)(1)	//实际上调用 (::b).invoke(1)
```

### 匿名函数

要传一个函数类型的参数或把一个函数类型的对象赋值给变量，出了用 `::method`语法糖实现，还可以用匿名函数实现：

```kotlin
fun a(fun(param: Int): String {
  return param.toString()
})

val d = fun(param: Int): String {
  return param.toString()
}
```

下面通过 `View.setOnClickListener` 展示Kotlin匿名函数的用法：

```java
public interface OnClickListener {
  void onClick(View v);
}

public void setOnClickListener(OnClickListener listener) {
  this.listener = listener;
}

view.setOnClickListener(new OnClickListener() {
  @Override
  void onClick(View v) {
    ...
  }
});
```

```kotlin
fun setOnClickListener(onClick: (View) -> Unit) {
  this.onClick = onClick
}

view.setOnClickListener(fun(v: View): Unit) {
  ...
}
//通过Lambda表达式简化
view.setOnClickListener() { v: View ->
  ...
}
//如果Lambda是函数唯一的参数时
view.setOnClickListener { v: View ->
	...
}
//如果Lambda是单参数时，参数也可以忽略不写
view.setOnClickListener {
  ...
  //Kotlin的Lambda对省略的唯一参数有默认的名字：it
  it.setVisibility(GONE)
}
```

当把一个匿名函数赋值给变量时：

```kotlin
val b = fun(param: Int): String {
  return param.toString()
}
//通过Lambda表达式简化
val b = { param: Int ->
	return param.toString()
}
//省略参数类型和返回的写法
val b: (Int) -> String = {
  it.toString()
}
```

### 匿名函数和Lambda表达式的本质

首先，Kotlin的匿名函数不是通常的函数，而是一个函数类型的对象，可以把它当做函数的参数来传递以及赋值给变量；同理，Lambda 其实也是一个函数类型的对象。



### 对比 Java 的 Lambda

Kotlin的 Lambda 跟 Java 8 的 Lambda 不一样，Java 8的只是一种便捷写法，本质上没有功能上的突破，而Kotlin的是一个实际的函数类型对象。

当和 Java 交互的时候，函数参数是 Java 的单抽象方法的接口，依然可以使用 Lambda 来写参数。这是 Kotlin对于来自 Java 的单抽象方法的接口为它们额外创建一个把参数替换为函数类型的桥接方法，让你可以间接地创建 Java 的匿名类对象。

这就是为什么？当你在 Kotlin 里调用 View.java 这个类的 setOnClickListener() 的时候，可以传 Lambda 给它来创建 OnClickListener 对象，但你照着同样的写法写一个 Kotlin 的接口，你却不能传 Lambda。因为 Kotlin 期望我们直接使用函数类型的参数，而不是用接口这种折中方案。

### 总结

- Kotlin里有一类Java中不存在的类型「函数类型」，这类类型的对象可以当函数来调用，还能作为函数的参数、函数的返回值以及赋值给变量。
- 创建一个函数类型对象的方式有三种：
  - `::method`语法糖
  - 匿名函数
  - Lambda表达式



## Coroutines 协程

> 协程是一种编程思想，不局限于特定语言。

Kotlin官方称「本质上，协程是轻量级的线程」，关于协程和线程的关系，我们可以从Android开发者的角度理解它们的关系：

- 我们所有的代码都是跑在线程中的，而线程是跑在进程中的。
- 协程没有直接和操作系统关联，它也是跑在线程中的，可以是单线程，也可以是多线程。
- 单线程中的协程总的执行事件并不会比不用协程少。
- Android中，在与线程进行网络请求，会抛出 `NetworkOnMainThreadException` 异常，协程也不例外，所以这种场景还是要切换线程。
- 协程设计的初衷是**为了解决并发问题**，让「协作式多任务」实现起来更方便。
- 协程是Kotlin内部提供的一套线程封装的API，让我们在写代码时，不用关注多线程就能方便的写出并发操作。

 

协程的一个典型的使用场景就是**线程控制**，就像Java中的 `Executor` 和 Android中的 `AsyncTask`

```java
//Java中实现并发操作
new Thread(new Runnable() {
  @Override
  public void run() {
    ...
  }
}).start();
```

```kotlin
//Kotlin中通过线程方式实现并发操作
Thread({
	...
}).start()
```

可以看到这里只是开启了一个新线程，至于它何时结束、执行结果，我们在主线程是无法直接知道的。这里使用 `Thread` 摆脱不了这些困难和不方便：

- 线程何时执行结束

- 线程间通信

  ```kotlin
  //对于以上问题，可以通过 Android的 AsyncTask 处理线程间通信
  object : AsyncTask<T0, T1, T2> {
    override fun doInBackground(vararg args: T0): String { ... }
    override fun onProgressUpdate(vararg args: T1) { ... }
    override fun onPostExecute(t3: T3) { ... }
  }
  ```

- 多线程管理

  ```kotlin
  //对于以上问题，可以通过 Java的 Executor 线程池或Android的 进行管理：
  val executor = Executors.newCachedThreadPool()
    executor.execute({
      ...
    })
  ```

下面看下协程进行网络请求获取用户信息并显示到UI上，是怎么处理以上这些问题的：

```kotlin
launch({
  val user = api.getUser()	//网络请求（IO线程）
  tvName.text = user.name		//更新UI（主线程）
})
```

-  `launch` 并不是一个顶层函数，它必须在一个对象中使用
- `launch`函数加上实现在 `{}` 中的具体逻辑就构成了一个协程
- `api.getUser` 是一个**挂起函数**，所以能够保证 `tvName.text` 的正确赋值，这里涉及到了协程中最著名的**「非阻塞式挂起」**概念。

### 闭包

以 `Thread` 为例，来看看什么是闭包：

```kotlin
Thread(object: Runnable {
  override fun run() {
    ...
  }
})

//满足 SAM, 简化为
Thread({
  ...
})

//使用闭包，再简化为
Thread {
  ...
}
```

形如 `Thread {...}` 这样结构中的 `{}` 就是一个闭包。

在Kotlin中有这样一个语法糖：当函数的最后一个参数是 Lambda 表达式时，可以将 Lambda 写在括号外，这就是它的闭包原则。

对于上文使用的 `launch` 函数，可以通过闭包简化：

```kotlin
launch {
  ...
}
```



>SAM——Single Abstract Method ，简称单抽象方法的接口



### 基本使用

前面提到，`launch` 不是顶层函数，是不能直接使用的，可以使用下面三种方法来创建协程：

```kotlin
//方式一 使用 runBlocking 顶层函数
runBlocking {
  getImage(imageId)
}

//方式二 使用 GlobalScope 单例对象
GlobalScope。launch {
  getImage(imageId)
}

//方式三 自行通过 CoroutineContext 创建一个 CoroutineScope 对象
val coroutineScope = CoroutineScope(context)
coroutineScope.launch {
  getImage(imageId)
}
```

- 方式一通常适用于单元测试的场景，而业务开发中不会用到这种方法，因为它是**线程阻塞**的。
- 方式二与 `runBlocking` 的区别在于不会阻塞线程。但不推荐这种用法，因为它的生命周期会和 app 一致，且不能取消。
- 方式三通过 `context` 参数管理和控制协程的生命周期（这里的 `context` 和Android中的不是同一个东西）。



协程最常用的功能是并发，而并发的典型场景就是多线程，可以使用 `Dispatchers.IO` 参数把任务切到 IO线程执行：

```kotlin
coroutineScope.launch(Dispatchers.IO) {
  ...
}
```

也可以使用 `Dispatchers.Main` 切换到主线程：

```kotlin
coroutineScope.launch(Dispatchers.Main) {
  ...
}
```

所以上文协程实现网络请求的完成例子应该是这样的：

```kotlin
coroutineScope.launch(Dispatchers.Main) {		//在主线程开启协程
  val user = api.getUser()		//IO线程执行网络请求
  tvName.text = user.name			//主线程更新UI
}
```

> 这个 `launch` 函数，具体的含义是：创建一个新的协程，并在指定线程上运行它。

而通过Java实现以上逻辑，通常要这样写：

```java
api.getUser(new Callback<User>() {
    @Override
    public void success(User user) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                nameTv.setText(user.name);
            }
        })
    }
    
    @Override
    public void failure(Exception e) {
        ...
    }
});
```

这种回调式的写法，打破了代码的顺序结构和完成性，读起来相当难受。。。



### 协程的 「1到0」

对于回调式的写法，若并发场景再复杂一些，代码的嵌套可能会更多，维护起来就非常麻烦。但Kotlin协程在多层网络请求只需要这么写：

```kotlin
coroutineScope.launch(Dispatchers.Main) {
  val token = api.getToken()
  val user = api.getUser(token)
  
  tvName.text = user.name
}
```

对于多个网络请求需要等待所有请求结束后再对UI进行更新，如果是使用回调式的写法，用并行的写法实现起来既困难又别扭，通过串行的方式实现可能会导致等待时间长了一倍（性能差了一倍）：

```kotlin
api.getAvatar(user, callback)
api.getCompanyLog(user, callback)

api.getAvatar(user) { avatar ->
  api.getCompanyLogo(user) { logo ->
		show(merge(avatar, logo))
	}
}
```

看看通过Kotlin的协程怎么实现：

```kotlin
coroutineScope.launch(Dispathchers.Main) {
  val avatar = async {api.getAvatar(user)}
  val logo = async {api.getCompanyLogo(user)}
  val merged = suspendingMerge(avatar, logo)		//合并结果
  show(merged)
}
```

可以看到比较复杂的并行网络请求，也能够通过协程写出结构清晰的代码。需要注意的是 `suspendingMerge` 并不是协程API中提供的方法，而是自定义的一个可「挂起」的结果合并方法。

#### `launch` 与 `async` 的区别

- 相同点：它们都可以用来启动一个协程，返回的都是 `Coroutine`
- 不同点：`async` 返回的 `Coroutine` 多实现了 `Deferred` 接口，它的意思就是延迟，也就是结果稍后才能拿到；我们可以调用 `Deferred.await()` 可以拿到结果。

上文例子中的 `avatar` 和 `logo` 的类型可以声明为 `Deferred` 通过 `await` 获取结果并更新到UI上

```kotlin
coroutineScope.launch(Dispathchers.Main) {
  val avatar: Deferred = async { api.getAvatar(user) }
  val logo: Deferred = async { api.getCompanyLogo(user) }
  val merged = suspendingMerge(avatar, logo)		//合并结果
  show(merged)
}
```



当需要切换线程或指定线程时，可以通过 `withContext`处理，它可以切换到指定的线程，并在闭包内的逻辑执行结束后，自动把线程切回去继续执行：

```kotlin
coroutineScope.launch(Dispathchers.Main) {
  val image = withContext(Dispathchers.IO) {
    getImage(imageId)
  }
  
	ivAvatar.setImageBitmap(image)
}
```



### `suspend`

`suspend` 是Kotlin的协程最核心的关键字，它的意思是「暂停」或「可挂起」。当代码执行到 `suspend` 函数的时候会「挂起」，并且这个「挂起」是非租塞式的。

`launch`、`async` 或者其它函数创建的协程，在执行到某一个 `suspend` 函数时，这个协程会从当前线程「挂起」，也就是说**这个协程从正在执行它的线程上脱离**。具体到代码其实是：协程的代码块中，线程执行到了 `suspend` 函数时，就暂时不再执行剩余的协程代码，跳出协程的代码块。

**挂起函数在执行完成之后，协程会重新切回它原先的线程**，简单的来讲，在Kotlin中所谓的「挂起」，就是**一个稍后会被自动切回来的线程调度操作**。

> 这个「切回来」的动作，在Kotlin中叫做 恢复（resume）

所以，要求 `suspend` 函数只能在协程里或者另一个 `suspend` 函数里被调用，还是为了要让协程能够在 `suspend` 函数切换线程之后再切回来。



如上例子，看看把 `withContext` 放进一个单独的函数是什么情况：

```kotlin
coroutineScope.launch(Dispathchers.Main) {
  val image = getImage(imageId)
  
	ivAvatar.setImageBitmap(image)
  
  fun getImage(imageId: Int) = withContext(Dispatchers.IO) {
    ...
  }
}
//Error: Suspend function'withContext' should be called only from a coroutine or another suspend funcion
```

大概意思是说，`withContext` 是一个 `suspend` 函数，它需要在协程或者另一个 `suspend` 函数中调用。

上面报错的代码，其实职业需要在前面加个 `suspend` 即可：

```kotlin
coroutineScope.launch(Dispathchers.Main) {
  val image = getImage(imageId)
  
	ivAvatar.setImageBitmap(image)
  
  suspend fun getImage(imageId: Int) = withContext(Dispatchers.IO) {
    ...
  }
}
```

#### 什么时候需要自定义 `suspend` 函数

如果某个函数比较耗时，货含有等待逻辑时，就可以把它写成 `suspend` 函数。

耗时操作一般都分为两类：I/O操作 和 CPU计算工作，比如文件读写、网络交互、图片处理、都是耗时的，另外本身就需要等待的操作，都可以写进 `suspend` 函数里。

如上例子，给函数加上 `suspend` 关键字，然后在 `withContext` 把函数的内容包裹住即可。

上文提到用 `withContext`是因为它在挂起函数里功能最简单直接：把线程自动切走和切回。可以看到它是通过 `Dispatchers` 调度器指定线程类型的：

- `Dispatchers.Main` Android 中的主线程
- `Dispatchers.IO` 针对磁盘和网络 IO 进行了优化，适合 IO 密集型任务，比如：文件读写、数据库操作以及网络请求
- `Dispatchers.Default` 适合 CPU 密集型任务，比如：计算



### `delay`

另外并不是只有 `withContext` 函数可以辅助我们实现自定义 `suspend` 函数，比如还有一个叫：`delay` 函数，它的作用是等待一段时间后再继续往下执行代码。使用它可以实现等待类型的耗时操作：

```kotlin
suspend fun suspendUntilDone() {
  while(!done) {
    delay(5)
  }
}
```



### 非阻塞挂起

协程的挂起，就是非阻塞式的，其实就是在讲协程在挂起的同时切线程这件事情。



### 协程与线程

在Kotlin里，**协程就是基于线程实现的一种更上层的工具API**，类似于 Java 自带的 `Executor` 系列API或者Android的 `Handler` 系列API。

只不过，协程不仅提供了方便的API，在设计思想上市一个**基于线程的上层框架**，可以理解为创造了一些新的概念用来帮你你更好的使用这些API，就像 `ReactiveX` 一样，为了更好的使用各种操作符API，创造了 `Observable` 等概念。



### 总结

- 协程是什么？
  协程就是切线程
- 挂起是什么？
  挂起就是可以自动切回来的切线程
- 挂起的非阻塞式是怎么回事？
  挂起的非阻塞式指的是它能用看起来阻塞的代码写出非阻塞的操作

Kotlin的协程并没有脱离Kotlin或者JVM创造新的东西，它只是将多线程的开发变得更简单了，可以说是因为Kotlin的诞生顺其自然造就的东西。