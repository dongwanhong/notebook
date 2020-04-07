# Angular 是如何监听数据变化的（二）

在 `AngularJS` 中，使用继承将每个 `Scope` 关联起来。它的功能十分强大，但往往也会产生一些难以理解的事情，所以在使用的过程中需要特别注意。

当我们创建一个 `AngularJS` 应用时，默认的就会创建一个根 `Scope`（$rootScope），它是所有 `Scope` 除了 `Object` 外最终会指向的父级。

通常会需要根据现有的 `Scope` 来创建新的 `Scope`，我们可以通过名为 `$new` 的函数来完成这项工作。

```javascript
/**
 * @description
 * 使用了原型式继承，根据现有的 Scope 创建新的 Scope 对象
 * @returns { object } 新的 Scope 对象
 */
Scope.prototype.$new = function () {
    var ChildScope = function () {};
    ChildScope.prototype = this;
    var child = new ChildScope();
    return child;
};
```

现在，如果我们在子级的作用域上查找数据时就会顺着作用域链来进行查找。需要注意的时，如果我们想要把子级上的更改同时反应在父级，那么需要把基本类型的值放在应用数据类型上面。

```javascript
var parent = new Scope();
var child = parent.$new();

// parent.name = 'Tom';
// child.name = 'Jack';
// console.log(parent.name); // 'Tom'
parent.user = {
    name: 'Tom'
};
child.user.name = 'Jack';
console.log(parent.name); // 'Jack'
```

另外，根据之前的实现。在任何新的子 `Scope` 中共享了同一个 `$$watchers`，所以当我们在任何地方调用 `$digest` 时，所有的监听器都会执行，而每每我们只需要监听当前 `Scope` 及其子 `Scope` 就可以了。为了解决这个问题，我们对之前的 `$new` 方法进行改进。

我们需要做的就是在继承时，为每一个新的作用域创建一个 `$$watchers` 来存储当前作用域的监听器。对于后者，现在有一个问题是一个作用域并不知道自己有无或者有几个子作用域，因此我们需要一个属性 `$$children`（数组） 来进行记录，与 `$$watchers` 一样，每个 `Scope` 都应该拥有自己的。

```javascript
function Scope() {
    // ... 省略部分代码
    this.$children = []; // 记录继承自己的子级 Scope
}

Scope.prototype.$new = function () {
    var ChildScope = function () {};
    ChildScope.prototype = this;
    var child = new ChildScope();
    child.$$watchers = [];
    child.$$children = [];
    this.$$children.push(child); // 将新建的 Scope 存储到父级中
    return child;
};
```

现在我们需要在调用父级的 `$digest` 时，同时调用子级的。为此，我们先创建一个方法，这个方法接受一个函数，然后递归的调用。

```javascript
/**
 * @description
 * 接受一个函数，将当前作用域传入执行后，如果正常则陆续传入自己的子作用域进行调用
 * @returns { boolean }
 */
Scope.prototype.$$everyScope = function (fn) {
    if (fn(this)) {
        return this.$$children.every(function (child) {
            return child.$$everyScope(fn);
        });
    } else {
        return false;
    }
};
```

我们把这个函数应用到 `$$digestOnce` 函数中，需要注意的是，如果在循环检测的过程中，如果我们检测到上一次的脏值已经干净了，那么就不需要再继续遍历检测啦。

```javascript
Scope.prototype.$$digestOnce = function () {
    var self = this;
    var dirty = false;
    var continueLoop = true;

    self.$$everyScope(function (scope) {
        var newValue, oldValue, equal;
        _.forEachRight(scope.$$watchers, function (watcher) {
            try {
                if (watcher) {
                    newValue = watcher.watchFn(scope);
                    oldValue = watcher.last;
                    equal = watcher.eq ? _.isEqual(newValue, oldValue) : (newValue === oldValue || (_.isNaN(newValue) && _.isNaN(oldValue)));
                    if (newValue !== oldValue && !equal) {
                        self.$$lastDirtyWatch = watcher;
                        watcher.last = watcher.eq ? _.cloneDeep(newValue) : newValue;
                        dirty = true;
                        watcher.listenerFn(newValue, oldValue === initWatchVal ? undefined : oldValue, scope);
                    } else if (self.$$lastDirtyWatch === watcher) {
                        continueLoop = false;
                        return false; // exit iteration
                    }
                }
            } catch (err) {
                console.log(err);
            }
        });
        return continueLoop;
    })

    return dirty;
};
```

