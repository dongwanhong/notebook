# UI-Router for AngularJS (二)
UI-Router is the defacto standard for routing in AngularJS.

## 状态名和路径
```javascript
// 基于字符串定义 state? 
$stateProvider.state('home', {
    url: '/home' // string
    // more config
});
 
// or 基于对象定义 state?
var home = {
    name: 'home',
    url: '/home' // string
    // more config
}
$stateProvider.state(home);
```

## Templates 模板
 * 指定一段HTML字符串，这也是设置模板的最简单的方式。

```javascript
$stateProvider.state('home', { 
    url: '/home',
    template: '<div>Hello world</div>'
});
```

 * 配置 `templateUrl` 属性来加载指定位置的模板，这是设置模板的常用方法。

```javascript
$stateProvider.state('home', { 
    url: '/home',
    template: 'home.html'
});
```

 * `templateUrl` 的值也可以是一个函数返回的 `url`，函数带一个预设参数 `stateParams`。

```javascript
$stateProvider.state('home', { 
    url: '/home',
    template: function (stateParams) {
        return 'stateParams.templateName' + '.html';
    }
});
```

 * 通过 `templateProvider` 函数返回模板的HTML。

```javascript
$stateProvider.state('home', { 
    url: '/home',
    templateProvider: function ($timeout, $stateParams) {
        return $timeout(function () {
            return '<div>Hello world</div>';
        });
    }
});
```

## Controllers 控制器
为模板指定一个控制器（controller）。如果模板没有定义，控制器不会被实例化。

 * 设置控制器。

```javascript
$stateProvider.state('home', { 
    template: ...,
    controller: function ($scope) {
        $scope.title = 'UI router';
    }
});
```

 * 如果在模块中已经定义了一个控制器，只需要指定控制器的名称即可。

```javascript
$stateProvider.state('home', { 
    template: ...,
    controller: 'HomeCtrl'
});
```

 * 使用 `controllerAs` 语法。

```javascript
$stateProvider.state('home', { 
    template: ...,
    controller: 'HomeCtrl as home'
});
```

 * 对于更高级的需要，可以使用 `controllerProvider` 来动态返回一个控制器函数或字符串。

```javascript
$stateProvider.state('home', { 
    template: ...,
    controllerProvider: function ($stateParams) {
        var ctrlName = $stateParams.type + "Ctrl";
        return ctrlName;
    }
});
```

控制器可以使用 `$scope.on()` 方法来监听事件状态转换。

## Resolve 预载入
使用预载入功能可以预先载入一系列依赖或者数据，然后注入到控制器中。

`resolve` 配置项是一个由 `key/value` 构成的对象。

 * key – { string } ：注入控制器的依赖项名称。
 * factory - { string | function } ：
   * string：一个服务的别名。
   * function：函数的返回值将作为依赖注入项，如果函数是一个耗时的操作，那么控制器必须等待该函数执行完成（be resolved）才会被实例化。

```javascript
$stateProvider.state('home', { 
    resolve: {
        // 此函数只是简单的返回了值，所以会被立即载入
        user: function () {
            return {
                name: 'Anani',
                age: 24
            }
        },
        // 可以给函数注入任何想要的服务依赖，比如这里的 $http
        userData: function ($http) {
            return $http({
                method: 'get',
                url: '/users'
            });
        },
        // 如果想对返回结果进行处理， 可以使用 .then 方法
        userFiltered: function ($http) {
            return $http({
                method: 'get',
                url: '/users'
            }).then(function (data) {
                return dosomething(data);
            });
        },
        // 前一个数据保证也可作为依赖注入到其他数据保证中
        detailData: function ($http, userFiltered) {
            return $http({
                method: 'get',
                url: '/users',
                params: {
                    name: userFiltered.data.user[0]
                }
            });
        },
        // Example using a service by name as string.
        // This would look for a 'translations' service
        // within the module and return it.
        // Note: The service could return a promise and
        // it would work just like the example above
        translations: "translations",
        // Example showing injection of service into
        // resolve function. Service then returns a
        // promise. Tip: Inject $stateParams to get
        // access to url parameters.
        translations2: function(translations, $stateParams){
            // Assume that getLang is a service method
            // that uses $http to fetch some translations.
            // Also assume our url was "/:lang/home".
            return translations.getLang($stateParams.lang);
        },
        // Example showing returning of custom made promise
        greeting: function($q, $timeout){
            var deferred = $q.defer();
            $timeout(function() {
                deferred.resolve('Hello!');
            }, 1000);
            return deferred.promise;
        }
    },
    // 控制器将等待上面的解决项都被解决后才被实例化
    controller: function ($scope, user, userData, userFiltered, detailData) {
        $scope.name = user.name;
        $scope.age = userData.age;
        $scope.user = userFiltered.name;
        $scope.age = detailData.age;
    }
});
```

