# Java 数据结构和复合框架



### try catch finally 语句块

#### 执行规则：

1. 如果**只有**`try`语句有`return`返回值，此后在`catch、finally`中对变量做任何的修改，都不影响`try`中`return`的返回值。
2. `try、catch`中有返回值，而`try`中抛出的异常恰好与`catch`中的异常匹配，则返回`catch`中的`return`值。
3. 如果`finally`块中有`return`语句，则返回`try`或`catch`中的返回语句忽略。
4. 如果`finally`块中抛出异常，则整个`try、catch、finally`块中抛出异常.并且没有返回值。

#### 使用try catch finally是需要注意的几点：

1. 尽量在`try`或者`catch`中使用`return`语句。通过`finally`块中达到对`try`或者`catch`返回值修改是不可行的。

2. `finally`块中避免使用`return`语句，因为`finally`块中如果使用`return`语句，会显示的忽略掉`try、catch`块中的异常信息，屏蔽了错误的发生。

3. `finally`块中避免再次抛出异常，否则整个包含try语句块的方法回抛出异常，并且会忽略掉`try、catch`块中的异常。



















