在我们的目前的实现中，如果我们在子级的 `Scope` 中调用 `$apply`、`$applyAsync` 和 `$evalAsync` 的话，它并不能调用整个 `$digest` 循环，这与 `AngularJs` 中的实现是不一致的，因此我们需要用一个属性（`$root`）来记录顶级的 `Scope`，从而通过它来开启整个程序中的 `$digest` 循环。

```javascript
function Scope() {
    // ... 省略部分代码
    this.$root = this; // 记录根作用域
}
```

然后分别改变对应方法中的循环的调用方式。

```javascript
Scope.prototype.$apply = function (expr) {
    try {
        this.$beginPhase("$apply");
        return this.$eval(expr);
    } finally {
        this.$clearPhase();
        this.$root.$digest();
    }
};

Scope.prototype.$evalAsync = function (expr) {
    var self = this;
    if (!self.$$phase && self.$$asyncQueue.length) {
        setTimeout(function () {
            if (self.$$asyncQueue.length) {
                self.$root.$digest();
            }
        }, 4);
    }
    // ... 省略部分代码
};
```

还记得前面优化时我们使用的 `$$lastDirtyWatch` 属性吗？现在有了 `$root` 属性，我们可以总是访问和使用根作用域上的 `$$lastDirtyWatch` 属性以确保正确性。因此我们把在之前对 `$$lastDirtyWatch` 属性的引用改成从 `$root` 属性上引用。

```javascript
// 除了初始声明
// 替换所有 .$$lastDirtyWatch 为 .$root.$$lastDirtyWatch
```

替换之后再回到最开始继承所用的 `$new` 方法，目前通过它产生的子级作用域总是继承了父级，但有时候我们希望产生一个隔离作用域，让它不继承任何其它的作用域，现在我们在之前的基础上通过传入一个参数来实现。

```javascript
/**
 * @description
 * 创建新的 Scope 对象
 * @param { boolean } isolated 是否创建隔离的 Scope 对象
 * @returns { object } 新的 Scope 对象
 */
Scope.prototype.$new = function (isolated) {
    var child = null;
    if (isolated) {
        child = new Scope();
        child.$root = this.$root;
    } else {
        var ChildScope = function () {};
        ChildScope.prototype = this;
        child = new ChildScope();
    }
    child.$$watchers = [];
    child.$$children = [];
    this.$$children.push(child);
    return child;
};
```

不过，无论是不是隔离的作用域它们的 `$$asyncQueue`、`$$applyAsyncQueue` 和 `$$postDigestQueue` 应该总是相同的，因此我们修改隔离作用中这三个属性的指向。

而且 `$$applyAsyncQueue` 还略微有点不同，因为它涉及到了我们之前用来优化的 `$$applyAsyncId` 属性，为了正常工作，我们同样把对 `$$applyAsyncId` 的引用都从根作用域开始。

```javascript
Scope.prototype.$new = function (isolated) {
    var child = null;
    if (isolated) {
        child = new Scope();
        child.$root = this.$root;
        child.$$asyncQueue = this.$$asyncQueue;
        child.$$postDigestQueue = this.$$postDigestQueue;
        child.$$applyAsyncQueue = this.$$applyAsyncQueue;
    }  // ... 省略部分代码
};

// 除了初始声明
// 替换所有 .$$applyAsyncId 为 .$root.$$applyAsyncId
```

现在继承也只能从当前的作用域开始继承，为了支持从指定的作用域开始继承，我们新添加一个参数。

```javascript
/**
 * @description
 * 创建新的 Scope 对象
 * @param { boolean } isolated 是否创建隔离的 Scope 对象
 * @param { object } parent 可选的，指定继承的对象
 * @returns { object } 新的 Scope 对象
 */
Scope.prototype.$new = function (isolated, parent) {
    var child = null;
    parent = parent || this;
    if (isolated) {
        child = new Scope();
        child.$root = parent.$root;
        child.$$asyncQueue = parent.$$asyncQueue;
        child.$$postDigestQueue = parent.$$postDigestQueue;
        child.$$applyAsyncQueue = parent.$$applyAsyncQueue;
    } else {
        var ChildScope = function () {};
        ChildScope.prototype = this;
        child = new ChildScope();
    }
    child.$$watchers = [];
    child.$$children = [];
    parent.$$children.push(child);
    return child;
};
```