## 为 $state 对象提供自定义数据
```javascript
// 基于对象和基于字符串定义 state
var home = { 
    name: 'home',
    templateUrl: 'home.html',
    data: {
        name: 'Anani',
        age: 24
    }  
};
$stateProvider
    .state(home)
    .state('home.girlFriend', {
        templateUrl: 'home.girlFriend.html',
        data: {
            name: 'Sharon',
            age: 23
        } 
});
```

们可以通过下面的方式来访问上面定义的数据：

```javascript
function dataCtrl ($state) { 
    console.log($state.current.data.name);
    console.log($state.current.data.age);
}
```

## Abstract States
抽象模板不能被激活，但是它的子模板可以被激活，从而被隐式激活。

在以下情况您可能需要用到它：
 * 为所有子状态 URLs 预置一个 URL。

```javascript
$stateProvider 
    .state('products', {
        abstract: true,
        url: '/products',
 
        // Note: abstract still needs a ui-view for its children to populate.
        // You can simply add it inline here.
        template: ''
    })
    .state('products.list', {
        // url will become '/products/list'
        url: '/list'
        //...more
    })
    .state('products.detail', {
        // url will become '/products/detail'
        url: '/detail',
        //...more
    });
```

 * To insert a template with its own ui-view(s) that its child states will populate.

```javascript
$stateProvider 
    .state('products', {
        abstract: true,
        templateUrl: 'products.html'
    })
    .state('products.list', {
        // loaded into ui-view of parent's template
        templateUrl: 'products.list.html'
    })
    .state('products.detail', {
        // loaded into ui-view of parent's template
        templateUrl: 'products.detail.html'
    });
```

```html
<!-- products.html --> 
<h1>Products Page</h1>
<div ui-view></div>
```

 * 通过 `resolve` 为子状态提供依赖。
 * 通过 `data` 提供自定义数据给子状态或事件侦听器使用。
 * 运行一个 `Onter` 或 `OnEnter` 函数，可以以某种方式修改应用程序。

## 参数列表
 * **name**：状态的名称。
 * **template**： string/function，`html` 模板字符串，或者一个返回 `html` 模板字符串的函数。
 * **templateUrl**：string/function，模板路径的字符串，或者返回模板路径字符串的函数。
 * **templateProvider**：function，返回 `html` 模板字符串或模板路径的服务。
 * **controller**：string/function，新注册一个控制器函数或者一个已注册的控制器的名称字符串。
 * **controllerProvider**：function，返回控制器或者控制器名称的服务
 * **controllerAs**：string，控制器别名。
 * **parent**：string/object，手动指定该状态的父级。
 * **resolve**：object，将会被注入 `controller` 去执行的函数，`<string,function>` 形式。
 * **url**：string，当前状态的对应url。
 * **views**：object，视图展示的配置。`<string,object>` 形式。
 * **abstract**：boolean，一个永远不会被激活的抽象的状态，但可以给其子级提供特性的继承。默认是 `true`。
 * **onEnter**：function，当进入一个状态后的回调函数。
 * **onExit**：function，当退出一个状态后的回调函数。
   * **reloadOnSearch**：boolean，如果为 `false`，那么当一个 `search/query` 参数改变时不会触发相同的状态，用于当你修改 $location.search() 的时候不想重新加载页面。默认为 `true`。
 * **data**：object，任意对象数据，用于自定义配置。继承父级状态的 `data` 属性。换句话说，通过原型继承可以达到添加一个 `data` 数据从而整个树结构都能获取到。
 * **params**：url里的参数值，通过它可以实现页面间的参数传递。

<p class="text-right">
    如需了解更多信息，点击查看 <a href="./angular-ui-router-2.html">UI-Router for AngularJS (二)</a>
</p>

## 参考资料
 * [https://ui-router.github.io/](https://ui-router.github.io/)
 * [https://github.com/angular-ui/ui-router/wiki](https://github.com/angular-ui/ui-router/wiki)
 * [http://bubkoo.com/2014/01/02/angular/ui-router/guide/index/](http://bubkoo.com/2014/01/02/angular/ui-router/guide/index/)