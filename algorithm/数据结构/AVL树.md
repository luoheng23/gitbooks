# AVL树

AVL树是一种高度平衡的[二叉搜索树](二叉搜索树.md)。

对每一个结点$$x$$,$$x$$的左子树与右子树的高度差至多为1。要实现一颗AVL树，需要在每个结点内维护一个额外的属性：$$x.h$$为结点$$x$$的高度。

AVL树具有二叉搜索树的全部性质，插入和删除操作需要维持结点高度的平衡，与普通的二叉搜索树操作不同。

<!-- toc -->

### AVL树的实现

```go
type height int

// AVLNode of tree
type AVLNode struct {
	key         int
	h           height
	left, right *AVLNode
	p           *AVLNode
}

// AVLTree is AVL tree
type AVLTree struct {
	null *AVLNode
	root *AVLNode
}
```

### 建树

```go
// CreateAVLTree create a tree
func CreateAVLTree(A []int) AVLTree {
	r := AVLTree{
		null: &AVLNode{},
	}
	r.root = r.null
	for _, a := range A {
		r.Insert(&AVLNode{key: a})
	}
	return r
}
```

### 树的旋转

旋转的详细解释参见[旋转](红黑树.md)。

#### 左旋

```go
// O(lgn)
func (r *AVLTree) leftRotate(x *AVLNode) {
	y := x.right
	x.right = y.left
	if y.left != r.null {
		y.left.p = x
	}
	y.p = x.p
	if x.p == r.null {
		r.root = y
	} else if x == x.p.left {
		x.p.left = y
	} else {
		x.p.right = y
	}
	y.left = x
	x.p = y
	x.h = max(x.left.h, x.right.h) + 1
	y.h = max(y.left.h, y.right.h) + 1
}
```

#### 右旋

```go
func (r *AVLTree) rightRotate(x *AVLNode) {
	y := x.left
	x.left = y.right
	if y.right != r.null {
		y.left.p = x
	}
	y.p = x.p
	if x.p == r.null {
		r.root = y
	} else if x == x.p.left {
		x.p.left = y
	} else {
		x.p.right = y
	}
	y.right = x
	x.p = y
	x.h = max(x.left.h, x.right.h) + 1
	y.h = max(y.left.h, y.right.h) + 1
}
```

### 结点插入

```go
// Insert a AVLNode O(lgn)
func (r *AVLTree) Insert(z *AVLNode) {
	y, x := r.null, r.root
	for x != r.null {
		y = x
		if z.key < x.key {
			x = x.left
		} else {
			x = x.right
		}
	}
	z.p = y
	if y == r.null {
		r.root = z
	} else if z.key < y.key {
		y.left = z
	} else {
		y.right = z
	}
	z.left = r.null
	z.right = r.null
	z.h = 1
	r.balance(z.p)
}
```

### 性质维护

插入结点的所有祖先结点可能都会不平衡。所以需要全部调整一遍。5种情况

1. 祖先结点左子树与右子树是平衡的，直接更新祖先结点的高度。
2. 祖先结点左子树比右子树高2，分两种情况：
   1. 左子树的右子树高于左子树：左旋使左子树更高
   2. 左子树的左子树高于右子树，对祖先结点进行右旋
3. 祖先结点右子树比左子树高2，与情况2互为镜像

![AVLInsert](E:\luoheng\my writing\gitbooks\algorithm\AVLInsert.PNG)

```go
// O(lgn)
func (r *AVLTree) balance(z *AVLNode) {
	for z != r.null {
		if z.left.h > z.right.h+1 {
            // 情况2中的情况1,左旋变成情况2中的情况2
			if z.left.right.h > z.left.left.h {
				r.leftRotate(z.left)
			}
            // 情况2中的情况2，更新z结点的指向
			r.rightRotate(z)
			z = z.p
		} else if z.right.h > z.left.h+1 {
			if z.right.left.h > z.right.right.h {
				r.rightRotate(z.right)
			}
			r.leftRotate(z)
			z = z.p
		} else {
            // 情况1
			z.h = max(z.left.h, z.right.h) + 1
		}
        // 更新所有祖先结点
		z = z.p
	}
}
```

### 结点删除

```go
// Delete a AVLNode
func (r *AVLTree) Delete(z *AVLNode) {
	y, x := z, z
	if z.left == r.null {
		x = z.right
		r.transplant(z, z.right)
	} else if z.right == r.null {
		x = z.left
		r.transplant(z, z.left)
	} else {
		y = r.Min(z.right)
		x := y.right
		if y.p == z {
			x.p = y
		} else {
			r.transplant(y, y.right)
			y.right = z.right
			y.right.p = y
		}
		r.transplant(z, y)
		y.left = z.left
		y.left.p = y
	}
	r.balance(x.p)
}
```

### 性质维护

移动或删除的结点会影响高度，所以对其进行`balance`操作。