现在，作用域继承的问题实现得差不多了，最后我们添加一个销毁作用域的方法，这个方法同时依赖它的父子作用域，但是目前为止，子级作用域并没有什么明确的方法去获取到，因此我们在 `$new` 方法中增加一项，为新生成的作用域添加 `$parent` 属性。

```javascript
Scope.prototype.$new = function (isolated, parent) {
    // ... 省略部分代码
    child.$parent = parent;
    return child;
};
```

然后我们就可以实现销毁的方法了。

```javascript
/**
 * @description
 * 从父级中移除自己
 */
Scope.prototype.$destroy = function () {
    if (this.$parent) {
        _.remove(this.$parent.$$children, this);
    }
    this.$$watchers = null;
};
```

前面我们以及实现了两种监听方式，现在我们来创建第三种监听，也就是 `$watchCollection`，它是对对象的一种浅的比较，所以它更快、占用的内存也更小。

它的实现与前面的设计很像，而且虽然它的目的是为了检测对象，但是当使用基本类型时也可以同样工作，我们可以把这部分工作交给之前的 `$watch` 去完成。

将 `newValue` 和 `oldValue` 声明保留在内部监视函数之外，我们可以在内部监视和侦听函数之间共享它们。

```javascript
/**
 * @description
 * @param {function} watchFn 指定关注的数据
 * @param {function} listenerFn 数据变化后的处理函数
 * @returns {void}
 */
Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue;
    var self = this;
    var internalWatchFn = function (scope) {
        newValue = watchFn(scope);
        oldValue = newValue;
    };
    var internalListenerFn = function () {
        listenerFn(newValue, oldValue, self);
    };
    return this.$watch(internalWatchFn, internalListenerFn);
};
```

往前，我们通过指定关注数据的函数的返回值异同来决定是否调用对应的处理函数，但是现在这个函数没有任何返回值，所以它的处理函数用域不会被执行。

为此，我们引入一个整数计数器在前后数据发生变化后使其递增。

```javascript
/**
 * @description
 * 比较两个值是否相等
 * @param {any} newValue
 * @param {any} oldValue
 * @param {boolean} deep 是否进行深度比较
 * @returns {boolean} 两个值是否相等
 */
function $$isEqual(newValue, oldValue, deep) {
    if (deep) {
        return _.equal(newValue, oldValue);
    }
    return newValue === oldValue || (_.isNaN(newValue) && _.isNaN(oldValue));
}

Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue;
    var self = this;
    var changeCount = 0;
    var internalWatchFn = function (scope) {
        newValue = watchFn(scope);
        if (!$$isEqual(newValue, oldValue, false)) {
            oldValue = newValue;
            changeCount++;
        }
        return changeCount;
    };
    var internalListenerFn = function () {
        listenerFn(newValue, oldValue, self);
    };
    return this.$watch(internalWatchFn, internalListenerFn);
};
```

现在，我们开始来实现对对象的检测，由于数组也是特殊的对象，所以我们还需要进行特俗的处理。

```javascript
Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue;
    var self = this;
    var changeCount = 0;
    var internalWatchFn = function (scope) {
        newValue = watchFn(scope);
        if (_.isObject(newValue)) {
            if (_.isArray(newValue)) {

            } else {

            }
        } else {
            if (!$$isEqual(newValue, oldValue, false)) {
                changeCount++;
            }
            oldValue = newValue;
        }
        return changeCount;
    };
    var internalListenerFn = function () {
        listenerFn(newValue, oldValue, self);
    };
    return this.$watch(internalWatchFn, internalListenerFn);
};
```

首先如果确定为数组了，我们可以简单的检测一下之前的值是否是数组，如果不是那么显然发生了改变，那么就需要增减整数的值。

如果前后的值都是数组了，那么就可以接着检测前后数组的长度是否相等。

当长度一致时，再对前后数组中的每一项进行比较，这里需要注意对同时是 `NaN` 进行特殊的处理。

