# B树

B树是为磁盘或其他直接存取的辅助存储设备而设计的一种平衡搜索[树](树.md)。

[TOC]

<!-- toc -->

## 定义

一颗B树是具有以下性质的有根树(根为$$T.root$$)：

1. 每个结点$$x$$有下面属性：
   1. $$x.n$$，当前存储在结点$$x$$中的关键字个数。
   2. $$x.n$$个关键字本身$$x.key_1,x.key_2,...,x.key_{x.n}$$，以非降序存放。
   3. $$x.leaf$$，一个$$bool$$值，如果$$x$$是叶节点，则为$$TRUE$$。
2. 每个内部结点$$x$$还包含$$x.n+1$$个指向孩子的指针$$x.c_1,x.c_2,...,x.c_{x.n+1}$$。叶节点没有孩子。
3. 关键字$$x.key_i$$对存储在各子树中的关键字范围进行分割，设$$k_i$$为任意存储在$$x.c_i$$为根的子树中的关键字，那么$$k_1\le x.key_1\le k_2\le ...\le x.key_{x.n}\le k_{x.n+1}$$
4. 每个叶结点具有相同的深度，即树的高度$$h$$。
5. 每个结点所包含的关键字个数有上界和下界，用一个被称为B树的最小度数的固定整数$$t\ge 2$$来表示这些界。
   1. 除了根节点以外的每个结点必须至少有$$t-1$$个关键字，因此除了根节点以外的每个内部结点至少有$$t$$个孩子，如果树非空，根节点至少有一个关键字。
   2. 每个结点至多可包含$$2t-1$$，因此一个内部结点至多$$2t$$个孩子，如果结点有$$2t-1$$个关键字，则称该结点是满的。

B树的操作需要访问硬盘，下面的实现中将B树整个保存在内存中，硬盘访问为空操作。

---

### 数据结构

```go
// BNode for BTree
type BNode struct {
	n    int      // number of keys
	keys []rune
	leaf bool     // leaf or not
	c    []*BNode // children (n+1)
}

// BTree is b tree
type BTree struct {
	root *BNode
    // B树的最小度数
	t    int
}
```

### 磁盘读取

```go
func diskRead(x *BNode, c *BNode) {}
func diskWrite(x *BNode)          {}
```

### 创建结点与树

```go
// CreateNode return a node
func (b *BTree) CreateNode() *BNode {
	return &BNode{
		n:    0,
        // 预分配最大的空间，以n来界定当前数组的上界
        // 这样很浪费空间，但比较好实现
		keys: make([]rune, 2*b.t-1),
		leaf: true,
		c:    make([]*BNode, 2*b.t),
	}
}

// CreateTree returns a BTree
func CreateTree(t int) BTree {
	b := BTree{}
	b.t = t
	b.root = b.CreateNode()
	return b
}
```

### 树的搜索

```go
// Search for key O(lgn)
func (b *BTree) Search(x *BNode, k rune) (*BNode, rune) {
	i := 0
	for i = 0; i < x.n && k > x.keys[i]; i++ {
	}
	if i < x.n && k == x.keys[i] {
		return x, x.keys[i]
	} else if x.leaf {
		return nil, 0
	}
    // key在子结点中，
	diskRead(x, x.c[i])
	return b.Search(x.c[i], k)
}
```

### 结点插入

由于每个结点的关键字个数最多为$$2t-1$$。所以如果往满结点插入关键字时，需要对结点进行分割。分割的方法是将当前的满结点切割成左$$t-1$$,右$$t-1$$以及中间的元素$$t$$。将左右变成新的结点，中间元素提至父节点用作两个新节点的分割元素。

#### 结点分割

```go
// x是父节点，要分割的结点是x.c[i]
func (b *BTree) splitChild(x *BNode, i int) {
	y := x.c[i]
    // z是y的右边t-1个关键字
	z := b.CreateNode()
	copy(z.keys, y.keys[b.t:y.n])
	if !y.leaf {
		copy(z.c, y.c[b.t:y.n+1])
	}
	z.n = b.t - 1
	y.n = b.t - 1
	x.n++
    // 把新建的z结点连上父节点
	copy(x.c[i+2:x.n+1], x.c[i+1:x.n])
	x.c[i+1] = z
	copy(x.keys[i+1:x.n], x.keys[i:x.n-1])
    // 将y.keys[y.n]提升至父节点
	x.keys[i] = y.keys[y.n]
	diskWrite(y)
	diskWrite(z)
	diskWrite(x)
}
```

但是如果父节点也是满的，这样做会导致父节点发生分割行为。所以为了防止这样不断向上回溯。在从根节点遍历下来的时候，将所有已满的结点提前分割，这样当插入新的关键字时，最多只用分割当前的结点，不用往父节点回溯。

#### 根节点是满的

根节点是满的需要单独考虑，因为根节点没有父节点，所以需要更改根节点的指向。

```go
// Insert a node
func (b *BTree) Insert(k rune) {
	r := b.root
	if r.n == 2*b.t-1 {
		s := b.CreateNode()
		b.root = s
		s.c[0] = r
		s.leaf = false
		b.splitChild(s, 0)
		b.insertNotFull(s, k)
	} else {
		b.insertNotFull(r, k)
	}
}
```

