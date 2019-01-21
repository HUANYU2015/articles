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
实际应用界面，通常由多层嵌套的组件组合而成。同样，URL中各段动态路径也按某种结构对应嵌套的各层组件，如：

~~~ javascript
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
~~~

借助 `vue-router` ，使用嵌套路由配置，就可以很简单地表达这种关系。

接着上节创建的app:

~~~ html
<div id="app">
  <router-view></router-view>
</div>
~~~

~~~ javascript
const User = {
    template: '<div>User {{$route.params.id}}</div>'
}

const router = new VueRouter({
    routers: [
        {path: '/user/:id', component: User}
    ]
})
~~~

这里的 `<router-view>` 是最顶层的出口，渲染最高级路由匹配到的组件。同样的，一个被渲染的组件也可以包含自己的嵌套 `<router-view>` 。例如，在 `User` 组件的模板中添加一个 `<router-view>` ：

~~~ javascript
const User = {
    template: '
        <div class="user">
            <h2>User {{$route.params.id}}</h2>
            <router-view></router-view>
        </div>
    '
}
~~~

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

~~~ javascript
const router = new VueRouter({
    router: [
        {
            path: '/user/:id', component: User,
            children: [
                {
                    // 当 /user/:id/profile匹配成功
                    // UserProfile会被渲染在User的<router-view>中
                    path: 'profile',
                    component: UserProfile
                }, {
                    // 当/user/:id/posts匹配成功
                    // UserPosts会被渲染在User的<router-view>中
                    path: 'posts',
                    component: UserPosts
                }
            ]
        }
    ]
})
~~~

> 注意：以 `/` 开头的嵌套路径会被当做**根路径**。这让你充分地使用嵌套组件，而无须设置嵌套路径。

`children` 配置就像 `routers` 配置一样的路径配置参数，所以可以嵌套多层路由。

此时，基于上面的配置，当访问 `/user/foo` 时，User的出口是不会渲染任何东西的，因为没有匹配到合适的子路由。如果想要渲染点什么，可以提供一个空的子路由：

~~~ javascript
const router = new VueRouter({
    routers: [
        {
            path: '/user/:id', component: User,
            children: [
                // 当 /user/:id 匹配成功，
                // UserHome 会被渲染在 User 的<router-view>中
                {path: '', component: UserHome}

                // ...其他子路由
            ]
        }
    ]
})
~~~

