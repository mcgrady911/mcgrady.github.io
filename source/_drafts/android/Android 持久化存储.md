# Android 持久化存储

SQLite、MySql等数据库的CRUD及SharedPreferences的数据持久化


SQLite:


①SQLite概述：

Android平台中嵌入了一个关系型数据库SQLite，和其他数据库不同的是SQLite存储数据时不区分类型
SQLite是手机自带的数据库，每一个数据库就是一个XML文件，每一个XML文件有一张或多张表

②创建SQLite数据库步骤：

a、定义类继承SQLiteOpenHelper
SQLiteOpenHelper是用来打开数据库的一个工具，其中有创建onCreate()数据库和升级upGrade()数据库方法，在获取数据库连接时，这些方法会自动执行,使用手机自带的SQLite数据库必须创建一个SQLiteOpenHelper子类，实现其onCreate(SQLiteDatabase), onUpgrade(SQLiteDatabase, int, int)方法，如果数据库不存在则创建，存在则检查数据库版本是不是最新，不是最新则升级数据库版本，维护其保持一个最佳的状态
b、获取数据库SQLiteDatabase对象
SQLiteOpenHelper. getWritableDatabase()：获取写数据库，当可写时可获取读数据库
SQLiteOpenHelper. getReadableDatabase()：获取读数据库
c、执行数据库增删改查操作
执行增删改操作，无返回结果集：SQLiteDatabase.execSQL(String sql,Object[] params);
执行查询操作：返回Cursor结果集SQLiteDatabase.rawQuery(String sql,String[] params);
或者执行CRUD操作：
SQLiteDatabase.insert();
SQLiteDatabase.update();
SQLiteDatabase.delete();
SQLiteDatabase.query();
d、关闭资源：(可选操作，每次获取数据库的时候数据库底层会自动关流，建议手动关闭)
SQLiteDatabase.close();
Cursor.close();



MySql:


①MySql概述:

结构化查询语言,Structured Query Language的缩写

②JDBC简介：

a、JDBC：java数据库连接，Java DataBase Connectivity的缩写
b、开发人员安装了不同的数据库服务器(如MySQL和Oracle)，则需要安装不同的数据库驱动，才能在java应用程序中连接该数据库并对该数据库的数据进行操作，而这样对开发人员来说是很麻烦的,于是乎SUN就提供了一套操作数据库的规范，该规范就是JDBC,不同数据库厂商实现了该规范(驱动)，开发人员只要按照该规范开发就行了
c、JDBC规范的类在JDK中的

③JDBC编程步骤：

a、注册驱动：
方式一：DriverManager.registDriver(new com.mysql.jdbc.Driver())
注意该Driver所在的包是com.mysql.jdbc.，而不是java.sql.包中
该种方式不建议使用，原因如下：
****严重依赖具体的数据库驱动程序,DriverManager是java.sql.*包中的
如果改用Orcal数据库，则得改程序
****会导致数据库的驱动程序注册2次，DriverManager在类一加载的时候
就注册了一次,后面再new com.mysql.jdbc.Driver()又注册一次
(查看源代码)
方式二：Class.forName("com.mysql.jdbc.Driver");开发用此方式，次方式避免了a中方式一出现的问题
b、建立与数据库的链接：java.sql.Connection，所有的与数据库的交互都必须在链接的基础上进行操作
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/day12","root", "xiaruri");
注：url的固定格式写法要记住："jdbc:mysql://localhost:3306/day12"
jdbc:代表协议  mysql:代表子协议   localhost:3306:代表主机和端口  day12:代表数据库
root:代表MySQL服务器用户名    xiaruri:用户名密码
实际开发中需把传进去的三个字符串抽取出来，写成配置文件，通过读取配置文件形式将读取到的传进去，这样避免换数据库时改程序，只需改配置文件即可
c、获取代表SQL语句对象Statement stmt = conn.createStatement();
d、执行SQL语句 String sql = "select id,name,password,email,birthday from users";
e、如果有结果集，会被封装成一个ResultSet对象ResultSet rs = stmt.executeQuery(sql);
f、遍历结果集
while(rs.next()){
//rs.getObject(1):结果中每条记录的列索引从1开始。开发JDBC框架时使用
System.out.println(rs.getObject("id")+"\t"+rs.getObject("name")+"\t"+
rs.getObject("password")+"\t"+rs.getObject("email")+
"\t"+rs.getObject("birthday"));
}
h、关闭使用的资源
rs.close();
stmt.close();
conn.close();

④开源的数据源框架：

a、可直接访问硬件
b、DBCP
DBCP是Apache软件基金组织下的开源连接池实现，使用DBCP数据源应用程序应在系统中增加如下
两个jar文件：
Commons-dbcp.jar：连接池的实现
Commons-pool.jar：连接池实现的依赖库
常用O-R Mapping映射工具
Hibernate     CMP JPA(Java Persistent API)
Ibatis(后来改名为MyBatis)
Commons DbUtils(只是对JDBC简单封装)
Spring JDBC Template
DbUtils简介：QueryRunner
MyBatis简介：MyBatis的前身就是iBatis。她是一个数据持久层框架
MyBatis支持普通SQL查询、存储过程和高级映射的优秀持久层框架
消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索，MyBatis使用简单的XML或注解用于配置和原始映射 , 将接口和Java的POJOs(Plain Old Java Objects，普通的Java对象)映射成数据库中的记录
c、C3P0



SharedPreferences：



SharedPreferences对象就相当于一个可持久化的Map集合
通过put方法存储持久化数据，需要commit才有效
通过get获取持久化数据

作者：lucas777
链接：https://www.jianshu.com/p/b3bf4859f470
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。