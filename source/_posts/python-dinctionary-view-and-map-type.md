title: Python - 字典的视图对象和映射类型
date: 2017/02/21 10:49:10
updated: 
tags:
    - Python
    - Tips
---

**字典**是 Python 中非常常用的一种**集合**类型，今天我们来看一下关于字典的**视图对象**和**映射类型**。

<!--more-->

## 字典的视图对象

Python 的字典提供以下三种方法来获取**视图对象**：

```python
dict.keys() 
# returns a new view of dictionary's keys
dict.values()
# returns a new view of dictionary's values
dict.items()
# returns a new view of dictionary's items
# items are (key, value) pairs
```

**视图对象**将会动态地与字典的**条目**绑定，也就是说，当字典发生变化的时候，他的视图对象也会相应地发生变化。

我们可以通过 `len(dictview)` 方法来获取字典中条目的数量：

```python
>>> a = {'one':1, 'two':2}
>>> keys = a.keys()
>>> len(keys)
2
```

字典的 `iter()` 方法对字典的视图也行之有效，其会返回基于**键**、**值**或者**键值对**的**迭代器**：

```python
iter(keys)
# iterator over keys is returned
```

同样的，使用 `in` 关键词来判定集合的成员也可以同时用在字典或者字典的视图上：

```python
>>> 'one' in keys
True
>>> 'three' in keys
False
```

> 更多关于字典的视图对象，请参阅[官方文档](https://docs.python.org/3.5/library/stdtypes.html#dictionary-view-objects)。

## 字典的标准映射类型

**字典**（`dict`）是 Python 的主要**映射类型**，它将**可哈希化**（hashable）的值映射为任意的对象。Python 中的字典和 Perl 语言、Ruby 语言中的哈希，C 语言中的哈希表是相似的。

我们可以使用 `{key: value}` 语法，`dict` 构造器或者会返回**元组**的 `zip` 方法来生成一个**字典**：

```python
d1 ={'first':1, 'second':2}
d2 = dict(first=1, second=2)
d3 = dict(zip(['first','second'], [1,2]))
d4 = dict({'second':2, 'first':1})
```

在**字典**上获取和使用一个**键**的**迭代器**：

```python
>>> it = iter(d1)
>>> type(it)
<class 'dict_keyiterator' at
0x000000005801A420>
>>> for k in it: print(k)
...
first
second
```

用字典来更新另一个字典：

```python
>>> prefs = {"fruit": "apple", "car": "Ford"}
>>> prefs
{'car': 'Ford', 'fruit': 'apple'}
>>> prefs2 = {"fruit": "orange"}
>>> prefs.update(prefs2)
>>> prefs
{'car': 'Ford', 'fruit': 'orange'}
```

还可以用一个旧的字典（`seq`）来构造一个新的字典，让新的字典和 `seq` 有相同的键：

```python
seq = ('one', 'two')
dict = dict.fromkeys(seq)
# dict is {'one': None, 'two': None }
dict = dict.fromkeys(seq,10)
# dict is  {'one: 10, 'two': 10 }
```

> 更多关于字典的标准映射类型，请参阅[官方文档](https://docs.python.org/3.5/library/stdtypes.html#mapping-types-dict)。