## 哈希与集合

python中的哈希叫做dict，集合叫做set。

哈希的键和集合的元素都必须是可hash的。

内建类型中，可变的类型都是不可hash的，如列表，dict和set都是不可hash的，不能作为哈希的键或集合的元素。数字、字符串以及布尔类型都是可hash的。当元组的内部元素都不可变时，它是可hash的。

自定义对象时，必须实现`__hash__`和`__eq__`两个魔法方法才能作为哈希的键或集合的元素。

## 哈希操作

### 长度

```python
>>> t1 = {1:2, 2:3}
>>> t1
{1: 2, 2: 3}
>>> len(t1)
2
>>>
```

### 赋值与删值

```python
>>> t1 = {1:2, 2:3}
>>> t1["hello"] = 4  # 当键存在时，更新该值
>>> t1
{1: 2, 2: 3, 'hello': 4}
>>> del t1[1]       # 删除值
>>> t1
{2: 3, 'hello': 4}
>>> t1.clear()     # 清空dict
```

### 查找

```python
>>> t1 = {1: 2, 2: 3, 'hello': 4}
>>> t1[5]        # 访问不存在的键会抛出异常
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 5
>>> 5 in t1      # 使用in操作符判断存在性
False
>>>
```

### 遍历

```python
>>> t1 = {1: 2, 2: 3, 'hello': 4}
>>> for key in t1:            # 默认对键进行迭代
...     print(key, t1[key])
...
1 2
2 3
hello 4
>>> for value in t1.values():    # 使用values迭代值
...     print(value)
...
2
3
4
>>> for key, value in t1.items(): # 使用items迭代键和值
...     print(key, value)
...
1 2
2 3
hello 4
>>>
```

## 集合操作

集合的四种操作：交集，并集，差集，对称差。这四种操作对应着四个运算符以及四种方法。

### 赋值与删值

```python
>>> t1 = {1, 2, 3}
>>> t1.pop()     # 随机弹出一个元素
1
>>> t1
{2, 3}
>>> t1.add(1)    # 添加一个元素
>>> t1
{1, 2, 3}
>>> t1.pop()
2
>>> t1
{1, 3}
>>> t1.remove(3)  # 删除特定元素
>>> t1
{1}
>>> t1.clear()   # 清空集合
>>>
```

### 交集

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1 & t2       # t1 &= t2 直接更新t1
{2, 3}
>>> t1.intersection(t2)  # t1.intersection_update(t2)
{2, 3}
>>>
```

### 并集

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1 | t2          # t1 |= t2
{1, 2, 3, 4}
>>> t1.union(t2)     # t1.update(t2) 默认的update
{1, 2, 3, 4}
>>>
```

### 差集

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1 - t2            # t1 -= t2
{1}
>>> t1.difference(t2)  # t1.difference_update(t2)
{1}
>>>
```

## 对称差

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1 ^ t2         # t1 ^= t2
{1, 4}
>>> t1.symmetric_difference(t2) # t1.symmetric_difference_update(t2)
{1, 4}
>>>
```

### 不相交集合

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1.isdisjoint(t2)
False
>>>
```

### 子集与超集

```python
>>> t1 = {1, 2, 3}
>>> t2 = {2, 3, 4}
>>> t1.issubset(t2)
False
>>> t1.issuperset(t2)
False
>>>
```

