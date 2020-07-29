# 数据结构与算法

## 数组

在计算机科学中，数组数据结构（英语：array data structure），简称数组（英语：Array），是由存储在一块连续的内存中的相同类型的元素（element）的集合所组成的数据结构。

在 JavaScript 里， 也可以在数组中保存不同类型的值。但最好我们还是遵守最佳实践。

## 栈

栈是一种遵从后进先出（LIFO）原则的有序集合。

新添加的或待删除的元素都保存在栈的同一端，称作栈顶，另一端则叫栈底。

```javascript
/**
 * 栈的实现：
 * push(element(s))：添加一个（或几个）新元素到栈顶，也称为压栈。
 * pop()：移除栈顶的元素，同时返回被移除的元素，也称为出栈。
 * peek()：返回栈顶的元素。
 * size()：返回栈里的元素个数。
 * isEmpty()：栈里没有任何元素就返回 true，否则返回 false。
 * clear()：移除栈里的所有元素。
 */
class Stack {
    constructor() {
        this._list = Symbol('_list')
        this[this._list] = []
    }

    push(...items) {
        this[this._list].push(...items)
    }

    pop() {
        return this[this._list].pop()
    }

    peek() {
        return this[this._list][this[this._list].length - 1]
    }

    size() {
        return this[this._list].length
    }

    isEmpty() {
        return this[this._list].length === 0
    }

    clear() {
        this[this._list].length = 0
    }
}
```

```javascript
/**
 * 利用栈对十进制数字进行任意进制转换
 * @param {number} num 被转换的数
 * @param {number} base 基数
 */
function _parseInt(num = 0, base = 10) {
    const stack = new Stack()
    const digits = '0123456789ABCDEF'
    let ret = ''
    while (num > 0) {
        stack.push(Math.floor(num % base))
        num = Math.floor(num / base)
    }
    while (!stack.isEmpty()) {
        ret += digits[stack.pop()]
    }
    return ret || 0
}
```

## 队列

队列是遵循 FIFO（First In First Out）原则的一组有序的项。 队列在尾部添加新元素，并从顶部移除元素。最新添加的元素必须排在队列的末尾。

```javascript
/**
 * 队列的实现：
 * enqueue(item(s))：向队列尾部添加一个（或多个）新的项。
 * dequeue()：移除队列的第一（即排在队列最前面的）项，并返回被移除的元素。
 * front()：返回队列中第一个元素。
 * isEmpty()：如果队列中不包含任何元素，返回 true，否则返回 false。
 * size()：返回队列包含的元素个数。
 */
class Queue {
    constructor() {
        this._list = Symbol('_queue')
        this[this._list] = []
    }

    enqueue(...items) {
        this[this._list].push(...items)
    }

    dequeue() {
        return this[this._list].shift()
    }

    front() {
        return this[this._list][0]
    }

    size() {
        return this[this._list].length
    }

    isEmpty() {
        return this[this._list].length === 0
    }
}
```

### 优先队列

在优先队列中，元素的添加和移除是基于优先级的。

优先队列可以简单的分为最小优先队列，其优先级的值较小的元素被放置在队列最前面（壹代表更高的优先级）；最大优先队列则与之相反，把优先级的值较大的元素放置在队列最前面。

实现一个优先队列，有两种选项：设置优先级，然后在正确的位置添加元素；或者用入列操作添加元素，然后按照优先级移除它们。这里我们使用前面一种方式，其它的方法都不用改变，现在来修改一下入列方法。

```javascript
// ...
enqueue(item, priority) {
    const oItem = {
        element: item,
        priority
    }
    const maxIndex = this.size() - 1
    if (maxIndex < 0) {
        this[this._list].push(oItem)
    } else {
        this[this._list].some((_oItem, index) => {
            if (oItem.priority < _oItem.priority) {
                this[this._list].splice(index, 0, oItem)
                return true
            } else if (index < maxIndex) {
                return false
            } else {
                this[this._list].push(oItem)
            }
        })
    }
}
// ...
```

### 循环队列

循环队列队尾被连接在队首之后以形成了一个循环。

