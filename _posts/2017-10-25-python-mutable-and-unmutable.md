---
layout: post
keywords: python, list, tuple, mutable, unmutable, 可变对象, 不可变对象
description: the diff between python mutable and unmutable, python中可变对象和不可变对象的区别
title: python中可变对象和不可变对象的区别
comments: true
---

{{ page.title }}
<p class="meta">25 Oct 2017</p>
<hr>

对于可变对象，比如列表，+= 运算符就地修改列表。

```
>> l = [1, 2]
>> print(id(l))
4396812360
>> l += [3, 4]
>> print(id(l))
4396812360
```

对于不可变对象，比如元组， += 运算符会创建新对象，然后重新绑定给变量。

```
>> t = (1, 2)
>> print(id(t))
4397257032
>> t += (3, 4)
>> print(id(t))
4395475416
```
当然包括 * =。


```
>> l1 = [1, 2, 3]
>> print(id(l1))
4333714888
>> l2 = list(l1)
>> print(id(l2))
4334138760
>> l3 = l1[:]
>> print(id(l3))
4334286792
```

对于不可变对象，比如tuple
```
>> t1 = (1, 2, 3)
>> print(id(t1))
4331049968
>> t2 = tuple(t1)
>> print(id(t2))
4331049968
>> t3 = t1[:]
>> print(id(t3))
4331049968
```
还包括str, bytes, frozenset。

不可变对象不变只是所含对象的标识。
