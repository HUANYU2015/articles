# [vue-router](https://router.vuejs.org/zh/guide/advanced/scroll-behavior.html#%E5%BC%82%E6%AD%A5%E6%BB%9A%E5%8A%A8)

## 安装

### 直接下载/CDN

### NPM

使用下面的命令或是使用vue-cli自动安装

~~~ xml
npm install vue-router
~~~

如果在一个模块化工程中使用它，必须要通过Vue.use()明确地安装路由功能：

~~~ javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)
~~~

### 构建开发板

如果你想使用最新的开发版，你可以从GitHub上直接clone，然后自己build一个vue-router

~~~ xml
git clone https://github.com/vuejs/vue-router.git node_modules/vue-router
cd node_modules/vue-router
npm install
npm run build
~~~

## 介绍

>### 版本说明
>对于TypeScript用户来说，**vue-router@3.0+**依赖**vue@2.5+**，反之亦然。

Vue Router是Vue.js官方的路由管理器。它和Vue.js的核心深度集成，让构建单页面应用变得易如反掌。它具有以下功能：

- 嵌套的路由/视图表
- 模块化、基于组件的路由配置
- 路由参数、查询、通配符
- 基于Vue.js过度系统的视图过度效果
- 细粒度的导航控制
- 带有自动激活的CSS class的链接
- HTML5历史模式或hash模式，在IE9中自动降级
- **自动的滚动条行为**

## 基础

### 起步

使用Vue.js+Vue Router 创建单页应用，非常简单。使用Vue.js，我们可以通过组合组件来构成应用程序，当你要把Vue Router添加进来，我们需要做的是：

- 在/router.js/index.js中将组件（component）映射到路由（routers），即创建路由实例 `VueRouter` ，并定义路由 `routers`
- 在程序入口文件 `main.js` 中通过 `router` 选项注入前面已创建的路由实例
- 在需要使用路由的组件中使用 `<router-link>` 标签的 `to` 属性指向相关路由
- 在页面入口文件（ `App.vue` ）中使用 `<router-view>` 指明路由出口，即告知路由匹配到的组件的渲染位置

通过注入路由器，我们可以在任何组件内通过 `this.$router` 访问路由器，通过 `this.$route` 访问当前路由。

 > 注意：当 `<router-link>` 对应的路由匹配成功，将自动设置class属性值 `.router-link-active` 。

### 动态路由匹配

我们经常需要把某种模式匹配到所有路由，全部映射到同一个组件。例如，有一个 `User` 组件，对于所有ID各不相同的用户，都要使用这个组件来渲染。那么，为了实现这个效果，可以在 `vue-router` 的路由路径中使用 `动态路径参数（dynamic segment）` 。

 ```javascript
 const User = {
     template: '<div>User</div>'
 }

 const router = new VueRouter({
     routers: [
        // 动态路径参数 以冒号开头
         {path: '/user/:id', component: User}
     ]
 })
 ```

现在，向 `/user/foo` 和 `/user/bar` 都将映射到相同的路由。

一个 **路径参数**使用 `:` 标记。当匹配到一个路由时，参数值会被设置到 `this.$route.params` ，并且可以在每个组件中使用。例如，我们可以更新 `User` 的模板，输出当前用户的ID：

 ```javascript
 const User = {
     template: '<div>{{$route.params.id}}</div>'
 }
 ```

你可以在一个路由中设置多段 **路径参数**，对应的值都会设置到 `$route.params` 中。例如：
|模式|匹配路径|$route.params|
|:------|:------|:------|
|/user/:username|/user/evan| `{username: ‘evan’}` |
|/user/:username/post/:post_id|/user/evan/post/123| `{username: 'evan', post_id: '123'}` |

除了 `$route.params` 外， `$route` 对象还提供了其他有用的信息，如 `$route.query` (如果URL中有查询参数)、 `$route.hash` 等等。

#### 响应路由参数的变化

当使用路由参数时，比如从 `/user/foo` 导航到 `/user/bar` ， **原来的组件实例将会被复用**。因为两个路由都渲染同个组件，比起销毁再创建，复用更加高效。 **不过，这个意味着组件的生命周期钩子不会再被调用**。

复用组件时，如需对路由参数的变化做出响应，可以简单地watch(检测变化) `$route` 对象：

