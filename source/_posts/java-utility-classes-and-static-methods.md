title: Java - 工具类和静态方法
date: 2017/02/21 14:12:16
updated: 
tags:
    - Java
    - Tips
---

**工具类**定义了一组通用的、功能复用的**方法**，它们不依赖于任何的**对象**，所以这些方法应该用 `static` 关键字声明成**静态的**。

```java
public class Utility {
  private Utility() {
  }
  public static multiply(int a, int b) {
    return a * b;
  }
  ...
}
```

上面的例子中，`multiply` 方法就是静态的，所以我们不必生成任何 `Utility` 类的**实例**来调用该方法。

为了防止 `Utility` 类被调用于**实例化**，我们可以将**构造函数**用 `private` 关键字定义成**私有的**。

> 更多关于静态方法的内容，请参阅[相关文档](http://javarevisited.blogspot.jp/2011/11/static-keyword-method-variable-java.html)。