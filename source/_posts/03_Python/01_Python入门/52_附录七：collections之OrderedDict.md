---
title: 52-附录七：Collections之OrderedDict
date: 2022-07-18 10:52:22
categories:
- Python
- Python入门
tags:
---

## **collections之OrderedDict**😀

 如果想让字典有序，可以使用collections.OrderedDict，它现在在C中实现，这使其快4到100倍。

\##1、collections.OrderedDict的基本使用  将类OrderedDict实例化会得到一个dict子类的实例，支持通常的dict方法。

```text
from collections import OrderedDict

od=OrderedDict()
print(isinstance(od,OrderedDict)) # True
print(isinstance(od,dict)) # True
```

 OrderedDict是记住键首次插入顺序的字典。如果新条目覆盖现有条目，则原始插入位置保持不变。

```text
od['name'] = 'egon'
od['age'] = 18
od['gender'] = 'male'
print(od) # OrderedDict([('name', 'egon'), ('age', 18), ('gender', 'male')])

od['age']=19
print(od) # OrderedDict([('name', 'egon'), ('age', 19), ('gender', 'male')])
```

 删除条目并重新插入会将其移动到末尾。

```text
del od['age']

od['age']=20
print(od) # OrderedDict([('name', 'egon'), ('gender', 'male'), ('age', 20)])
```

## **2、方法popitem(last=True)**

 调用有序字典的popitem()方法会删除并返回(key, value)对。如果last为真，则以LIFO(后进先出)顺序返回这些键值对，如果为假，则以FIFO(先进先出)顺序返回。

```text
from collections import OrderedDict

od=OrderedDict()

od['k1']='egon'
od['k2']='tom'
od['k3']='jack'

print(od.popitem(last=False))
print(od.popitem(last=False))
print(od.popitem(last=False))
'''
('k1', 'egon')
('k2', 'tom')
('k3', 'jack')
'''
```

## **3、方法move_to_end(key, last=True)**

 该方法用于将一个已存在的key移动到有序字典的任一端。如果last为True（默认值），则移动到末尾，如果last为False，则移动到开头。如果key不存在，引发KeyError

```text
from collections import OrderedDict

od = OrderedDict()

od = OrderedDict.fromkeys('abcde')
od.move_to_end('b')
print(''.join(od.keys())) # acdeb

od.move_to_end('b', last=False)
print(''.join(od.keys())) # bacde
```

## **4、OrderDict对象之间的相等性判断**

 OrderedDict对象之间的相等性判断是顺序敏感的

```text
判断：od1 == od2
底层实现相当于：list(od1.items()) == list(od2.items())
```

OrderedDict对象与其他映射对象之间的相等性测试与常规字典类似，对顺序不敏感，所以我们可以在使用常规字典的任何位置替换为OrderedDict对象，并不会影响使用。

```text
od1=OrderedDict()
od2=OrderedDict()
od3=OrderedDict()

od1['k1']=111
od1['k2']=222
od1['k3']=333


od2['k1']=111
od2['k2']=222
od2['k3']=333

od3['k1']=111
od3['k3']=333
od3['k2']=222


print(od1 == od2) # OrderDict之间的相等判断，即list(od1.items())==list(od2.items())，所以结果为True
print(od1 == od3) # OrderDict之间的相等判断，即list(od1.items())==list(od3.items())，所以结果为False



d={'k1':111,'k3':333,'k2':222} # 定义常规字典

print(od1 == d) # OrderDict对象与常规字典比较，对顺序不敏感，所以结果为True
```

## **5、OrderedDict构造函数和update()**

 OrderedDict构造函数和update()方法都可以接受关键字参数，但是它们的顺序丢失，因为OrderedDict构造函数和update()方法都属于Python的函数调用，而Python的函数调用语义使用常规无序字典传递关键字参数。请在python2中测试

```text
from collections import OrderedDict
od1=OrderedDict(x=1,y=2,z=3)
print(od1) # 顺序错乱：OrderedDict([('y', 2), ('x', 1), ('z', 3)])


od2=OrderedDict()
od2.update(a=1)
od2.update(b=2)
od2.update(c=3)
print(od2) # 顺序正常：OrderedDict([('a', 1), ('b', 2), ('c', 3)])


od3=OrderedDict()
od3.update(d=4,e=5,f=6)
print(od3) # 顺序错乱：OrderedDict([('e', 5), ('d', 4), ('f', 6)])
```

## **6、OrderedDict与sort结合**

 由于有序字典会记住其插入顺序，因此可以与排序结合使用以创建排序字典：

```text
>>>
>>> # 标准未排序的常规字典
>>> d = {'banana': 3, 'apple': 4, 'pear': 1, 'orange': 2}

>>> # 按照key排序的字典
>>> OrderedDict(sorted(d.items(), key=lambda t: t[0]))
OrderedDict([('apple', 4), ('banana', 3), ('orange', 2), ('pear', 1)])

>>> # 按照value排序的字典
>>> OrderedDict(sorted(d.items(), key=lambda t: t[1]))
OrderedDict([('pear', 1), ('orange', 2), ('banana', 3), ('apple', 4)])

>>> # 按照key的长度排序的字典
>>> OrderedDict(sorted(d.items(), key=lambda t: len(t[0])))
OrderedDict([('pear', 1), ('apple', 4), ('orange', 2), ('banana', 3)])
```

## **7、自定义OrderDict变体**

 我们通过继承OrderDict类来实现在原有的基础之上上定制化我们的子类（即OrderDict变体）。

 比如我们在用新条目覆盖现有条目时，我们不想像OrderDict原先那样保留原始的插入位置，而是将覆盖的条目移动到结尾，实现如下

```text
class LastUpdatedOrderedDict(OrderedDict):
    'Store items in the order the keys were last added'

    def __setitem__(self, key, value):
        if key in self:
            del self[key]
        OrderedDict.__setitem__(self, key, value)

od5=LastUpdatedOrderedDict()
od5['k1']=111
od5['k2']=222
od5['k3']=333
print(od5) # LastUpdatedOrderedDict([('k1', 111), ('k2', 222), ('k3', 333)])

od5['k2']=2222222222
print(od5) # 覆盖的值跑到末尾，LastUpdatedOrderedDict([('k1', 111), ('k3', 333), ('k2', 2222222222)])
```

\##8、OrderDict与collections.Counter结合

 有序字典可以与Counter类结合，以便计数器记住首次遇到的顺序元素：

```text
from collections import OrderedDict,Counter

class OrderedCounter(Counter, OrderedDict):
    'Counter that remembers the order elements are first encountered'
    def __repr__(self):
        print('====>')
        return '%s(%r)' % (self.__class__.__name__, OrderedDict(self))
    def __reduce__(self):
        return self.__class__, (OrderedDict(self),)


c1 = Counter(['bbb','ccc','aaa','aaa','ccc'])
print(c1)  # 顺序错乱：Counter({'ccc': 2, 'aaa': 2, 'bbb': 1})

c2=OrderedCounter(['bbb','ccc','aaa','aaa','ccc'])
print(c2)  # 顺序保持原有：OrderedCounter(OrderedDict([('bbb', 1), ('ccc', 2), ('aaa', 2)]))
```