# UI-Router for AngularJS (一)
UI-Router is the defacto standard for routing in AngularJS.

## 原生路由
如果使用原生路由的话，`Angular` 的视图是通过 `ng-view` 这个指令进行加载的。比如：`<div ng-view></div>`，通常，我们都会把这个指令放在 `index.html` 这个文件里面。

然后具体的路由信息在 `javascript` 文件中进行配置：

```html
// 引入文件 
<script type="text/javascript" src="JS/angular.min.js"></script>
<script type="text/javascript" src="JS/angular-ui-router.min.js"></script>
 
// 配置路由
<script type="text/javascript">
angular.module('app', [])
    .config('AppCtrl', ['$routeProvider', function ( $routeProvider ) {
        $routeProvider
            .when('/', {
                templateUrl: 'views/page-a.html',
                controller: 'PageACtrl'
            })
            .when('/next', {
                templateUrl: 'views/page-b.html',
                controller: 'PageBCtrl'
            })
 
        /* 对其他不合法的路由进行处理 */
        $routeProvider.otherwise('/404');
    }]);
</script>
```

原生路由由于不支持嵌套路由，所以每次切换路由时都会刷新整个页面，因此，`ui-router` 插件应运而生。

## UI router 简单使用
 * 引入文件

```javascript
// routine web page? 
<script type="text/javascript" src="JS/angular.min.js"></script>
<script type="text/javascript" src="JS/angular-ui-router.min.js"></script>
```

```javascript
// webpack + angular? 
// main.js-入口文件
import Angular from 'angular'; // 引入angular
import uiRoute from 'angular-ui-router'; // 引入 angular-ui-router
```

 * 注入依赖

```javascript
// routine web page? 
var app = angular.module('app', ['ui.router']);
```

```javascript
// webpack + angular? 
// main.js-入口文件
let App = Angular.module('app', [uiRoute]);
```

 * 配置路由

```javascript
// routine web page? 
app.config(["$stateProvider", function ($stateProvider){
    $stateProvider     
    .state("home", { // 路由名
        url: '/',    // 路径 
        template: '<div>Hello world</div>'
    })      
}]);
```

```javascript
// webpack + angular? 
// JS/route.js    
function router ($stateProvider, $urlRouterProvider) {
    // $urlRouterProvider-路由重定向
    $urlRouterProvider.when('', '/index'); // 对非法路由进行重定向
    $stateProvider
        .state('home', {    // 路由名
            url: '/index',  // 路径
            template: '<div>Hello world</div>', // 模板
        })
    $urlRouterProvider.otherwise('/page-404');
}
export default router;
 
// main.js
import router from 'JS/index.js'; // 引入路由配置
App.config(['$stateProvider', '$urlRouterProvider', router]); // 配置路由
```

 * 定义视图

```html
<!-- routine web page? or webpack + angular? --> 
<div ui-view></div>
```

 * 使用路由

```javascript
// .js 中
var stateName = 'home'; // 值为在路由配置中设置的路由名
$state.go(stateName);
```

```html
<!-- .html 中 -->
<a ui-sref="stateName">跳转</a>
<!-- 值依旧为在路由配置中设置的路由名 -->
```

<p class="text-right">
    如需了解更多信息，点击查看 <a href="./angular-ui-router-1.html">UI-Router for AngularJS (一)</a>
</p>

## 参考资料
 * [https://ui-router.github.io/](https://ui-router.github.io/)
 * [http://hao.jobbole.com/angular-ui-router/](http://hao.jobbole.com/angular-ui-router/)
 * [https://blog.csdn.net/shenlei19911210/article/details/51325707/](https://blog.csdn.net/shenlei19911210/article/details/51325707/)
 * [https://blog.csdn.net/zcl_love_wx/article/details/52034193](https://blog.csdn.net/zcl_love_wx/article/details/52034193)