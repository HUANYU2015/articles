# [vue-router](https://router.vuejs.org/zh/guide/advanced/scroll-behavior.html#%E5%BC%82%E6%AD%A5%E6%BB%9A%E5%8A%A8)

## 安装

### 直接下载/CDN

### NPM 

使用下面的命令或是使用vue-cli自动安装

~~~ xml
npm install vue-router
~~~

$检测测试简答支持合适检测终于简要必须知道输出加载$
~~检测~~


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
 run build
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
| 模式                          | 匹配路径            | $route.params                        |
| :---------------------------- | :------------------ | :----------------------------------- |
| /user/:username               | /user/evan          | `{username: ‘evan’}`               |
| /user/:username/post/:post_id | /user/evan/post/123 | `{username: 'evan', post_id: '123'}` |

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

| **声明式**                | **编程式**         |
| :------------------------ | :----------------- |
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

| **声明式**                        | **编程式**            |
| :-------------------------------- | :-------------------- |
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

“重定向”即，当用户访问 `/a` 时，URL将会被替换成 `/b` ，然后匹配路由为 `/b` 。
而别名的路由匹配则保持不变。例如，**`/a` 的别名是 `/b` ，意味着，当用户访问 `/b` 时，URL会保持为 `/b` ，但是路由匹配则为 `/a` ，就像用户访问 `/a` 一样**。

上面对应的路由配置为：

~~~ javascript
const router = new VueRouter({
    routers: [
        {path: '/a', component: A, alias: '/b'}
    ]
})
~~~

借助“别名”，你可以不必受限于配置的嵌套路由结构，而自由地将UI结构映射到任意的URL。
更多高级用法，请[移步例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-alias/app.js)

### 路由组件传参

在组件中，使用 `$route` 会使该组件与其对应的路由形成高度耦合，从而使组件只能在某些特定的URL上使用，限制了组件的灵活使用。

为了解决这个问题，可以使用 `props` 将组件和路由解耦：

**与 `$route` 耦合**

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

**通过 `props` 解耦**

~~~ javascript
const User = {
    props: ['id'],
    template: '<div>User {{id}}</div>'
}

const router = new VueRouter({
    routers: [
        {path: '/user/:id', component: User, props: true},

        // 对于包含命名视图的路由，你必须分别为每个命名视图设置 ‘props’选项：
        {
            path: '/user/:id',
            components: {default: User, sidebar: Sidebar},
            props: {default: true, sidebar: false}
        }
    ]
})
~~~

这样你便可以在任何地方使用该组件，使得该组件更易于重用和测试。

#### 布尔模式

如果 `props` 被设置为 `true` ，`route.params` 将会被设置为组件属性。

#### 对象模式

如果 `props` 是一个对象，它会按原样设置为组件属性。当 `props`是静态（字符串）的时候有用。

~~~ javascript
const router = new VueRouter({
    routers: [
        {path: '/promotion/from-newletter', component: Promotion, props: {newletterPopup: false}}
    ]
})
~~~

#### 函数模式

可以创建一个函数，该函数返回 `props` 。从而便可以将参数转换成另一种类型，比如可以将金泰值与基于路由的参数结合等等。

~~~ javascript
const router = new VueRouter({
    router: [
        {
            path: '/search', component: SearchUser, props: (route) => ({query: route.query.q})
        }
    ]
})
~~~

URL `/search?q=vue` 会将 `{query: 'vue'}` 作为属性传递给 `SearchUser` 组件。

> 注意: 请尽可能保持 `props` 函数为[**无状态**][3]的，因为它只会在路由发生变化时起作用。如果你需要状态来定义 `props` ，请使用**包装组件**，这样Vue才可以对状态变化作出反应。

