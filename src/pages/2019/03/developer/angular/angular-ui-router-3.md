# UI-Router for AngularJS (四)
UI-Router is the defacto standard for routing in AngularJS.

## 嵌套路由
嵌套路由共有四种方法，由 `'dot notation'`，`parent property`（其值可以为字符串和对象），第三方模块(stateHelper)，这里不对 `stateHelper` 进行介绍，需要的同学可点击 [stateHelper](https://github.com/marklagendijk/ui-router.stateHelper) 查看。

## dot notation
使用嵌套路由时，默认行为是子状态将其 `url` 附加到其每个父状态的 `url`，定义的方式很简单，就是状态名的格式设置格式为  `<parentName>.<sonName>`。

```javascript
$stateProvider 
    .state('products', {
        url: '/products',
        ...
    })
    .state('products.list', {
        url: '/list',
        ...
    });
```

它们路径表现为：
 * 'products' state matches "/products"
 * 'products.list' state matches "/products/list".

当然我们也可以为其设置绝对路径，这样就不会受父路由的影响，而操作的方式就是在配置 `url` 是在前面添加 `^` 即可。

```javascript
$stateProvider 
    .state('products', {
        url: '/products',
        ...
    })
    .state('products.list', {
        url: '^/list',
        ...
    });
```

它们路径表现为：
 * 'products' state matches "/products"
 * 'products.list' state matches "/list".

## parent property
通过设置 `parent` 属性来指定它的父状态。

```javascript
// parent 的值可以是父状态的名称 
$stateProvider
    .state('products', {
        url: '/products',
        ...
    })
    .state('list', {
        url: '/list',
        parent: 'products'
        ...
    });

// parent 的值也可以是一个描述状态的对象
var products = { 
    name: 'products',
    templateUrl: 'products.html'
}
var productsList = { 
    name: 'list',
    parent: products,
    templateUrl: 'products.list.html'
}
 
$stateProvider
  .state(products)
  .state(productsList);
```

**父子状态的定义顺序可以随意，因为创建它们时会自动为其排队，首先创建父级状态。**

如果我们定义了子状态，那么其父级状态必须存在。

状态的名称不能重复，即使它们身处不同的父级状态下。

**当子状态被激活时，其父级状态也会被激活。**

**当子状态被激活时，将会加载其 `template` 显示在父级的 `ui-view `中，当没被指定父级状态时，显示在 `index.html` 的 `ui-view` 中。**

## 多命名视图
你可以给的视图进行命名，这样在一个模板里你可以拥有多个视图。

## 基本使用
要创建命名视图只需要在设置路由对象时添加 `views` 属性。需要注意的是如果你设置了 `views` 属性，那么路由的 `template`，`templateUrl`，`templateProvider` 将会被忽略，因此你可以创建一个抽象路由来包含模板，子路由都定义在 `views` 属性里。

```html
<!-- index.html -->
<body>
<div ui-view="viewName1"></div>	
<div ui-view="viewName2"></div>
<div ui-view="viewName3"></div>
</body>
```

```javascript
$stateProvider 
    .state('report', {
        views: {
            'viewName1': {},
            'viewName2': {},
            'viewName3': {}
        }
    });
```

在每个视图中可以设置自己的模板(template，templateUrl，templateProvider)和控制器(controller)。

```javascript
$stateProvider 
    .state('report', {
        views: {
            'viewName1': {
                templateUrl: 'view1.html',
                controller: 'ViewOneCtrl'
            },
            'viewName2': {
                templateUrl: 'view2.html',
                controller: 'ViewTwoCtrl'
            },
            'viewName3': {
                templateUrl: 'view3.html',
                controller: 'ViewThreeCtrl'
            }
        }
    });
```

##  相对命名和绝对命名
在定义 `views` 属性时，如果视图名称中包含 `@`，那么视图名称就是绝对命名的方式，否则就是相对命名的方式。

相对命名的意思是相对于父模板中的 `ui-view`，而绝对命名则指定了相对于哪个状态的模板。

在定义 `views` 属性后，其内部如果包含了 `@`，那么 `@` 前面部分为空表示未命名的 `ui-view`，`@` 后面为空则表示相对于根模板，通常是 `index.html`。

请注意，一旦使用了 `@` 则表示绝对命名的方式。

```html
<!-- index.html --> 
<body ng-app>
<div ui-view></div>
<div ui-view="status"></div>
</body>
<!-- products.html -->
<h1>My Products</h1>
<div ui-view></div>
<div ui-view="detail"></div>
<!-- products.detail.html -->
<h1>Products Details</h1>
<div ui-view="info"></div>
```

```javascript
$stateProvider 
    .state('products', {
        // 加载到 index.html 中未命名视图(ui-view)中
        templateUrl: 'products.html'
    })
    .state('products.detail', {
        views: {
            // 相对命名
            // products.html 中 <div ui-view='detail'/>
            "detail": {},
            // 相对命名
            // 对应 products.html 中的未命名 ui-view <div ui-view/>
            "": {},
            // 绝对命名
            // 对应 products.detail.html 中 <div ui-view='info'/>
            "info@products.detail": {},
            // 绝对命名
            // 对应 products.html 中 <div ui-view='detail'/>
            "detail@products": {},
            // 绝对命名
            // 对应 products.html 中的未命名视图： <div ui-view/>
            "@products": {},
            // 绝对命名
            // 对应根模板(index.html) 中名为 status 的视图： <div ui-view='status'/>
            "status@": {},
            // 绝对命名
            // 对应 index.html 中未命名视图：<div ui-view/>
            "@": {}
        }
    });
```

## 继承依赖
视图可以从它们所属的状态继承解析的依赖关系，但可能不继承它们的兄弟视图的那些。

```javascript
$stateProvider.state('multipleViews', { 
    resolve: {
        data1: function () {
            return {};
        }
    },
    views: {
        'view1@multipleViews': {
            templateUrl: 'multiple-views-view1.html',
            controller: function ($scope, data1, data2) {
                /* has access to data1 and data2,
                   but *not* data3 */
            },
            resolve: {
                data2: function () {
                    return {};
                }
            },
        },
        'view2@multipleViews': {
            templateUrl: 'multiple-views-view2.html',
            controller: function ($scope, data1, data3) {
                /* has access to data1 and data3,
                   but *not* data2 */
            },
            resolve: {
                data3: function () {
                    return { value: 'view2' };
                },
            },
        },
    }
});
```

<p class="text-right">
    如需了解更多信息，点击查看 <a href="./angular-ui-router-4.html">UI-Router for AngularJS (四)</a>
</p>

## 参考资料
 * [https://github.com/angular-ui/ui-router/wiki/URL-Routing](https://github.com/angular-ui/ui-router/wiki/URL-Routing)