~~~ javascript
const User = {
    template: '...',
    watch: {
        '$route' (to, from) {
            // 对路由变化做出响应...
        }
    }
}
~~~

或者使用2.2中引入的 `beforeRouteUpdate` **导航守卫**：

~~~ javascript
const User = {
    template: '...',
    beforeRouteUpdate (to, from, next) {
        // react to route changes...
        // don't forget to call next()
    }
}
~~~

#### 捕获所有路由或者404 Not found路由

常规参数只会匹配被 `/` 分隔的URL片段中的字符。如果想匹配任意路径，可以使用通配符（`*`）：

~~~ javascript
{
    // 会匹配所有路径
    path: '*'
}, {
    // 会匹配以 ‘、user-’开头的任意路径
    path: '/user-*'
}
~~~

使用通配符路由需要确保路由的顺序是正确的，也就是说含有通配符的路由应该放在最后。

路由 `{path: '*'}` 通常用于客户端**404错误**。如果你使用了History模式，需要确保[正确配置服务器](#HTML\ History模式)。

当使用一个通配符时，`$route.params` 会自动添加一个名为 `pathMatch` 参数。它包含了URL通过通配符被匹配的部分：

~~~ javascript
// 给出一个路由 {path: 'user-*'}
this.$router.push('/user-admin')
this.$route.params.pathMatch // ‘admin’
// 给出一个路由 {path: '*'}
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
~~~

#### 高级匹配模式

`vue-router` 使用**path-to-regexp**作为路径匹配引擎，所以支持很多高级匹配模式，如：可选的动态路径参数、匹配零个或多个、一个或多个，甚至是自定义正则匹配。具体可以查看它的[文档](https://github.com/pillarjs/path-to-regexp#parameters)学习高杰的路径匹配，还有[这个例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-matching/app.js)展示 `vue-router` 怎么使用这类匹配。

#### 匹配优先级

有时，同一个路径可以匹配多个路由，此时，匹配的优先级按照路由的定义顺序：谁先定义的，谁的优先级就更高。

### 嵌套路由

<!-- [测试](#进阶) -->


### 编程式导航

### 命名路由

### 命名视图

### 重定向和别名

### 路由组件传参

### HTML History模式

## 进阶

### 导航守卫

### 路由元信息

### 过度动效

### 数据获取

### 滚动行为

使用前端路由，当切换到新路由时，想要页面滚到顶部或者是就像重新加载页面那样保持原先的滚动位置。vue-router能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。

**注意：这个功能只在支持 `history.pushState` 的浏览器中可用。**

当创建一个Router实例，你可以提供一个 `scrollBehavior` 方法：

~~~ javascript
const router = new VueRouter({
    router: [...],
    scrollBehavior (to, from, savedPosition) {
        // return 期望滚动到那个位置
    }
})
~~~

`scrollBehavior` 方法接收 `to` 和 `from` 路由对象。第三个参数 `savedPosition` 当且仅当 `popstate` 导航（通过浏览器的**前进/后退**按钮触发）时才可用。
这个方法返回滚动位置的对象信息，如：

- `{x: number, y: number}`
- `{selector: string, offset? : {x: number, y: number}}`(offset只在2.6.0+支持)

如果返回一个falsy的值，或者是一个空对象，那么不会发生滚动。
例如：

~~~ javascript
scrollBehavior （to, from, savedPosition） {
    return {x: 0, y: 0}
}
~~~

对于所有路由导航，简单地让页面滚动到页面顶部。

返回 `savedPosition` ，在按下**后退/前进**按钮时，就会想浏览器的原生表现那样：

~~~ javascript
scrollBehavior (to, from, savedPosition) {
    if (savedPosition) {
        return savedPosition
    } else {
        return {x: 0, y: 0}
    }
}
~~~

如果你要模拟“滚动到锚点”的行为，则可如下编写代码：

~~~ javascript
scrollBehavior (to, from, savedPosition) {
    if (to.hash) {
        return {
            selector: to.hash
        }
    }
}
~~~

我们还可以利用[路由元信息][1]更细颗粒度地控制滚动。查看完整例子请[移步这里][2]。

[1]:https://segmentfault.com/a/1190000012578301
[2]:https://github.com/vuejs/vue-router/blob/next/examples/scroll-behavior/app.js

### 路由懒加载