最后，对于一些类数组的数据，我们需要将其当作是数组来进行处理，所以我们在对新值进行判断时改为使用类数组函数进行判断。

```javascript
/**
 * @description
 * 向 lodash 添加方法检测是否是类数组
 * @param {*} target 检测的对象
 * @returns {boolean} 是否是类数组
 */
_.mixin({
    isArrayLike: function (target) {
        if (_.isNull(target) || _.isUndefined(target)) {
            return false;
        }
        var length = target.length;
        return _.isNumber(length);
    }
});


Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    // ... 省略部分代码
    var internalWatchFn = function (scope) {
        newValue = watchFn(scope);
        if (_.isObject(newValue)) {
            if (_.isArrayLike(newValue)) {
                if (!_.isArray(oldValue)) { // 类型
                    changeCount++;
                    oldValue = [];
                }
                if (newValue.length !== oldValue.length) { // 长度
                    changeCount++;
                    oldValue.length = newValue.length;
                }
                _.forEach(newValue, function (newItem, index) { // 数组元素
                    var bothNaN = _.isNaN(newItem) && _.isNaN(oldValue[index]);
                    if (!bothNaN && newItem !== oldValue[index]) {
                        changeCount++;
                        oldValue[index] = newItem;
                    }
                });
            } else {
                // ...
            }
        } else {
            if (!$$isEqual(newValue, oldValue, false)) {
                changeCount++;
            }
            oldValue = newValue;
        }
        return changeCount;
    };
    // ... 省略部分代码
};
```

同理，我们对其它对象进行检测，如果旧值不是对象或者是类数组对象，那么就是发生了改变。

如果通过了上面的校验，那么接下来就需要校验对象身上的属性前后是否发生了改变。同样这里我们需要对前后都是 `NaN` 的情况进行特殊处理。

```javascript
if (_.isObject(newValue)) {
    if (_.isArrayLike(newValue)) {
        // ... 省略部分代码
    } else {
        if (!_.isObject(oldValue) || _.isArrayLike(oldValue)) {
            changeCount++;
            oldValue = {};
        }
        _.forOwn(newValue, function (newVal, key) { // 校验属性的值
            var bothNaN = _.isNaN(newVal) && _.isNaN(oldValue[key]);
            if (!bothNaN && oldValue[key] !== newVal) {
                changeCount++;
                oldValue[key] = newVal;
            }
        });
        _.forOwn(oldValue, function (oldVal, key) { // 是否移除了属性
            if (!newValue.hasOwnProperty(key)) {
                changeCount++;
                delete oldValue[key];
            }
        });
    }
}
```

可以看出上面总是会做两次全量的循环，现在我们来对其进行一些优化，用两个值来记录前后属性的个数，如果之前的个数比后续的多，则说明有移除属性，否则就没必要循环了。

```javascript
Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue;
    var self = this;
    var changeCount = 0;
    var oldLength;
    var internalWatchFn = function (scope) {
        var newLength;
        newValue = watchFn(scope);
        if (_.isObject(newValue)) {
            if (_.isArrayLike(newValue)) {
                // ... 省略部分代码
            } else {
                if (!_.isObject(oldValue) || _.isArrayLike(oldValue)) {
                    changeCount++;
                    oldValue = {};
                    oldLength = 0;
                }
                newLength = 0;
                _.forOwn(newValue, function (newVal, key) { // 校验属性的值
                    newLength++;
                    if (oldValue.hasOwnProperty(key)) {
                        var bothNaN = _.isNaN(newVal) && _.isNaN(oldValue[key]);
                        if (!bothNaN && oldValue[key] !== newVal) {
                            changeCount++;
                            oldValue[key] = newVal;
                        }
                    } else {
                        changeCount++;
                        oldLength++;
                        oldValue[key] = newVal;
                    }
                });
                if (oldLength > newLength) {
                    changeCount++;
                    _.forOwn(oldValue, function (oldVal, key) { // 是否移除了属性
                        if (!newValue.hasOwnProperty(key)) {
                            oldLength--;
                            delete oldValue[key];
                        }
                    });
                }
            }
        } else {
            if (!$$isEqual(newValue, oldValue, false)) {
                changeCount++;
            }
            oldValue = newValue;
        }
        return changeCount;
    };
    // ... 省略部分 代码
};
```

