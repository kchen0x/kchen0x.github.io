title: Java - 双花括号初始化
date: 2017/02/22 15:54:09
updated: 
tags:
    - Java
    - Tips
---

相较于手动地**初始化**具有初始元素的 set、list 和 map，Java 提供了另外一种更直接和简便的方法来快速完成：**双花括号初始化**。例如下方的代码：

```java
public Set<String> mySet = new 
HashSet<String>();
mySet.add("one");
mySet.add("two");
mySet.add("three");

someFunction(mySet);
```

就可以利用**双花括号初始化**简化成这样：

```java
someFunction(new HashSet<String>() {{
  add("one");
  add("two");
  add("three");
}});
```

> 更多资料，请查看[相关文档](http://wiki.c2.com/?DoubleBraceInitialization)。