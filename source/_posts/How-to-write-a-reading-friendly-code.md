title: 写让别人能读懂的代码
date: 2015/10/5 21:00:00
tag:
	- Coding

---

> 原文来自[richieyang](http://www.cnblogs.com/richieyang/p/4840614.html)的博客，根据自己的理解对原文有所更改。

随着软件行业的不断发展，历史遗留的程序越来越多，代码的维护成本越来越大，甚至大于开发成本。而新功能的开发又常常依赖于旧代码，阅读旧代码所花费的时间几乎要大于写新功能的代码。

我前几天看了一本书，书中有这么一句话：

> “复杂的代码往往都是新手所写，只有经验老道的高手才能写出简单，富有表现力的代码”

此话虽然说的有点夸张，可是也说明了经验的重要性。

我们所写的代码除了让机器执行外，还需要别人来阅读。所以我们要写：

1. 让别人能读懂的代码
2. 可扩展的代码
3. 可测试的代码(代码应该具备可测试性，对没有可测试性的代码写测试，是浪费生命的表现)

其中2，3点更多强调的是面向对象的设计原则。而本文则更多关注于局部的代码问题，本文通过举例的方式，总结平时常犯的错误和优化方式。

<!--more-->

本文的例子基于两个指导原则：

> 一.DRY(Don't repeat yourself)

此原则如此重要，简单来说是因为：

- 代码越少，Bug也越少
- 没有重复逻辑的代码更易于维护，当你修复了一个bug，如果相同的逻辑还出现在另外一个地方，而你没意识到，你有没有觉得自己很冤？

> 二.TED原则

- 简洁(Terse)
- 具有表达力(Expressive)
- 只做一件事(Do one thing)

## 举例说明

### 1.拒绝注释，用代码来阐述注释

> 反例：

```php
/// <summary>
/// !@#$%^&^&*((!@#$%^&^&*((!@#$%^&^&*((!@#$%^&^&*((
/// </summary>
/// <returns></returns>
 public decimal GetCash()
 {
     //!@#$%^&^&*((!@#$%^&^&*((
     var a = new List<decimal>() { 2m, 3m, 10m };
     var b = 2;
     var c = 0m;
     //!@#$%^&^&*((!@#$%^&^&*((!@#$%^&^&*((
     foreach (var p in a)
     {
         c += p*b;
     }
     return c;
 }
```

> 重构后：

```php
public decimal CalculateTotalCash()
{
    var prices=new List<decimal>(){2m,3m,10m};
    var itemCount = 2;
    return prices.Sum(p => p*itemCount);
}
```

良好的代码命名完全可以替代注释的作用，如果你正在试图写一段注释，从某种角度来看，你正在试图写一段别人无法理解的代码。

当你无法为你的方法起一个准确的名称时，很可能你的方法不止做了一件事，违反了(Do one thing)。特别是你想在方法名中加入：And，Or，If等词时。

> 注释：方法带Or/And，这个BCL就有不少，IsNullOrEmpty，FirstOrDefault之类，重点不应该是纠结一个方法干了几件事，而是这几件事适不适合放一起干，本来适合放一起的，为了所谓单一原则，非要拆开，就是教条了~

### 2.为布尔变量赋值

> 反例：

```php
public bool IsAdult(int age)
{
    bool isAdult;
    if (age > 18)
    {
        isAdult = true;
    }
    else
    {
        isAdult = false;
    }
    return isAdult;
}
```

> 重构后：

```php
public bool IsAdult(int age)
{
    return age > 18;
}
```

### 3.双重否定的条件判断

> 反例：

```php
if (!isNotRemeberMe)
{

}
```

> 重构后：

```php
if (isRemeberMe)
{

}
```

不管你有没有见过这样的条件，反正我见过。见到这样的条件判断，我顿时就晕了。

### 4.拒绝HardCode,拒绝挖坑

> 反例：

```php
if (carName == "Nissan")
{

}
```

> 重构后：

```php
if (car == Car.Nissan)
{

}
```

既然咱们玩的是强类型语言，咱就用上编译器的功能，让错误发生在编译阶段。

### 5.拒绝魔数，拒绝挖坑

> 反例：

```php
if (age > 18)
{

}
```

> 重构后：

```php
const int adultAge = 18;
 if (age > adultAge)
{

}
```

所谓魔数(Magic number)就是一个魔法数字，读者完全弄不明白你这个数字是什么，这样的代码平时见的多了。

### 6.复杂的条件判断

> 反例：

```php
if (job.JobState == JobState.New
    || job.JobState == JobState.Submitted
    || job.JobState == JobState.Expired
    || job.JobTitle.IsNullOrWhiteSpace())
{
    //....
}
```

> 重构后：

```php
 if (CanBeDeleted(job))
{
    //
}

private bool CanBeDeleted(Job job)
{
    var invalidJobState = job.JobState == JobState.New
                          || job.JobState == JobState.Submitted
                          || job.JobState == JobState.Expired;
    var invalidJob = string.IsNullOrEmpty(job.JobTitle);

    return invalidJobState || invalidJob;
}
```

有没有豁然开朗的感觉？

### 7.嵌套判断

> 反例：

```php
var isValid = false;
if (!string.IsNullOrEmpty(user.UserName))
{
    if (!string.IsNullOrEmpty(user.Password))
    {
        if (!string.IsNullOrEmpty(user.Email))
        {
            isValid = true;
        }
    }
}
return isValid;
```

> 重构后：

```php
if (string.IsNullOrEmpty(user.UserName)) return false;
if (string.IsNullOrEmpty(user.Password)) return false;
if (string.IsNullOrEmpty(user.Email)) return false;
return true;
```

第一种代码是受到早期的某些思想：使用一个变量来存储返回结果。事实证明，你一旦知道了结果就应该尽早返回。

### 8.使用前置条件

> 反例：

```php
if (!string.IsNullOrEmpty(userName))
{
    if (!string.IsNullOrEmpty(password))
    {
        //register
    }
    else
    {
        throw new ArgumentException("user password can not be empty");
    }
}
else
{
    throw new ArgumentException("user name can not be empty");
}
```

> 重构后：

```php
if (string.IsNullOrEmpty(userName)) throw new ArgumentException("user name can not be empty");
if (string.IsNullOrEmpty(password)) throw new ArgumentException("user password can not be empty");
//register
```

重构后的风格更接近契约编程，首先要满足前置条件，否则免谈。

### 9.参数过多，超过3个

> 反例：

```php
public void RegisterUser(string userName, string password, string email, string phone)
{

}
```

> 重构后：

```php
public void RegisterUser(User user)
{

}
```

过多的参数让读者难以抓住代码的意图，同时过多的参数将会影响方法的稳定性。另外也预示着参数应该聚合为一个Model。

### 10.方法签名中含有布尔参数

> 反例：

```php
public void RegisterUser(User user, bool sendEmail)
{

}
```

> 重构后：

```php
public void RegisterUser(User user)
{

}

public void SendEmail(User user)
{

}
```

布尔参数在告诉方法不止做一件事，违反了Do one thing。

### 11.写具有表达力的代码

> 反例：

```php
private string CombineTechnicalBookNameOfAuthor(List<Book> books, string author)
{
    var filterBooks = new List<Book>();

    foreach (var book in books)
    {
        if (book.Category == BookCategory.Technical && book.Author == author)
        {
            filterBooks.Add(book);
        }
    }
    var name = "";
    foreach (var book in filterBooks)
    {
        name += book.Name + "|";
    }
    return name;
}
```

> 重构后：

```php
private string CombineTechnicalBookNameOfAuthor(List<Book> books, string author)
 {
     var combinedName = books.Where(b => b.Category == BookCategory.Technical)
         .Where(b => b.Author == author)
         .Select(b => b.Name)
         .Aggregate((a, b) => a + "|" + b);

     return combinedName;
 }
```

相对于命令式代码，声明性代码更加具有表达力，也更简洁。这也是函数式编程为什么越来越火的原因之一。

## 关于DRY

平时大家重构代码，一个重要的思想就是DRY。我要分享一个DRY的反例：

项目在架构过程中会有各种各样的MODEL层，例如：DomainModel，ViewModel，DTO。很多时候这几个Model里的字段大部分是相同的，于是有人就会想到DRY原则，干脆直接用一种类型，省得粘贴复制，来回转换。

这个反例失败的根本原因在于：这几种Model职责各不相同，虽然大部分情况下内容会有重复，但是他们担当着各种不同的角色。

考虑这种场景：

DomainModel有一个字段`DateTime Birthday{get;set;}`，ViewModel同样具有`DateTime Birthday{get;set;}`。需求升级:要求界面不再显示生日，只需要显示是否成年。我们只需要在ViewModel中添加一个`Bool IsAdult{get{return ....}}`即可，DomainModel完全不用变化。

## 利用先进的生产工具

以vs插件中的Reshaper为例，本文列举的大部分反例，Reshaprer均能给予不同程度的提示。经过一段时间的练习，当Reshaper对你的代码给予不了任何提示的时候，你的代码会有一个明显的提高。

截图说明Reshaper的提示功能：

![reshaper1](http://data.kchen.cc/imageo_20150926142447.png-960.jpg)

![reshaper2](http://data.kchen.cc/imageo_20150926142514.png-960.jpg)

光标移动在波浪线处，然后`Alt+Enter`,Resharper 会自动对代码进行优化。

## 总结

如果你能够避免本文总结的反例，你的代码就已经具备了优秀代码应有的基因。当然高质量的代码还需要良好的设计和遵循面向对象编程的原则。 如果想了解更多相关内容，请阅读《代码大全》，《代码整洁之道》，《重构 改善既有代码的设计》，《敏捷软件开发 原则、模式与实践》。