前面，我们对类数组进行了判断，在一点程度上判断是居于 `length` 属性的，但对于一些后天拥有 `length` 属性的对象可能会存在误判。

现在，我们需要对判断的函数进行一次改进。

```javascript
_.mixin({
    isArrayLike: function (target) {
        if (_.isNull(target) || _.isUndefined(target)) {
            return false;
        }
        var length = target.length;
        return length === 0 ||
            (_.isNumber(length) && length > 0 && (length - 1) in obj);
    }
});
```

显然，目前为止存在一个问题，就是 `internalWatchFn` 总是比 `internalListenerFn` 先执行的，由于在 `internalWatchFn` 已经更新了旧值，所以现在我们在 `internalListenerFn` 函数中拿到的新旧值总是相等的，因此我们需要使用单独的变量来对旧值进行记录，而且是使用浅拷贝。

```javascript
Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue, olderValue;
    var self = this;
    var changeCount = 0;
    var oldLength;
    var trackOlderValue = (listenerFn.length > 1);
    var internalWatchFn = function (scope) {
        // ... 省略部分代码
    };
    var internalListenerFn = function () {
        listenerFn(newValue, olderValue, self);

        if (trackOlderValue) {
            olderValue = _.clone(newValue);
        }
    };
    return this.$watch(internalWatchFn, internalListenerFn);
};
```

由于第一次运行时，我们没有给 `olderValue` 给与任何值，所以在第一次做判断时可能会出现偏差，因为我们需要使用一个布尔值来进行区别对待。

```javascript
Scope.prototype.$watchCollection = function (watchFn, listenerFn) {
    var oldValue, newValue, olderValue;
    var self = this;
    var changeCount = 0;
    var oldLength;
    var trackOlderValue = (listenerFn.length > 1);
    var firstRun = true;
    var internalWatchFn = function (scope) {
        // ... 省略部分代码
    };
    var internalListenerFn = function () {
        if (firstRun) {
            listenerFn(newValue, newValue, self);
            firstRun = false;
        } else {
            listenerFn(newValue, olderValue, self);
        }

        if (trackOlderValue) {
            olderValue = _.clone(newValue);
        }
    };
    return this.$watch(internalWatchFn, internalListenerFn);
};
```

到此，我们就要开始来实现 `AngularJS` 中的事件系统了，它使用了著名的 `publish-subscribe-messaging` 模式，并使用 `Scope` 对象上的 `$on` 方法来进行注册，然后把注册的事件名称作为 `key`，对应的处理函数作为值存储在 `$$listeners` 对象中。

和 `$$watchers` 的处理方式一样，每个子级都应该有自己的 `$$watchers` 对象。

```javascript
function Scope() {
    // ... 省略部分代码
    this.$$listeners = {}; // 存储注册的事件
}

Scope.prototype.$new = function (isolated, parent) {
    // ... 省略部分代码
    child.$$children = [];
    child.$$listeners = {};
    parent.$$children.push(child);
    return child;
};

/**
 * @description
 * 注册事件和事件被触发时的处理函数
 * @param {string} eventName 事件名称
 * @param {function} listener 处理函数
 * @returns {void}
 */
Scope.prototype.$on = function (eventName, listener) {
    var listeners = this.$$listeners[eventName];
    if (!listeners) {
        this.$$listeners[eventName] = listeners = [];
    }
    listeners.push(listener);
};
```

现在事件已经被注册好了，我们需要通过函数来进行触发。

```javascript
/**
 * @description
 * 触发指定的事件（AngularJS 中没有抽离 $emit、$broadcast 中的公共部分，也就是说没有此函数）
 * @param {string} eventName 被触发事件的名称
 * @returns {void}
 */
Scope.prototype.$$fireEventOnScope = function (eventName) {
    var listeners = this.$$listeners[eventName] || [];
    _.forEach(listeners, function (listener) {
        listener();
    });
};

/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {void}
 */
Scope.prototype.$emit = function (eventName) {
    this.$$fireEventOnScope(eventName);
};

/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {void}
 */
Scope.prototype.$broadcast = function (eventName) {
    this.$$fireEventOnScope(eventName);
};
```

可见，`$emit` 和 `$broadcast` 都可以触发已经被注册的指定事件了，但是处理函数中目前为止没有接受到一个事件对象。因此，我们首先给它一个简单的事件对象，至少可以通过 `name` 来获取事件名称。

