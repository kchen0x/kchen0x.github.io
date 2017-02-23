title: Java -  对非不可变对象使用拷贝构造器
date: 2017/02/22 16:03:18
updated: 
tags:
    - Java
    - Tips
---

比起使用 `clone` 方法，在 Java 中拷贝一个对象更加简单的办法是使用**拷贝构造器**。比方说，我们的 `Person` 类中有如下的构造函数：

```java
public Person(String name, int age) {
  this.name = name;
  this.age = age;
}
```

那么就可以有这样的拷贝构造器：

```java
public Person(Person person) {
  this(person.getName(), person.getAge());
}
```

该构造器可以使用同样的 `name` 和 `age` 属性，创建另一个 `Person` 对象。然而，必须要注意的是，拷贝**不可变对象**是完全没有必要的，所以我们只能用这种拷贝构造器来操作那些状态可变的对象。举例来说，只要 `Person` 类有这样的方法：

```java
public void setName(String name) {
  this.name = name;
}
```

那么就大可使用拷贝构造器。

> 更多关于拷贝构造器的内容，请参阅[相关文档](http://www.javapractices.com/topic/TopicAction.do?Id=71)。