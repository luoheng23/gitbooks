# map

## 定义

```go
var ages map[string]int // map[K]V
ages := make(map[string]int)
ages := map[string]int{
    "alice": 31,
    "charlie": 34,
}
```

## 遍历

```go
for name, age := range ages {
}
```

## 长度

```go
len(ages)
```

## 键值是否存在

```go
if age, ok := ages["bob"]; !ok {
}
```

## 删除键值

```go
delete(ages, "alice")
```