使用循环队列的一个例子就是用来计算击鼓传花游戏 （Hot Potato） 。在这个游戏中，几个人围成一个圆圈，然后依次把花尽快地传递给旁边的人。某一时刻停止， 此时花在谁手里，谁就退出圆圈结束游戏。接着继续重复这个过程，最后剩下的一个人就是胜利者。

```javascript
function hotPotato(list, num) {
    const queue = new Queue() // 这里使用的是普通队列
    list.forEach(item => queue.enqueue(item))
    while (queue.size() > 1) {
        for (let i = 0; i < num; i++) {
            queue.enqueue(queue.dequeue())
        }
        console.log(`${queue.dequeue()}在击鼓传花游戏中被淘汰了。`)
    }
    console.log(`${queue.dequeue()}在击鼓传花游戏中获得了胜利。`)
}

const list = ['小明', '小花', '小华', '小军', '小李']
hotPotato(list, 8)

// 小军在击鼓传花游戏中被淘汰了。
// 小李在击鼓传花游戏中被淘汰了。
// 小华在击鼓传花游戏中被淘汰了。
// 小明在击鼓传花游戏中被淘汰了。
// 小花在击鼓传花游戏中获得了胜利。
```

## 链表

链表存储有序的元素集合，但不同于数组，链表中的元素在内存中并不是连续放置的。每个元素由一个存储元素本身的节点和一个指向下一个元素的引用（也称指针或链接）组成。

```javascript
/**
 * 辅助构造函数
 * @param value 节点的值
 */
function Node(value) {
    this.value = value
    this.next = null
}

/**
 * 链表的实现：
 * append(value)：向列表尾部添加一个新项。
 * add(value)：向列表的头部添加新项。
 * insert(index, value)：向列表的指定位置插入一个新项。
 * remove(value)：从列表中移除一项。
 * indexOf(value)：返回元素在列表中的索引。如果列表中没有该元素则返回 -1。
 * isEmpty()： 如果链表中不包含任何元素， 返回 true， 如果链表长度大于 0 则返回 false。
 * size()：返回链表包含的元素个数。
 * travel()：遍历整个链表。
 */
class LinkedList {
    constructor(value = null) {
        if (value !== null) {
            this._head = new Node(value)
        } else {
            this._head = null
        }
    }

    add(value) {
        const node = new Node(value)
        node.next = this._head
        this._head = node
    }

    append(value) {
        let current = this._head
        const node = new Node(value)
        if (this.isEmpty()) {
            this._head = node
        } else {
            while (current.next) {
                current = current.next
            }
            current.next = node
        }
    }

    insert(index, value) {
        let curIndex = 0
        let prevNode = this._head
        const targetIndex = index - 1
        const node = new Node(value)
        if (index <= 0) {
            return this.add(value)
        }
        if (index > this.size() - 1) {
            return this.append(value)
        }
        while (curIndex < targetIndex) { // 找到插入位置的前一个节点
            pos++
            prevNode = prevNode.next
        }
        node.next = prevNode.next
        prevNode.next = node
    }

    isEmpty() {
        return this._head === null
    }

    size() {
        let count = 0
        let current = this._head
        while (current) {
            count++
            current = current.next
        }
        return count
    }

    travel() {
        let current = this._head
        while (current) {
            console.log(current.value)
            current = current.next
        }
    }

    search(value) {
        let index = -1
        let current = this._head
        while (current) {
            index++
            if (current.value === value) {
                return index
            }
            current = current.next
        }
        return -1
    }

    remove(value) {
        let prevNode = null
        let current = this._head
        while (current) {
            if (current.value === value) {
                if (prevNode) {
                    prevNode.next = current.next
                } else {
                    // 如果找的节点是头节点，直接修改头节点的指向
                    this._head = current.next
                }
                break
            } else {
                prevNode = current
                current = current.next
            }
        }
    }
}
```

在实现上，也可以为该类添加一个私有属性（_length）来记录当前列表中元素的个数；后续在添加或移除元素时对其进行更改，这样在获取列表中元素的个数时，就不需要在遍历列表了，而是直接返回 `_length` 属性的值。

### 双向链表

双向链表和普通链表的区别在于，在链表中，一个节点只有链向下一个节点的链接，而在双向链表中，链接是双向的：一个链向下一个元素， 另一个链向前一个元素。