#### 其他结点的插入

```go
func (b *BTree) insertNotFull(x *BNode, k rune) {
	i := x.n - 1
	if x.leaf {
        // 叶结点，找到合适的位置插入即可
		for i >= 0 && k < x.keys[i] {
			x.keys[i+1] = x.keys[i]
			i--
		}
		x.keys[i+1] = k
		x.n++
		diskWrite(x)
	} else {
        // 非叶结点，找到k的具体位置
		for i >= 0 && k < x.keys[i] {
			i--
		}
		i++
		diskRead(x, x.c[i])
        // 如果满了，就分割
		if x.c[i].n == 2*b.t-1 {
			b.splitChild(x, i)
            // 确定k的详细位置
			if k > x.keys[i] {
				i++
			}
		}
        // 递归插入，新节点一定是插入到叶结点上
		b.insertNotFull(x.c[i], k)
	}
}

```

### 结点删除

为了使删除后，B树的性质不被破坏，我们设计的删除过程作用的结点保证，无论何时，结点$$x$$递归调用自身时，$$x$$中关键字个数至少为最小度数$$t$$。这样删除关键字之后，$$x$$中的关键字个数至少为$$t-1$$，符合性质。

#### 结点删除的种种情况：

1. 如果关键字$$k$$在结点$$x$$中，并且$$x$$是叶结点，直接删除$$k$$
2. 如果关键字$$k$$在结点$$x$$中，并且$$x$$是内部结点，则：
   1. 如果在$$k$$前面的子结点至少包含$$t$$个关键字，则找出$$k$$的前驱替代$$k$$，然后递归删除子结点中的$$k$$的前驱。
   2. 对称的，如果$$k$$后面的子结点至少包含$$t$$个关键字，则找后继，并删后继。
   3. 如果前后子结点都少于$$t$$，则将$$k$$，后面的子结点全都并入前面的子结点中，然后删除$$x$$中的$$k$$以及k后面的子结点。递归删除k前面子结点中的$$k$$。
3. 如果关键字不在结点$$x$$中，则存在某个$$i$$使$$k$$在$$x.c_i$$中（默认结点存在）。如果$$x.c_i$$有$$t$$个关键字，则递归删除$$k$$即可，如果$$x.c_i$$只有$$t-1$$个关键字，则：
   1. 如果相邻兄弟有$$t$$个关键字，则将$$x$$中的某个关键字降至$$x.c_i$$，将兄弟的一个关键字升至$$x$$。将该兄弟对应的孩子指针一直$$x.c_i$$，这样$$x.c_i$$增加了一个关键字。
   2. 如果相邻兄弟有$$t-1$$个关键字，则如$$2.3$$一样，合并这些结点，然后递归删除。

```go
// Delete a node
func (b *BTree) Delete(x *BNode, k rune){
	i := 0
    // 找到k的位置
	for ; i < x.n; i++ {
		if x.keys[i] >= k {
			break
		}
	}
    // 如果k在x中
	if i < x.n && x.keys[i] == k {
        // 情况1，k在叶结点中，直接删除
		if x.leaf {
			copy(x.keys[i:], x.keys[i+1:])
			x.n--
        // 情况2.1，找前驱
		} else if x.c[i].n >= b.t {
			x.keys[i] = x.c[i].keys[x.c[i].n-1]
			b.Delete(x.c[i], x.keys[i])
		} else if x.c[i+1].n >= b.t {
			x.keys[i] = x.c[i+1].keys[0]
			b.Delete(x.c[i+1], x.keys[i])
		} else {
			x.c[i].keys[x.c[i].n] = x.keys[i]
			x.c[i].n++
			copy(x.c[i].keys[x.c[i].n:], x.c[i+1].keys[:x.c[i+1].n])
			x.c[i].n += x.c[i+1].n
			copy(x.c[i].c[x.n+1:], x.c[i+1].c[:x.c[i+1].n+1])
			copy(x.keys[i:], x.keys[i+1:])
			copy(x.c[i+1:], x.c[i+2:])
			x.n--
			b.Delete(x.c[i], k)
		}
	} else if i < x.n && x.keys[i] > k && !x.leaf {
		if x.c[i].n < b.t {
			x.c[i].keys[x.c[i].n] = x.keys[i]
			x.c[i].n++
			if x.c[i+1].n >= b.t {
				x.keys[i] = x.c[i+1].keys[0]
				x.c[i].c[x.c[i].n++] = x.c[i+1].c[0] 
				b.Delete(x.c[i+1], x.keys[i])
				b.Delete(x.c[i], k)
			} else {
				copy(x.c[i].keys[x.c[i].n:], x.c[i+1].keys[:x.c[i+1].n])
				x.c[i].n += x.c[i+1].n
				copy(x.c[i].c[x.n+1:], x.c[i+1].c[:x.c[i+1].n+1])
				copy(x.keys[i:], x.keys[i+1:])
				copy(x.c[i+1:], x.c[i+2:])
				x.n--
			}
		}
		b.Delete(x.c[i], k)
	}
	if x.n == 0 && x == b.root {
		b.root = x.c[0]
	}
}
```

