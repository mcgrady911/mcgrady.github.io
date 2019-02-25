---
title: 设计模式
date: 2018-01-23 21:30:50
updated: 2018-01-23 21:30:50
tags: [android]
categories: android
---

# Design Pattern



## SOLID 设计原则



### 单一职责原则（The Single Responsibility Principle ）

#### 类的单一原则
#### 接口的单一原则

接口的单一的职责是**定义单一职责或相关职责的一组方法。**

#### 方法的单一原则

方法的单一职责原则是**一个方法实现一个功能，解决一个方法。**

#### 总结

**单一职责原则的优点：**

- 降低复杂性
- 提高可读性
- 提高可维护性

**单一原则的定义：**

单一职责原则提出了一个编写程序的标准，用“职责”或“变化原因”来衡量接口或
类设计得是否优良，但是“职责”和“变化原因”都是不可度量的，因项目而异，因环境而异。
对于单一职责原则，**建议接口一定要做到单一职责，类的设计尽量做到只有一个原因引起的变化**。



### 开放封闭原则（The Open Closed Principle ）

#### 定义

**一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。**

#### 总结

**这个原则有两个特性，一个是说“对于扩展是开放的”，另一个是说“对于更改是封闭的”。面对需求，对程序的改动是通过增加新代码进行的，而不是更改现有的代码。这就是“开放-封闭原则”的精神所在。**



### 里氏替换原则（The Liskov Substitution Principle ）

#### 定义1
如果对每一个类型为 T1的对象 o1，都有类型为 T2 的对象o2，使得以 T1定义的所有程序 P 在所有的对象 o1 都代换成 o2 时，程序 P 的行为没有发生变化，那么类型 T2 是类型 T1 的子类型。
#### 定义2
**所有引用基类的地方必须能透明地使用其子类的对象**。

通俗点讲，只要父类能出现的地方子类就可以出现，而且替换为子类也不会产生任何错误或异常，使用者可能根本就不需要知道是父类还是子类。但是，反过来就不行了，有子类出现的地方，父类未必就能适应。

#### 总结

继承作为面向对象三大特性之一，在给程序设计带来巨大便利的同时，也带来了弊端。比如使用继承会给程序带来侵入性，程序的可移植性降低，增加了对象间的耦合性，如果一个类被其他的类所继承，则当这个类需要修改时，必须考虑到所有的子类，并且父类修改后，所有涉及到子类的功能都有可能会产生故障。
**使用继承时应遵循里式替换原则。**



### 依赖倒置原则（The Dependency Inversion Principle ）

#### 定义
高层模块哦不应该依赖底层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。

#### 总结

**依赖倒置原则就是要我们面向接口编程，理解了面向接口编程，也就理解了依赖倒置。**

```java
public class Client {

	public static void main(String[] args) {
        
        // 通过构造传递依赖对象
        ICar brz = new Brz();
        IDriver jacket = new Driver(brz);
        jacket.drive();
        
        // 通过接口声明依赖对象
        IDriver jacket = new Driver();
        ICar brz = new Brz();
        jacket.drive(brz);
        
        // 通过Setter方法传递依赖对象
        IDriver jacket = new Driver();
        jacket.setCar(new Brz());
        jacket.drive();
	}
}

public interface ICar {
	public void run();
    //or
    public void run(ICar car);
}

public class Brz implements ICar {
	@Override
	public void run() {
		System.out.println("Brz start running");
	} 
}

public interface IDriver {
	public void drive();
}

public class Driver implements IDriver {

    private ICar car;
    
    public Driver(ICar car) {
        this.car = car;
    }
   
	@Override
	public void drive() {
		car.run();
	}
    //or
    @Override
    public void drive(ICar car) {
        car.run();
    }
    
    public void setCar(Icar car) {
        this.car = car;
    }
}
```



### 接口分离原则（The Interface Segregation Principle ）

#### 定义

建立功能单一细化的接口，接口中的方法尽量少，避免接口庞大臃肿。通过分散定义多个接口，可以预防外来变更的扩散，提高系统的灵活性和可维护性。

