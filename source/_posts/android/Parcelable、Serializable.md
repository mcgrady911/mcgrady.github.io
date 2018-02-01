# Parcelable、Serializable
[TOC]

在Android开发中，不能将对象的引用传递给Activity或者Fragments，
通过Android的API，将对象进行Parcelable或者Serializable化，将这些对象需要放到Intent或者Bundle里，即可传递复杂的数据类型或者自定义类。



## Serializable

Serializable是一种表示接口（marker interface），这意味着无需实现方法，Java便会对这个对象进行高效的序列化操作。
```java
public class SerializableDeveloper implements Serializable
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;
 
    static class Skill implements Serializable {
        String name;
        boolean programmingRelated;
    }
}
```



## Parcelable

```java
class ParcelableDeveloper implements Parcelable {
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;
 
    ParcelableDeveloper(Parcel in) {
        this.name = in.readString();
        this.yearsOfExperience = in.readInt();
        this.skillSet = new ArrayList<Skill>();
        in.readTypedList(skillSet, Skill.CREATOR);
        this.favoriteFloat = in.readFloat();
    }
 
    void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(yearsOfExperience);
        dest.writeTypedList(skillSet);
        dest.writeFloat(favoriteFloat);
    }
 
    int describeContents() {
        return 0;
    }
 
 
    static final Parcelable.Creator<ParcelableDeveloper> CREATOR
            = new Parcelable.Creator<ParcelableDeveloper>() {
 
        ParcelableDeveloper createFromParcel(Parcel in) {
            return new ParcelableDeveloper(in);
        }
 
        ParcelableDeveloper[] newArray(int size) {
            return new ParcelableDeveloper[size];
        }
    };
 
    static class Skill implements Parcelable {
        String name;
        boolean programmingRelated;
 
        Skill(Parcel in) {
            this.name = in.readString();
            this.programmingRelated = (in.readInt() == 1);
        }
 
        @Override
        void writeToParcel(Parcel dest, int flags) {
            dest.writeString(name);
            dest.writeInt(programmingRelated ? 1 : 0);
        }
 
        static final Parcelable.Creator<Skill> CREATOR
            = new Parcelable.Creator<Skill>() {
 
            Skill createFromParcel(Parcel in) {
                return new Skill(in);
            }
 
            Skill[] newArray(int size) {
                return new Skill[size];
            }
        };
 
        @Override
        int describeContents() {
            return 0;
        }
    }
}
```



## Parcelable和Serializable的区别

**优点：**
Serializable 简单易用
Parceable 速度至上
**缺点：**
Serializable 使用了反射，序列化过程较慢。这种机制会在序列化的时候创建许多的临时对象，容易出发垃圾回收。
Parcelable 实现Parcelable接口需要大量的模板代码，这使得对象代码变得难以阅读和维护。



## 速度测试

### 测试方法
- 通过一个对象放到一个bundle里，然后调用Bundle#writeToParcel(Parcel, int)方法来模拟传递对象给一个activity的过程，然后再把这个对象取出来。
- 在一个循环里运行1000次。
- 两种方法分别运行10次减少内存管理，CPU被其他应用占用等情况的干扰。
- 参与测试的对象就是代码中的SerializableDeveloper和ParcelableDeveloper
- 在多种Android软硬环境上进行测试

### 结果
![parcelable-vs-serializable](./parcelable-vs-serializable-e1366334109758.png)
**由此可以得出：**Parcelable 比 Serializable快了10多倍。有趣的是，即使在Nexus 10这样性能强悍的硬件上，一个相当简单的对象的序列化和反序列化的过程要花将近一毫秒。



## 总结

如果你想成为一个优秀的软件工程师，你需要花多点时间来实现Parcelable，因为这将会为你对象的序列化过程快10多倍，而且占用较少的资源。

但是大多情况下，Serializable的龟速不会太引人注目。你想偷点懒就用它把，不过要记得serialization是一个比较耗资源的操作，尽量少用。

如果你想传递一个包含多个对象的列表，那么整个序列化的过程的时间开销可能会超过一秒，这回让屏幕转向的时候变得很卡顿。