详细用法，请查看[例子]或下面代码(https://github.com/vuejs/vue-router/blob/dev/examples/route-props/app.js)

~~~ javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import Hello from './Hello.vue'

Vue.use(VueRouter)

function dynamicPropsFn (route) {
  const now = new Date()
  return {
    name: (now.getFullYear() + parseInt(route.params.years)) + '!'
  }
}

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/', component: Hello }, // No props, no nothing
    { path: '/hello/:name', component: Hello, props: true }, // Pass route.params to props
    { path: '/static', component: Hello, props: { name: 'world' }}, // static values
    { path: '/dynamic/:years', component: Hello, props: dynamicPropsFn }, // custom logic for mapping between route and props
    { path: '/attrs', component: Hello, props: { name: 'attrs' }}
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Route props</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/hello/you">/hello/you</router-link></li>
        <li><router-link to="/static">/static</router-link></li>
        <li><router-link to="/dynamic/1">/dynamic/1</router-link></li>
        <li><router-link to="/attrs">/attrs</router-link></li>
      </ul>
      <router-view class="view" foo="123"></router-view>
    </div>
  `
}).$mount('#app')
~~~

### HTML History模式

 参考:</br>
[一](https://juejin.im/post/5a61908c6fb9a01c9064f20a)</br>
[二](https://juejin.im/post/5b31a4f76fb9a00e90018cee)</br>
[三](https://www.jianshu.com/p/3fcae6a4968f)

#### 为什么要有hash 和 history

对于Vue这类渐进式前端开大框架，为了构建SPA（单页面应用），需要引入**前端路由系统**，这也是Vue_router存在的意义。前端路由的核心，就在于--**改变视图的同时不会向后端发出请求**。

为了达到这一目的，当前浏览器提供了一下两种支持：

1. **`hash`**--即地址栏URL中的 `#` 符号（此hash不是密码学里的散列运算)。</br>比如这个URL: `http://www.abc.com/#/hello` ，hash的值为 `#/hello` 。它的特点在于：hash虽然出现在URL中，但不会被包括在HTTP请求中，对后端完全没有影响，因此**改变hash不会重新加载页面**。
2. **`history`**--利用了HTML5 History Interface 中新增的 `pushState()` 和 `replaceState()` 方法（需特定浏览器支持）。</br>这两个方法应用于浏览器的历史记录栈，在当前已有的 `back`、 `forward`、`go` 的基础之上，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然当前URL被改变了，但是浏览器不会立即向后端发送请求。

因此可以说，hash模式和history模式都属于浏览器自身的属性，Vue-Router只是利用了这两个特性来实现前端路由（通过调用浏览器提供的接口）。

#### hash模式

hash模式背后的原理是 `onhashchange` 事件，可以在window对象上监听这个事件：

~~~ javascript
window.onhashchange = function(event) {
    console.log(event.oldURL, event.newURL);
    let hash = location.hash.slice(1);
    document.body.style.color = hash;
}
~~~

上面的diamante可以实现通过改变hash来改变页面字体颜色的功能，可以在一定程度上说明hash的原理。

更关键的一点是，因为hash发生变化的URL都会被浏览器记录下来，从而浏览器的前进后退都可以使用了。同时点击后退时，页面字体颜色也会发生变化。这样一来，尽管浏览器没有请求服务器，页面颜色也会发生变化，但是页面状态和URL一一关联起来了。

#### history模式应用场景

一般场景下，hash 和 history 都可以，除非你更在意颜值，`#` 符号夹杂在URL里看起来确实不太美观。

另外，根据 [Mozilla Develop Network][4]的介绍，调用 `history.pushState()` 相比于直接修改 `hash` ，存在以下优势：

- `pushState()` 设置的新URL可以使与当前URL**同源**的任意URL；而 `hash` 只可修改 `#` 后面的部分，因此只能设置与当前URL**同文档**的URL；
- `pushState()` 设置新的URL可以与当前URL一模一样，这样也会把记录添加到栈中；而 `hash` 设置的新值必须与原来不一样才会触发动作将记录添加到栈中；
- `pushState()` 通过 `stateObject` 参数可以添加任意类型的数据到记录中；而 `hash` 只可添加端字符串；
- `pushState()` 可额外设置 `title`属性供后续使用。

当然，`history` 模式也不是没有缺陷。SPA虽然在浏览器中游刃有余，但是当真要通过URL向后端发起HTTP请求时，两者的差异就表现出来了。尤其在用户手动输入URL后回车，或者刷新（重启）浏览器的时候。

1. `hash` 模式下，仅 `hash` 符号之前的内容会被包含在请求中，如 `http://www.abc.com` ，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回404错误。
2. `history` 模式下，前端的URL必须和实际向后端发起请求的URL一致，如 `http://www.abc.com/book/id`。如果后端缺少对 `/book/id` 的路由处理，将返回404错误。

因此，Vue-Router官网里如此描述：“**不过这种模式要玩好，还需要后台配置支持**......所以你要在服务端增加一个覆盖所有情况的候选资源：如果URL匹配不到任何静态资源，则应该返回同一个index.html页面，这个页面就是你APP依赖的页面。”

#### 后端配置例子

// todo 待续！*****待续*****

$nginx$

$原生Node.js$

$基于Node.js的Express$

$InternetInformation Services（IIS）$

$Caddy$

$Firebase主机$

#### 警告⚠️

给个警告，因为这么做以后，服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。具体代码如下：

~~~ javascript
const router = new VueRouter({
    mode: 'history',
    routers: [
        {path: '*', component: NotFoundComponent}
    ]
})
~~~

或者，如果你使用 Node.js 服务器，你可以用服务端路由匹配到来的 URL，并在没有匹配到路由的时候返回 404，以实现回退。更多详情请查阅 [Vue 服务端渲染文档][5]。

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
[3]:https://endual.iteye.com/blog/1340359
[4]:https://developer.mozilla.org/zh-CN/docs/Web
[5]:https://ssr.vuejs.org/zh/

### 路由懒加载
