# RecyclerView 详解

## RecyclerView 基本设计结构

![](../images/android/RecyclerView.png)

### ViewHolder
对于Adapter来说，一个ViewHodler对应的是数据集合中的一个data，也是Recycler缓存池的基本单元。

### Adapter
Adapter的工作是把data和View进行绑定，主要负责ViewHolder的创建以及数据变化时通知RecyclerView。

### AdapterDataObservable
AdapterDataObservable是数据源变化时的被观察者。
Adapter是数据源的直接接触者，当数据源发生变化时，它需要通知给RecyclerView。这里使用的模式是观察者模式。
`adapter.notifyXX()`方法实际上会调用AdapterDataObservable的`notifyChanged()`

### RecyclerViewDataObservable
RecyclerViewDataObserver是观察者。
用来监听Adapter数据变化的观察者。

### LayoutManager
它是RecyclerView的布局管理者，RecyclerView在``onLayout()``时，会利用它来layoutChildren,它决定了RecyclerView中的子View的摆放规则。但不止如此, 它做的工作还有:
1. 测量子View
2. 对子View进行布局
3. 对子View进行回收
4. 对子View动画的调度
5. 负责RecyclerView滚动的实现
### Recycler
对于LayoutManager来说，它是ViewHolder的提供者。对于RecyclerView来说，它是ViewHolder的管理者，是RecyclerView最核心的实现。
![1](../images/android/RecyclerView_Recycler.png)

### RecycledViewPool
它是一个可以被复用的ViewHolder缓存池。即可以给多个RecycledView来设置统一个RecycledViewPool，内部利用一个ScrapData来保存ViewHolder集合。

## RecyclerView 刷新机制

## RecyclerView 复用机制