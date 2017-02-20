title: Java - 通过 default 更新接口类方法
date: 2017/02/19 16:57:50
updated: 
tags:
    - Java
---

假设有下列接口：


```java
public interface Cooking {
    public void fry();
    public void boil();
    public void chop();
}
```

为了添加新的功能，直接向 `Cooking` 接口类中增加 `microwave()` 方法将会造成不少的麻烦。任何实现了 `Cooking` 接口的类现在都必须要同步更新实现这个方法，以确保整个类重新生效、正常工作。

为了避免这种情况，我们可以给 `microwave()` 方法一个默认 `default` 实现：

```java
public interface Cooking {
    public void fry();
    public void boil();
    public void chop();
    default void microwave() {
    //some code implementing microwave
    }
}
```

因为 `microwave()` 方法已经在 `Cooking` 接口类中有了默认实现的定义，所有实现了该接口的类都不需要额外实现 `microwave()` 方法来确保其正常工作了。

这个方法在我们不破坏现有代码的情况下，想要增加一些新的功能时，十分的好用。

> Note：该方法需要 Java 8 以上的支持。详见[官方文档](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)。
