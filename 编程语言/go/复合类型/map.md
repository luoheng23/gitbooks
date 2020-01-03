## 哈希

哈希的键必须可以比较。

```
var ages map[string]int // map[K]V
ages := make(map[string]int)
ages := map[string]int{
    "alice": 31,
    "charlie": 34,
}
```

## 哈希操作

### 长度

```go
len(ages)
```

### 赋值

```go
ages["alice"] = 45  // 如果键不存在则添加键值，如果键存在则更新值
```

### 查找

```go
age, ok := ages["alice"]
if ok {
    // alice 存在，age有值
}
```

### 遍历

```go
for name, age := range ages {
    // 按键值对遍历
}
```

### 删除键值

```go
delete(ages, "alice")
```