```javascript
Scope.prototype.$$fireEventOnScope = function (eventName) {
    var event = { name: eventName };
    var listeners = this.$$listeners[eventName] || [];
    _.forEach(listeners, function (listener) {
        listener(event);
    });
    return event;
};
```

接着，我们还需要把触发事件时传入的更多参数传递给事件处理函数。

```javascript
/**
 * @description
 * 触发指定的事件（AngularJS 中没有抽离 $emit、$broadcast 中的公共部分，也就是说没有此函数）
 * @param {string} eventName 被触发事件的名称
 * @returns {object} 事件对象
 */
Scope.prototype.$$fireEventOnScope = function (eventName, args) {
    var event = {
        name: eventName
    };
    var listenerArgs = [event].concat(args);
    var listeners = this.$$listeners[eventName] || [];
    _.forEach(listeners, function (listener) {
        listener.apply(null, listenerArgs);
    });
    return event;
};

/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {*} 取决于 $$fireEventOnScope 函数的返回值
 */
Scope.prototype.$emit = function (eventName) {
    var args = _.rest(arguments);
    return this.$$fireEventOnScope(eventName, args);
};

/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {*} 取决于 $$fireEventOnScope 函数的返回值
 */
Scope.prototype.$broadcast = function (eventName) {
    var args = _.rest(arguments);
    return this.$$fireEventOnScope(eventName, args);
};
```

现在事件可以创建和触发了，但是很多时候我们需要移除掉已经注册的事件，因此我们和前面注册监听一样，返回一个函数来移除事件。需要特别注意的一点时，如果注册的事件在处理函数中移除了自己，如果我们用直接从事件数组中移除元素的方法可能会导致会跳过一个处理函数，因此我们采用将移除的元素设置为 `null` 值，然后在触发函数中进行判断即可。

```javascript
/**
 * @description
 * 注册事件和事件被触发时的处理函数
 * @param {string} eventName 事件名称
 * @param {function} listener 处理函数
 * @returns {function} 移除对应事件的处理函数
 */
Scope.prototype.$on = function (eventName, listener) {
    var listeners = this.$$listeners[eventName];
    if (!listeners) {
        this.$$listeners[eventName] = listeners = [];
    }
    listeners.push(listener);
    return function () {
        var index = listeners.indexOf(listener);
        if (index >= 0) {
            listeners[index] = null;
        }
    };
};

/**
 * @description
 * 触发指定的事件，并移除为 null 的项（AngularJS 中没有抽离 $emit、$broadcast 中的公共部分，也就是说没有此函数）
 * @param {string} eventName 被触发事件的名称
 * @returns {object} 事件对象
 */
Scope.prototype.$$fireEventOnScope = function (eventName, args) {
    var event = {
        name: eventName
    };
    var listenerArgs = [event].concat(args);
    var listeners = this.$$listeners[eventName] || [];
    var i = 0;
    while (i < listeners.length) {
        if (listeners[i] === null) {
            listeners.splice(i, 1);
        } else {
            listeners[i].apply(null, listenerArgs);
            i++;
        }
    }
    return event;
};
```

现在，我们开始来事项 `$emit` 包括自己直到根作用域一直触发指定事件的特性。

```javascript
/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {object} 事件对象
 */
Scope.prototype.$emit = function (eventName) {
    var event = {
        name: eventName
    };
    var args = [event].concat(_.rest(arguments));
    var scope = this;
    do {
        scope.$$fireEventOnScope(eventName, args);
        scope = scope.$parent;
    } while (scope);
    return event;
};

/**
 * @description
 * 触发指定的事件，并移除为 null 的项（AngularJS 中没有抽离 $emit、$broadcast 中的公共部分，也就是说没有此函数）
 * @param {string} eventName 被触发事件的名称
 * @returns {void}
 */
Scope.prototype.$$fireEventOnScope = function (eventName, args) {
    var listeners = this.$$listeners[eventName] || [];
    var i = 0;
    while (i < listeners.length) {
        if (listeners[i] === null) {
            listeners.splice(i, 1);
        } else {
            listeners[i].apply(null, args);
            i++;
        }
    }
};
```

同理，根据 `$broadcast` 的特性进行调整。