```javascript
// 下面列举了双向链表不同的实现部分

/**
 * 辅助构造函数
 * @param value 节点的值
 */
function Node(value) {
    this.value = value
    this.next = null
    this.prev = null
}

// ...
constructor(value = null) {
    if (value !== null) {
        this._head = new Node(value)
    } else {
        this._head = null
    }
}

add(value) {
    const node = new Node(value)
    if (this.isEmpty()) {
        return this._head = node
    }
    node.next = this._head
    this._head.prev = node
    this._head = node
}

append(value) {
    let current = this._head
    const node = new Node(value)
    if (this.isEmpty()) {
        this._head = node
    } else {
        while (current.next) {
            current = current.next
        }
        current.next = node
        node.prev = current
    }
}

insert(index, value) {
    let curIndex = 0
    let current = this._head
    const targetIndex = index - 1
    const node = new Node(value)
    if (index <= 0) {
        return this.add(value)
    }
    if (index > this.size() - 1) {
        return this.append(value)
    }
    while (curIndex < targetIndex) { // 找到插入位置的前一个节点
        pos++
        current = current.next
    }
    node.prev = current
    node.next = current.next
    current.next.prev = node
    current.next = node
}

remove(value) {
    let current = this._head
    while (current) {
        if (current.value === value) {
            if (current === this._head) { // 如果找的节点是头节点
                this._head = current.next
                if (current.next) { // 如果只有一个节点，经过上一步的操作后 this._head 为 null 所以改变前驱前要做一次判断排除
                    current.next.prev = null
                }
            } else {
                current.prev.next = current.next
                if (current.next) { // 如果是最后一个节点，经过上一步操作后 current.next 为 null，所以需要进行判断排除
                    current.next.prev = current.prev
                }
            }
            break
        } else {
            current = current.next
        }
    }
}
// ...
```

### 循环链表

循环链表可以像链表一样只有单向引用，也可以像双向链表一样有双向引用。

循环链表和单向链表之间唯一的区别在于，最后一个元素指向下一个元素的指针不是引用 null， 而是指向第一个元素。而双向循环链表则是头部元素的前驱指向尾元素，尾部元素的后继指向头部元素。

## 集合

集合是由一组无序且唯一的项组成的。

```javascript
/**
 * add(value)：向集合添加一个新的项。
 * delete(value)：从集合移除一个值。
 * has(value)：如果值在集合中，返回true，否则返回false。
 * clear()：移除集合中的所有项。
 * size()：返回集合所包含元素的数量。与数组的length属性类似。
 * values()：返回一个包含集合中所有值的数组。
 */
function Set() {}
```

