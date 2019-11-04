# 反转数组

颠倒数组中元素的顺序。

## Example_1

```javascript
// input: [1, 2, 3, 4, 5, 6, 7, 8]
// output: [8, 7, 6, 5, 4, 3, 2, 1]
```

- **使用 Array 对象上的方法**

```javascript
function reverse(arr) {
    if (Object.prototype.toString.call(arr) !== "[object Array]") { // ES6：Array.isArray
        return []
    }
    return arr.reverse()
}
```

- **自定义函数**

```javascript
function swap(index1, index2, arr) {
    var temp = arr[index1]
    arr[index1] = arr[index2]
    arr[index2] = temp
}

function reverse(arr) {
    var i = 0,
        length,
        count

    if (Object.prototype.toString.call(arr) !== "[object Array]") {
        return []
    }

    if ((length = arr.length - 1) < 1) {
        return arr
    }
    count = length / 2

    for (; i < count; i++) {
        swap(i, length - i, arr)
    }

    return arr
}
```
