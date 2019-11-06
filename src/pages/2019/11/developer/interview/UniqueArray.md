# 数组去重

去除给定数组中的重复数据。

## Example_1

```javascript
// input: [0, 4, 3, 4, 5, 0, 8, 8]
// output: [0, 4, 3, 5, 8]
```

- **基础的 FOR 循环**

```javascript
function unique(arr) {
    var arrLen = arr.length,
        result = [],
        resLen = 0,
        i = 0,
        j = 0

    for (; i < arrLen; i++) {
        for (j = 0, resLen = result.length; j < resLen; j++) {
            if (arr[i] === result[j]) {
                break
            }
        }
        if (j === resLen) {
            result.push(arr[i])
        }
    }
    return result
}
```

- **ES6 Set**

```javascript
function unique(arr) {
    return [...new Set(arr)]
}
```