`ECMAScript 6` 提供了新的数据结构 Set，支持以上的所有需求，这里就不多做实现了，关于 Set 更过内容可以查看阮老师的 [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/set-map#Set)。

```javascript
let setA = new Set();
setA.add(1);
setA.add(2);
setA.add(3);
let setB = new Set();
setB.add(3);
setB.add(4);
setB.add(5);
setB.add(6);
```

并集：对于给定的两个集合，返回一个包含两个集合中所有元素的新集合。

```javascript
Set.prototype.union = function(target) {
    return new Set([...this].concat([...target]))
}

console.log(setA.union(setB)) // Set(6) {1, 2, 3, 4, 5, 6}
```

交集：对于给定的两个集合，返回一个包含两个集合中共有元素的新集合。

```javascript
Set.prototype.intersection = function (target) {
    return new Set([...target].filter(item => this.has(item)))
}

console.log(setA.intersection(setB)) // Set(1) {3}
```

差集：对于给定的两个集合，返回一个包含所有存在于第一个集合且不存在于第二个集合的元素的新集合。

```javascript
Set.prototype.difference = function (target) {
    return new Set([...this].filter(item => !target.has(item)))
}

console.log(setA.difference(setB)) // Set(2) {1, 2}
```

子集：验证一个给定集合是否是另一集合的子集。

```javascript
Set.prototype.subset = function (target) {
    return [...this].every(item => target.has(item))
}

console.log(setA.subset(setB)) // false
```

## 字典

字典是跟集合很相似的一种数据结构，都可以用来存储无序不重复的数据。不同的地方是集合以 `[值，值]` 的形式存储，而字典则是以 `[键，值]` 的形式存储。字典也被称作映射。

```javascript
/**
 * 字典的主要方法：
 * set(key,value)：向字典中添加新元素。
 * delete(key)：通过使用键值来从字典中移除键值对应的数据值。
 * has(key)：如果某个键值存在于这个字典中，则返回true，反之则返回false。
 * get(key)：通过键值查找特定的数值并返回。
 * clear()：将这个字典中的所有元素全部删除。
 * size()：返回字典所包含元素的数量。与数组的length属性类似。
 * keys()：将字典所包含的所有键名以数组形式返回。
 * values()：将字典所包含的所有数值以数组形式返回。
 */
function Dictionary() {}
```

`ECMAScript 6` 提供了新的数据结构 Map，支持以上的所有需求，这里就不多做实现了，关于 Map 更过内容可以查看阮老师的 [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/set-map#Set)。

### 散列表

散列表（HashTable）是字典类的一种散列表实现方式，散列算法的作用是尽可能快地在数据结构中找到一个值。

```javascript
/**
 * 散列函数:
 * 求得给定 key 的每个字符的 ASCII 码值的和得到一个数字；
 * 然后与一个任意数做除法取余以得到一个更小的数。
 */
function loseloseHashCode(str) {
    return [...str].reduce((total, cur, index) => total + str.codePointAt(index), 0) % 37
}

/**
 * 散列表的基本实现：
 * put(key,value)：向散列表增加一个新的项（也能更新散列表）。
 * remove(key)：根据键值从散列表中移除值。
 * get(key)：返回根据键值检索到的特定的值。
 */
function HashTable() {
    this._table = []
}

HashTable.prototype.put = function (key, value) {
    this._table[loseloseHashCode(key)] = value
}

HashTable.prototype.get = function (key) {
    return this._table[loseloseHashCode(key)]
}

/**
 * 注意删除时不能将位置本身从数组中移除（这会改变其他元素的位置）；
 * 否则，当下次需要获得或移除一个元素的时候，这个元素会不在我们用散列函数求出的位置上。
 */
HashTable.prototype.remove = function (key) {
    this._table[loseloseHashCode(key)] = undefined
}
```

现在有一个明显的问题，我们根据函数求得的值前后很可能会重复，所以容易对原本位置上的值进行覆盖，如果不加处理将会导致整个表遭到破坏而无法使用。处理这种冲突的方法包括：分离链接、线性探查和双散列法。接下来我们采用线性探查的方法来解决冲突。

```javascript
/**
 * 辅助构造函数
 */
function ValueFactory(key, value) {
    this.key = key
    this.value = value
}

/**
 * 散列表的基本实现：
 * put(key,value)：向散列表增加一个新的项（也能更新散列表）。
 * remove(key)：根据键值从散列表中移除值。
 * get(key)：返回根据键值检索到的特定的值。
 */
function HashTable() {
    this._table = []
}

HashTable.prototype.put = function (key, value) {
    let pos = loseloseHashCode(key)
    while (this._table[pos]) {
        pos++
    }
    this._table[pos] = new ValueFactory(key, value)
}

HashTable.prototype.get = function (key) {
    let cur = null
    let pos = loseloseHashCode(key)
    const len = this._table.length
    while (pos < len) {
        cur = this._table[pos]
        if (cur && cur.key === key) {
            return this._table[pos]
        }
        pos++
    }
}

HashTable.prototype.remove = function (key) {
    let cur = null
    let pos = loseloseHashCode(key)
    const len = this._table.length
    while (pos < len) {
        cur = this._table[pos]
        if (cur && cur.key === key) {
            this._table[pos] = undefined
        }
        pos++
    }
}
```

可见，解决冲突会带来一些复杂的工作，当数据量过大时对性能的损耗也不小，所以最好使用更加优秀的散列函数以避免冲突。下面时社区推崇的一种散列函数。

```javascript
var djb2HashCode = function (key) {
    var hash = 5381 // 初始化一个 hash 变量并赋值为一个质数
    for (var i = 0; i < key.length; i++) { // 迭代参数 key
        // 将 hash 与 33 相乘（用来当作一个魔力数），并和当前迭代到的字符的 ASCII 码值相加
        hash = hash * 33 + key.charCodeAt(i)
    }
    return hash % 1013 // 将相加的和与另一个随机质数（比我们认为的散列表的大小要大）相除的取余
}
```

和散列表相类似的还有散列集合的实现。散列集合由一个集合构成，但是插入、 移除或获取元素时，使用的是散列函数。不同之处在于，不再添加键值对，而是只插入值而没有键。

## 树

树是一种分层数据的抽象模型。

- 位于树顶部的节点叫作根节点。
- 没有子元素的节点称为外部节点或叶子节点。
- 节点的深度取决于它的祖先节点的数量。
- 树的高度取决于所有节点深度的最大值。

二叉树中的节点最多只能有两个子节点：一个是左侧子节点，另一个是右侧子节点。

```javascript
/**
 * 创建节点的辅助函数
 */
function Node(value) {
    this.value = value
    this.left = null
    this.right = null
}

/**
 * 二叉树的实现：
 * insert(key)：向树中插入一个新的键。
 * remove(key)：从树中移除某个键。
 */
class BinaryTree {
    constructor(value) {
        if (value != null) {
            this._root = new Node(value)
        } else {
            this._root = null
        }
    }

    insert(value) {
        const node = new Node(value)
        const queue = new Queue()
        queue.enqueue(this._root)
        if (this._root === null) {
            return this._root = node
        }
        while (queue.size()) {
            const current = queue.dequeue()
            if (current.left) {
                queue.enqueue(current.left)
            } else {
                current.left = node
                break
            }
            if (current.right) {
                queue.enqueue(current.right)
            } else {
                current.right = node
                break
            }
        }
    }
}
```

二叉搜索树（BST）是二叉树的一种，但是它只允许你在左侧节点存储（比父节点）小的值， 在右侧节点存储（比父节点）大（或者等于）的值。

```javascript
/**
 * 创建节点的辅助函数
 */
function Node(value) {
    this.value = value
    this.left = null
    this.right = null
}

/**
 * 二叉查找树的实现：
 * insert(key)：向树中插入一个新的键。
 * inOrderTraverse：通过中序遍历方式遍历所有节点。
 * preOrderTraverse：通过先序遍历方式遍历所有节点。
 * postOrderTraverse：通过后序遍历方式遍历所有节点。
 * min：返回树中最小的值/键。
 * max：返回树中最大的值/键。
 * search(key)：在树中查找一个键，如果节点存在，则返回 true；否则返回 false。
 * remove(key)：从树中移除某个键。
 */
class BinarySearchTree {
    constructor(value) {
        if (value !== null && value !== undefined) {
            this._root = new Node(value)
        } else {
            this._root = null
        }
    }

    insert(value) {
        const node = new Node(value)
        if (this._root === null) {
            this._root = node
        } else {
            this._inertNode(this._root, node)
        }
    }

    _inertNode(curNode, newNode) {
        if (newNode.value < curNode.value) {
            if (curNode.left === null) {
                curNode.left = newNode
            } else {
                this._inertNode(curNode.left, newNode)
            }
        } else {
            if (curNode.right === null) {
                curNode.right = newNode
            } else {
                this._inertNode(curNode.right, newNode)
            }
        }
    }

    breadthFirstTraverse() {
        if (this._root === null) return
        let curNode = null
        const queue = [this._root]
        while (queue.length) {
            curNode = queue.pop()
            console.log(curNode.value)
            if (curNode.left) {
                queue.push(curNode.left)
            }
            if (curNode.right) {
                queue.push(curNode.right)
            }
        }
    }

    preOrderTraverse(node = this._root) {
        if (node === null) return
        console.log(node.value)
        if (node.left) {
            this.preOrderTraverse(node.left)
        }
        if (node.right) {
            this.preOrderTraverse(node.right)
        }
    }

    inOrderTraverse(node = this._root) {
        if (node === null) return
        if (node.left) {
            this.inOrderTraverse(node.left)
        }
        console.log(node.value)
        if (node.right) {
            this.inOrderTraverse(node.right)
        }
    }

    postOrderTraverse(node = this._root) {
        if (node === null) return
        if (node.left) {
            this.postOrderTraverse(node.left)
        }
        if (node.right) {
            this.postOrderTraverse(node.right)
        }
        console.log(node.value)
    }

    min(node = this._root) {
        if (node !== null) {
            while (node && node.left) {
                node = node.left
            }
            return node.value
        }
    }

    max(node = this._root) {
        if (node !== null) {
            while (node && node.right) {
                node = node.right
            }
            return node.value
        }
    }

    search(value) {
        return this._searchNode(this._root, value)
    }

    _searchNode(node, value) {
        if (node === null) return false
        if (value < node.value) {
            return this._searchNode(node.left, value)
        } else if (value > node.value) {
            return this._searchNode(node.right, value)
        } else {
            return true
        }
    }

    remove(value) {
        this._root = this._removeNode(this._root, value)
        // 移除的是根节点的情况？？？
    }

    _removeNode(node, value) {
        let min = 0
        if (node === null) return null
        if (value < node.value) {
            node.left = this._removeNode(node.left, value)
            return node
        } else if (value > node.value) {
            this._removeNode(node.right, value)
            return node
        } else {
            if (node.left === null && node.right === null) {
                node = null // 赋予 null 值来移除该节点
                return null // 返回 null 来将对应的父节点指针赋予 null 值
            } else if (node.left === null) {
                node = node.right
                return node
            } else if (node.right === null) {
                node = node.left
                return node
            } else {
                min = this.min(node.right) // 找出该节点右侧最小的节点
                node.value = min
                this._removeNode(node.right, min)
            }
        }
    }
}
```

### [自平衡二叉搜索树](https://zhuanlan.zhihu.com/p/56066942)

自平衡二叉搜索树（AVL）任何一个节点左右两侧子树的高度之差最多为 1，它会在添加或移除节点时尽量试着成为一棵完全树。

平衡二叉树中不存在平衡因子大于 1 的节点。在一棵平衡二叉树中，节点的平衡因子只能取 0 、1 或者 -1 ，分别对应着左右子树等高，左子树比较高，右子树比较高。

在自平衡二叉搜索树插入或移除节点和二叉搜索树基本相同，不同的是我们在过程中需要检验它的平衡因子（Balance Factor，某节点的左子树与右子树的高度差即为该节点的平衡因子），如果有需要，则将其逻辑应用于树的自平衡。

## 图

图是网络结构的抽象模型。图是一组由边连接的节点（或顶点）。

一条边两端的顶点称为称为相邻顶点。

一个顶点的度是其相邻顶点的数量。

简单路径要求不包含重复的顶点。

如果图中不存在环，则称该图是无环的。

如果图中每两个顶点间都存在路径，则该图是连通的。

如果图中每两个顶点间在双向上都存在路径，则该图是强连通的。

### 实现

图最常见的实现是邻接矩阵。每个节点都和一个整数相关联，该整数将作为数组的索引。我们用一个二维数组来表示顶点之间的连接。如果索引为 `i` 的节点和索引为 `j` 的节点相邻，则 `array[i][j] === 1`，否则 `array[i][j] === 0`。

邻接矩阵在描述非强连通的图会记录许多的零（即使是不存在的边 A-A），而且也不灵活。

另外，我们还可以用关联矩阵来表示图。在关联矩阵中，矩阵的行表示顶点，列表示边。我们使用二维数组来表示两者之间的连通性，如果顶点 `v` 是边 `e` 的入射点，则 `array[v][e] === 1`； 否则，`array[v][e] === 0`。

关联矩阵通常用于边的数量比顶点多的情况下，以节省空间和内存。

当然，我们也可以使用一种叫作邻接表的动态数据结构来表示图，邻接表由图中每个顶点的相邻顶点列表所组成，就像下面这样。

```javascript
class Graph {
    constructor() {
        this._vertices = []
        this._adjList = {} // 本应是字典结构，这里直接用对象充当
    }

    /**
     * 将顶点添加到顶点列表中，并且在邻接表中，设置顶点 v 作为键对应的字典值为一个空数组
     * @param {*} v 顶点
     */
    addVertex(v) {
        this._vertices.push(v)
        this._adjList[v] = []
    }

    /**
     * 通过将 w 加入到 v 的邻接表中，我们添加了一条自顶点 v 到顶点 w 的边
     * @param {*} v 顶点
     * @param {*} w 顶点
     */
    addEdge(v, w) {
        this._adjList[v].push(w)
        this._adjList[w].push(v)
    }

    /**
     * 打印图目前的结构
     */
    toString() {
        let s = '',
            len = this._vertices.length,
            len2 = 0,
            i = 0,
            j = 0,
            neighbors = null
        for (; i < len; i++) {
            s += this._vertices[i] + ' -> '
            neighbors = this._adjList[this._vertices[i]]
            len2 = neighbors.length
            for (j = 0; j < len2; j++) {
                s += neighbors[j] + ' '
            }
            s += '\n'
        }
        return s
    }
}
```

图遍历可以用来寻找特定的顶点或寻找两个顶点之间的路径，检查图是否连通，检查图是否含有环等。下面来看一下采用广度优先遍历图的实现。

```javascript
class Graph {
    // ...
    bfs(v, cb = () => {}) {
        const visitedSet = new Set()
        const queue = [v] // 本应是一个队列，这里用数组代替

        while (queue.length) {
            if (!visitedSet.has(v)) {
                cb(v)
                queue.push(...this._adjList[v])
                visitedSet.add(v)
            }
            queue.shift()
            v = queue[0]
        }
    }

    /**
     * 给定一个非加权的图 G 和源顶点 v，找出对每个顶点 u，u 和 v 之间最短路径的距离（以边的数量计）
     * 使用广度优先遍历时，由于是按层次遍历的，所以以边计算距离的话，刚好等于层数（取决于你的初始值），所以在遍历时只要记住当前的层数即可
     * @param {*} v 起始节点
     */
    bfsSp(v) {
        const visitedSet = new Set(),
            pathMap = new Map(),
            distanceMap = new Map(),
            result = {},
            queue = [v], // 本应是一个队列，这里用数组代替
            root = v
        let level = 0,
            levelSize = 0,
            i

        // 核心代码
        while (levelSize = queue.length) {
            for (i = 0; i < levelSize; i++) {
                this._adjList[v].forEach(item => {
                    if (visitedSet.has(item) || queue.includes(item)) return
                    pathMap.set(item, v)
                    queue.push(item)
                })
                visitedSet.add(v)
                distanceMap.set(v, level)
                queue.shift()
                v = queue[0]
            }
            level++
        }

        // 整理输出
        this._vertices.forEach(item => {
            let vertex = item,
                path = ''
            for (; vertex !== root; vertex = pathMap.get(vertex)) {
                path += vertex
            }
            path = path === '' ? '' : path + root
            path = path.split('').reverse().join(' -> ')
            result[item] = {
                distance: distanceMap.get(item),
                path
            }
        })

        return result
    }
}

// 测试代码
const graph = new Graph()
const vertices = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
vertices.forEach(item => graph.addVertex(item))
graph.addEdge('A', 'B')
graph.addEdge('A', 'C')
graph.addEdge('A', 'D')
graph.addEdge('C', 'D')
graph.addEdge('C', 'G')
graph.addEdge('D', 'G')
graph.addEdge('D', 'H')
graph.addEdge('B', 'E')
graph.addEdge('B', 'F')
graph.addEdge('E', 'I')

// console.log(graph.toString()) // 查看图结构
// graph.bfs('A', (v) => { // 广度优先遍历
//     console.log(v)
// })
// console.log(graph.bfsSp('A')) // 最短路径
```

将一个有向图无环图（DAG）的所有顶点排成一个线性序列，若该线性序列满足：1、每个顶点出现且只出现一次；2、若存在一条从顶点 A 到顶点 B 的路径，那么在序列中顶点 A 出现在顶点 B 的前面。则该线性序列称为该图的一个拓扑排序。(注意：有向无环图是拓扑排序存在的充分必要条件）。

如何找到这个线性序列呢？在任一有向无环图中，必然存在入度为 0 的顶点。否则，每个顶点都至少有一条入边，这意味着包含环路。我们可以从这点入手，采用深度遍历进行处理。

```javascript
class Graph {
    // ...
    dfs(cb = () => {}) {
        const visitedSet = new Set()
        const _visite = (v) => {
            if (visitedSet.has(v)) return
            visitedSet.add(v)
            cb(v)
            this._adjList[v].forEach(item => _visite(item))
        }

        this._vertices.forEach(item => {
            _visite(item)
        })
    }
}
```
