在Python中，有两种情况下可以省略括号。
1. 将生成器作为函数的唯一参数
2. 元组作为字典的键

### 示例如下

```python
# 正常版本
s = sum((i for i in range(10)))
# 省略括号
s = sum(i for i in range(10))
# 正常版本
s = "".join((i for i in "hello world"))
# 省略括号
s = "".join(i for i in "hello world")
# 字典
s = {(1, 2, 3): "hello world"}
print(s[(1, 2, 3)], s[1, 2, 3])
```

### 输出结果
```python
hello world hello world
```
