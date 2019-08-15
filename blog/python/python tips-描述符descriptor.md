描述符（descriptor）是实现了`__get__`、`__set__`、`__del__`方法的类，进一步可以细分为两类：

- 数据描述符：实现了`__get__`和`__set__`
- 非数据描述符：没有实现`__set__`

描述符在类的属性调用中起着很重要的作用，类在调用属性时，遵守两个规则：

- 按照实例属性、类属性的顺序选择属性，即实例属性优先于类属性
- 如果在类属性中发现同名的数据描述符，那么该描述符会优先于实例属性

### 非数据描述符会被实例属性覆盖

```python
class A:
    def __get__(self, obj, cls):
        return f"{obj}: get"

class B:
    value = A()

    def __init__(self):
        self.value = 4

def main():
    g = B()
    print(g.value)
    print(g.__dict__)

if __name__ == "__main__":
    main()
```

### 输出结果

```python
4
{'value': 4}
```

### 数据描述符优于实例属性

```python
class A:
    def __get__(self, obj, cls):
        return f"{obj}: get"

    def __set__(self, obj, value):
        print(f"{obj}: set, {value}")

class B:
    value = A()

    def __init__(self):
        self.value = 4

def main():
    g = B()
    print(g.value)
    print(g.__dict__)

if __name__ == "__main__":
    main()
```

### 输出结果

```python
<__main__.B object at 0x000001165EB85898>: set, 4
<__main__.B object at 0x000001165EB85898>: get
{}
```

从上述两个例子中可以看到，类`B`的`value`属性是一个描述符，当`value`属性是一个数据描述符时，它屏蔽了实例的同名属性`value`，实例对`value`属性的读取与赋值都会直接被转移到类属性`value`上。

### 使用描述符实现类的**静态方法**与**类方法**

```python
from functools import partial

class Staticmethod:

    def __init__(self, method):
        self.method = method

    def __get__(self, obj, cls):
        return self.method

class Classmethod:

    def __init__(self, method):
        self.method = method
    
    def __get__(self, obj, cls):
        return partial(self.method, cls)

class A:

    @Staticmethod
    def f(self):
        print(f"I'm method f, the value is {self}")
    
    @Classmethod
    def c(self):
        print(f"my class is {self}")

a = A()
a.f(23)
A.f(23)
a.c()
A.c()
```

### 输出结果

```python
I'm method f, the value is 23
I'm method f, the value is 23
my class is <class '__main__.A'>
my class is <class '__main__.A'>
```

**静态方法**与**类方法**统一了[类属性的两种引用方式](https://www.cnblogs.com/luoheng23/p/10989572.html)。这种统一的过程可以使用描述符修改属性访问的默认方式实现。**静态方法**限制实例的默认绑定，将方法当做普通函数使用；**类方法**始终将类作为第一个参数传入，上述的`partial`将类固定为方法的第一个参数。

## 总结

1. 描述符是实现了`__get__`、`__set__`、`__del__`等特殊方法的类，在属性访问时起着很大的作用。
2. 数据描述符会覆盖同名的实例属性，通过使用数据描述符，达到通过实例修改类变量的目的。
3. 描述符用于修改属性的默认访问方式，借此可以实现**类方法**与**静态方法**。