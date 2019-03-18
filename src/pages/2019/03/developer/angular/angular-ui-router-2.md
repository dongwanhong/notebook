# UI-Router for AngularJS (三)
UI-Router is the defacto standard for routing in AngularJS.

## 基本概念
通常，URL具有动态部分，被称为参数，这在现在的单页面程序开发中使用的很多。

比如在一个商品展示的网页，当我们点击一件商品进入详情页时，它们所用的页面通常是同一个，只是获取的数据不同，为了给每件商品提供正确的详情数据，因此我们需要提供动态的数据去区别它们。

页面传参的方式一般具有两种，一种是 `url` 传参，这样传递的参数会在 `url` 中显示出来，另外一种传递参数的方式是直接在路由配置中通过指定 `params` 属性，这样传递的参数不会出现在 `url` 中。

## 路径传参
路由配置。

```javascript
// basic params 
$stateProvider
    .state('home.products', {
        url: "/home/:productId", // 也可以换作：url: "/home/{productId}"
        templateUrl: 'home.detail.html',
        controller: function ($stateParams) {
            // If we got here from a url of /home/1
            console.log($stateParams.productId); // 1
        }
    });
 
// query params
$stateProvider
    .state('home.products', {
        url: "/products?myParam" // will match to url of "/products?myParam=value"
    });
$stateProvider
    .state('home.products', {
        url: "/products?myParam1&myParam2" // will match to url of "/products?myParam1=value1&myParam2=value2"
    });
```

在页面中传参。

```html
<!-- The value for id can be anything in scope. -->
<a ui-sref="home.products({productId: id})">View product</a>

<!-- 多个参数 -->
<a ui-sref="home.products({myParam1=value1, myParam2=value2})">View product</a>
```

```javascript
// 或者您也可以将它们传递给$ state.go（）
$state.go('products', {param1: value1});
 
// 多个参数
$state.go('products', {param1: value1, param2: value2});
```

## 配合 params 属性传参
就像前面所说的，配合 `params` 属性传参的参数不会暴露在 `url` 中，其用法区别仅在于在配置路由时，预先设置参数的值为 `null` 即可。

```javascript
.state('products', { 
    url: "/products",
    params: {
        param1: null
    },
    templateUrl: 'products.html'
});

// 多个参数
.state('products', {
    url: "/products",
    params: {
        param1: null,
        param2: null
    },
    templateUrl: 'products.html'
});
```

## 获取父级的参数
`$StateParams` 仅包含注册在当前状态下的参数，不包含其他状态下的参数，即使是上级的 `url` 参数也获取不到。

```javascript
$stateProvider.state('products.detail', { 
    url: '/products/:productId',
    controller: function ($stateParams) {
        $stateParams.productId  //*** Exists! ***//
    }
}).state('products.detail.subitem', {
    url: '/item/:itemId', 
    controller: function ($stateParams) {
        $stateParams.productId //*** Watch Out! DOESN'T EXIST!! ***//
        $stateParams.itemId //*** Exists! ***//  
    }
});
```

若想让下级获取到当前状态的参数，需使用 `resolve()`。该函数会在画面渲染出来前先执行完成。

```javascript
$stateProvider.state('products.detail', {
    url: '/products/:productId',
    controller: function ($stateParams) {
        $stateParams.productId  //*** Exists! ***//
    },
    resolve:{
        productId: ['$stateParams', function ($stateParams) {
            return $stateParams.productId;
        }]
    }
}).state('products.detail.subitem', {
    url: '/item/:itemId', 
    controller: function ($stateParams, productId){
        productId //*** Exists! ***//
        $stateParams.itemId //*** Exists! ***//  
    }
});
```

## 传递对象
传递。

```javascript
var obj = {
    name: 'Anani',
    age: 24
};
$state.go("home.person", {object: JSON.stringify(obj)});
```

接收。

```javascript
JSON.parse($state.params.object);
```

<p class="text-right">
    如需了解更多信息，点击查看 <a href="./angular-ui-router-3.html">UI-Router for AngularJS (三)</a>
</p>

## 参考资料
 * [https://github.com/angular-ui/ui-router/wiki/URL-Routing](https://github.com/angular-ui/ui-router/wiki/URL-Routing)