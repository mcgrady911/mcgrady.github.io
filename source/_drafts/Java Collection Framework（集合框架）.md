# Java Collection Framework（集合框架）

[TOC]



## 前言
在面向过程的C语言中，数据结构是用`struct`来描述，而在面向对象的编程中，数据结构是用类来描述的，并且包含有对该数据结构操作的方法。
在Java语言中，Java语言的设计者对常用的数据结构和算法做了一些规范（接口）和实现（具体实现接口的类）。**所有抽象出来的数据结构和操作（算法）统称为Java集合框架（Java Collection Framework）**。
![Alt text](./A192136220-94766.jpg)
![Alt text](./Image.png)





## Collection
### Set
无序，不允许重复。
#### TreeSet
#### HashSet




### List
有序，可有重复元素。
#### LinkedList
LinkedList类是双向链表，列表中的每个节点都包含了对前一个和后一个元素引用。
LinkedList实现了Queue接口。
#### Vector
#### ArrayList




### Map
具有映射关系的集合，其中的元素是以键值对`key-value`的形式存储的，能够实现根据`key`快速查找`value`。
#### TreeMap
#### HashMap
##### LinkedHashMap




### Queue
队列集合
Queue接口集成了Collection接口。

