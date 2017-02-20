title: Java - 在泛型中使用受限类型参数
date: 2017/02/19 17:19:14
updated: 
tags:
    - Java
---

许多时候在 Java 中使用泛型是非常方便且必要的，但是我们不可接受所有类型的泛型来确保每一个功能都能得到维护。

为了解决这个问题，我们可以使用受限类型参数来限制泛型只能接受特定类型的实际参数。

```java
public <T extends Shape> 
  void drawAll(List<T> shapes){
    for (Shape s: shapes) {
        s.draw(this);
    }
}
```

上面这个方法是用来绘制一组形状（`Shape`）的，如果我们使用不受限类型参数的泛型来实现这个方法的话，有可能导致的问题是其他类型可能并不能并不具备 `draw()` 方法。

通过指定 `<T extends Shape>` 我们可以确保只有 `Shape` 的子类可以被传递到这个方法中来。

> 关于这一部分的更多内容，详见[官方文档](https://docs.oracle.com/javase/tutorial/java/generics/bounded.html)。