```javascript
/**
 * @description
 * 触发指定的事件
 * @param {string} eventName 被触发事件的名称
 * @returns {object} 事件对象
 */
Scope.prototype.$broadcast = function (eventName) {
    var event = {
        name: eventName
    };
    var args = [event].concat(_.rest(arguments));
    this.$$everyScope(function (scope) {
        scope.$$fireEventOnScope(eventName, args);
    });
    return event;
};
```

显然，我们现在的事件对象还很薄弱，因此接下来我们来给它加上一些东西，首先使用 `targetScope` 来记录触发此事件的 `Scope`，用 `currentScope` 来记录当前处理函数所处的 `Scope`。

最后我们在事件触发过程完成后清除掉 `currentScope` 的指向。

```javascript
Scope.prototype.$emit = function (eventName) {
    var event = {
        name: eventName,
        targetScope: this
    };
    var args = [event].concat(_.rest(arguments));
    var scope = this;
    do {
        event.currentScope = scope;
        scope.$$fireEventOnScope(eventName, args);
        scope = scope.$parent;
    } while (scope);
    event.currentScope = null;
    return event;
};

Scope.prototype.$broadcast = function (eventName) {
    var event = {
        name: eventName,
        targetScope: this
    };
    var args = [event].concat(_.rest(arguments));
    this.$$everyScope(function (scope) {
        event.currentScope = scope;
        scope.$$fireEventOnScope(eventName, args);
    });
    event.currentScope = null;
    return event;
};
```

在真实的 DOM 事件中，我们可以调用事件对象上的 `stopPropagation` 方法来阻止事件冒泡，同样的，我们对自定义的对象也加上这个方法并实现它。

```javascript
Scope.prototype.$emit = function (eventName) {
    var stopPropagation = false;
    var event = {
        name: eventName,
        targetScope: this,
        stopPropagation: function () {
            stopPropagation = true;
        }
    };
    var args = [event].concat(_.rest(arguments));
    var scope = this;
    do {
        event.currentScope = scope;
        scope.$$fireEventOnScope(eventName, args);
        scope = scope.$parent;
    } while (!stopPropagation && scope);
    event.currentScope = null;
    return event;
};
```

在真实的 DOM 事件中，另一个经常使用的方法就是 `preventDefault`，虽然目前为止我们注册的事件，还没有任何默认的行为，但我们可以先实现这个方法，尽管它只是改变一下 `defaultPrevented` 的值。

```javascript
Scope.prototype.$emit = function (eventName) {
    var stopPropagation = false;
    var event = {
        name: eventName,
        targetScope: this,
        defaultPrevented: false,
        stopPropagation: function () {
            stopPropagation = true;
        },
        preventDefault: function () {
            event.defaultPrevented = true;
        }
    };
    // ... 省略部分代码
};

Scope.prototype.$broadcast = function (eventName) {
    var event = {
        name: eventName,
        targetScope: this,
        defaultPrevented: false,
        preventDefault: function () {
            event.defaultPrevented = true;
        }
    };
    // ... 省略部分代码
};
```

许多时候，对于直到一个 `Scope` 何时被销毁是很有用的，因此我们在之前提供的销毁 `Scope` 的方法中增加一行来广播销毁事件，以便一些 `Scope` 随之销毁绑定的事件等一些事情。

同时，如果一个 `Scope` 被销毁，我们应该注销掉其上的注册的事件和对应的处理函数。

```javascript
Scope.prototype.$destroy = function () {
    this.$broadcast('$destroy');
    if (this.$parent) {
        _.remove(this.$parent.$$children, this);
    }
    this.$$watchers = null;
    this.$$listeners = {};
};
```

最后，为了注册的事件执行时导致的错误不至于让整个程序崩溃，所以我们需要对处理函数进行处理。简单的处理就是，捕获错误，然后输出到控制台。

```javascript
Scope.prototype.$$fireEventOnScope = function (eventName, args) {
    var listeners = this.$$listeners[eventName] || [];
    var i = 0;
    while (i < listeners.length) {
        if (listeners[i] === null) {
            listeners.splice(i, 1);
        } else {
            try {
                listeners[i].apply(null, args);
            } catch (err) {
                console.log(err);
            }
            i++;
        }
    }
};
```