提供以上案例的可运行代码请[**移步这里**](https://jsfiddle.net/yyx990803/L7hscd8h/)。

### 编程式导航

除了使用 `<router-link>` 创建a标签来定义导航链接，还可以借助router的实例方法，通过编写代码来实现。

#### router.push(location, onComplete?, onAbort?)

> 注意：在Vue实例内部，可以通过 `$router` 访问路由实例。因此你可以调用 `this.$router.push` 。

想要导航到不同的URL，则使用 `router.push` 方法。这个方法会向history栈添加一个新的记录📝。所以，当用户点击浏览器后退按钮时，则回到之前的URL。

当你点击 `<router-link>` 时，这个方法会在内部调用，故点击 `<router-link :to="...">` 等同与调用 `router.push(...)` 。

|**声明式**|**编程式**|
|:--------|:--------|
| `<router-link :to="...">` | `router.push(...)` |

该方法的参数可以是一个字符串路径，或者一个描述地址的对象。例如：

~~~ javascript
// 字符串 -> /home
router.push('home')

// 对象 -> /home
router.push({path: 'home'})

// 命名的路由 -> /user/123
router.push({name: 'user', params: {userId: '123'}})

// 带查询参数， -> /register?plan=private
router.push({path: 'register', query: {plan: 'private'}})
~~~

> 注意：如果提供了 **`path`**，**`params`** 会被忽略（上述例子中的 `query` 不属于这种情况）。此外，还有以下一些方式作为替代：

~~~ javascript
const userId = '123'
router.push({name: 'user', params: {userId}}) // -> /user/123
router.push({path: '/user/${userId}'}) // -> /user/123

// 这里的 params 不生效
router.push({path: '/user', params: {userId}}) // -> /user
~~~

同样的规则也适用于 `router-link` 组件的 `to` 属性。

在2.2.0+，函数 `router.push` 和 `router.replace` 中，提供的 `onComplete` 和 `onAbort` 回调作为第二、第三个参数。前者将会在导航成功完成时（在所有的异步钩子被解析之后）进行相应的调用，后者在导航成功终止时（导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由）进行相应的调用。

#### router.replace(location, onComplete?, onAbort?)

跟 `router.push` 很像，位移的不同就是，它不会像history提价新纪录，而是替换掉当前的history记录。

|**声明式**|**编程式**|
|:--------|:--------|
| `<router-link :to="..." replace>` | `router.replace(...)` |

#### router.go(n)

这个方法的参数是一个整数，作用是在history记录中向前后向后移动n步，类似 `window.histroy.go(n)` 。

~~~ javascript
// 在浏览器记录中前进一步，等同于history.forword()
router.go(1)

// 后退一步记录，等同于history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果history记录不够用，就默默地失败
router.go(-100)
router.go(100)
~~~

#### 操作History

注意 `router.push` `router.replace` 和 `router.go` 与`window.histroy.pushState` `window.history.replaceState` 和  `window.history.go` 比较相像，实际上它们确实四效仿 `window.history` **API**的。

此外，Vue Router 的导航方法（push/replace/go）在各类路由模式（`history` `hash`和`abstact`）下表现一致。

### 命名路由

有时候，通过一个名称来识别一个路由显得更方便一些，特别是在链接一个路由，或者执行一些跳转的时候。可以在创建Router实例的时候，在 routers 配置中给某个路由设置名称。

~~~ javascript
const router = new VueRouter({
    routers: [
        {
            path: '/user/:userId',
            name: 'user',
            component: User
        }
    ]
})
~~~

要链接一个命名路由，可以给 `router-link` 的 `to` 属性传一个对象，该对象包含一个 name 属性：

~~~ javascript
// -> /user/123
<router-link :to="{name: 'user', params: {userId: 123}}">User</router-link>
~~~

也可以使用代码实现：

~~~ javascript
router.push({name: 'user', params: {userId: 123}})
~~~

这两种方式都会把路由导航到 `/user/123` 路径。

完整例子请[**移步这里**](https://github.com/vuejs/vue-router/blob/dev/examples/named-routes/app.js)。

### 命名视图

有时候需要同时（同级）展示多个视图，而不是嵌套显示，例如创建一个页面，其布局有 sidebar （侧导航）和 main （主内容）两个视图，这个时候命名视图（添加 `name` 属性）就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是单独一个出口。默认（`default`）视图（router-view），可无需设置name属性。代码如下：

~~~ html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
~~~

单个视图使用单个组件渲染，因此对于`同一个路由`，多个视图就需要多个组件。确保正确使用 `components` 配置（复数）

~~~ javascript
const router = new VueRouter({
    routers: [
        {
            path: '/',
            components: {
                default: Foo,
                a: Bar,
                b: Baz
            }
        }
    ]
})
~~~

以上案例相关可运行代码请[**移步这里**](https://jsfiddle.net/posva/6du90epg/)

#### 嵌套命名视图

当页面布局比较复杂时，可能需要使用命名视图创建视图嵌套。需要结合嵌套路由和命名视图来实现。以一个设置面板为例：

~~~ javascript
/settings/emails                                       /settings/profile
+-----------------------------------+                  +------------------------------+
| UserSettings                      |                  | UserSettings                 |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +------------>  | | Nav | UserProfile        | |
| |     +-------------------------+ |                  | |     +--------------------+ |
| |     |                         | |                  | |     | UserProfilePreview | |
| +-----+-------------------------+ |                  | +-----+--------------------+ |
+-----------------------------------+                  +------------------------------+
~~~

`Nav` 只是一个常规组件。
`UserSettings` 是一个视图组件。
`UserEmailSubscriptions`、`UserProfile`、`UserProfilePreview` 是嵌套的视图组件。

`UserSettings` 组件代码如下：

~~~ javascript
const UserSettings = {
    template: `
    <div class="us">
        <h2>User Settings</h2>
        <UserSettingsNav/>
        <router-view class ="us__content"/>
        <router-view name="helper" class="us__content us__content--helper"/>
</div>
    `,
  components: { UserSettingsNav }
}
~~~

然后通过路由配置完成布局：

~~~ javascript
const router = new VueRouter({
    mode: 'history',
    routers: [
        {
            path: '/settings',
            component: UserSettings,
            children: [{
                path: 'emails',
                component: UserEmailSubscriptions
            }, {
                path: 'profile',
                components: {
                    default: UserProfile,
                    helper: UserProfilePreview
                }
            }]
        }
    ]
})
~~~

一个完整的示例请[**移步这里**](https://jsfiddle.net/posva/22wgksa3/)

### 重定向和别名

#### 重定向

重定向也是通过 `routers` 配置来完成的，下面的例子是从 `/a` 重定向到 `/b` ：

~~~ javascript
const router = new VueRouter({
    routers: [
        {path: '/a', redirect: '/b'}
    ]
})
~~~

重定向的目标也可以是一个命名的路由：

~~~ javascript
const router = new VueRouter({
    routers: [
        {path: '/a', redirect: {name: 'foo'}}
    ]
})
~~~

甚至是一个方法，该方法返回重定向目标：

~~~ javascript
const router = new VueRouter({
    routers: [
        {path: '/a', redirect: to => {
            // 方法接收 目标路由 作为参数
            // return 重定向的 字符串路径或路径对象
        }}
    ]
})
~~~

> 注意[导航守卫](#导航守卫)并没有应用在跳转路由上，而仅仅应用在其目标上。在下面这个例子中，为 /a 路由添加一个 beforeEach 或 beforeLeave 守卫并不会有任何效果。
>
> 其它高级用法，请参考[**例子**](https://github.com/vuejs/vue-router/blob/dev/examples/redirect/app.js)。

#### 别名

TODO 待续

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
