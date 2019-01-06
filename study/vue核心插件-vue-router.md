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
- **自动以滚动条行为**

## 基础

### 起步

### 动态路由匹配

### 嵌套路由

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

**注意：这个功能只在支持`history.pushState`的浏览器中可用。**

当创建一个Router实例，你可以提供一个`scrollBehavior`方法：

~~~ javascript
const router = new VueRouter({
    router: [...],
    scrollBehavior (to, from, savedPosition) {
        // return 期望滚动到那个位置
    }
})
~~~

`scrollBehavior`方法接收`to`和`from`路由对象。第三个参数`savedPosition`当且仅当`popstate`导航（通过浏览器的**前进/后退**按钮触发）时才可用。
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

返回`savedPosition`，在按下**后退/前进**按钮时，就会想浏览器的原生表现那样：

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
