title: Java - 使用 WeakHashMap
date: 2017/02/22 15:12:31
updated: 
tags:
    - Java
    - Tips
---

`WeakHashMap` 是一种特别的 `Map` 实现，它的所有键都存储在 `WeakReference` 中。相较于 `HashMap`，`WeakHashMap` 与其功能基本完全一致，除了唯一一个显著的差别：如果 Java 的内存管理器不再有某个键对象的**强引用**，那么这个**条目**就会被从该 Map 中移除。

我们这样来创建一个 `WeakHashMap`：

```java
HashMap map = new WeakHashMap();
```

我们可以通过引用某些对象的方式使用 `WeakHashMap` 来保存资源，但是同时允许这个资源在没有引用的时候被 Java 的**垃圾回收器**回收：

```java
System.gc();
```

所以大家普遍认为，可以通过 `WeakHashMap` 来解决缓存问题，因为当引用过期之后就会被丢弃。

这个类的另一种用法是创建 canonical map，你可以存储额外的属性在某个对象里面，因为当它们过期之后，`WeakHashMap` 的相应条目就会立马被丢弃。

> 关于 `WeakHashMap` 的使用例子，请阅读[参考资料](http://stackoverflow.com/questions/10599710/weakhashmap-example)。