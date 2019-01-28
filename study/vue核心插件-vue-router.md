# vue-router

## 安装

---

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
run build
~~~

## 介绍

---

> $版本说明$
> 对于TypeScript用户来说，**vue-router@3.0+**依赖**vue@2.5+**，反之亦然。

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

---

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

> 注意[导航守卫](#导航守卫)并没有应用在跳转路由(from)上，而仅仅应用在其目标路由上(to)。在下面这个例子中，为 /a 路由添加一个 beforeEach 或 beforeLeave 守卫并不会有任何效果。
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

如果`props`被设置为 `true` ，`route.params` 将会被设置为组件属性。

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

详细用法，请查看[例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-props/app.js)或下面代码

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
  template: '
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
  '
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

// 下面是一些针对未匹配到静态资源后，返回index.html页面的设置。

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

---

### 导航守卫

> 导航表示路由正在发生改变。

正如其名， `vue-router` 提供的导航守卫主要通过跳转或取消的方式用来守卫导航。并且有多种方式植入导航过程中，比如全局的、单路由独享的、或者组件级的。

注意，参数或查询的改变并不会进入或离开导航守卫。你可以通过观察 `$route` 对象来应对这些变化，或者使用 `beforeRouteUpdate` 组件内守卫。

#### 全局前置守卫

可以使用 `router.beforeEach` 注册全局前置守卫：

~~~ javascript
const router = new VueRouter({...})

router.beforeEach((to, from, next) => {
    // ...
})
~~~

当一个导航被触发时，全局前置守卫按照其创建顺序被依次调用。导航是异步解析执行的，并且导航在所有守卫(hooks)被解析(resolve)完成前一直处于等待中。

每个守卫函数接收三个参数：

- **`to: Route`** : 将被导航至的目标路由对象。
- **`from: Route`** : 当前路由对象，将要因导航而离开该路由。
- **`next: Function`** : 必须要调用该函数来解析这个钩子(hook)，其执行效果依赖于提供给该 `next` 函数的参数：
  - **`next()`** : 进行传输线路(pipeline)中的下一个钩子(hook)，如果所有钩子都执行完毕，则导航的状态就是确认的(confirmed)；
  - **`next(false)`** : 中断当前导航。如果浏览器URL被改变了(可能是用户手动操作或点击回退按钮)，那么导航目标(URL地址)重置为 `from` 路由对应的地址;
  - **`next('/')` 或 `next({path: '/'})`** : 重定向到一个不同的位置。当前导航将被中断，然后开始一个新的导航，你可以向 next 函数传递任意位置对象，</br>且允许设置诸如：`replace: true`，`name: 'home'`之类的选项，以及任意在 `router-link` 组件的 `to` 属性及 `router.push` 方法中使用的选项；
  - **`next(error)`** : (2.4.0+)如果传入 `next` 函数的参数是 `Error` 的实例，导航将被中止且错误会传递给经由 `router.onError()` 方法注册的回调函数(callbacks)。

**要确保调用 `next` 函数，否则守卫(hook)将不会被解析，可查看[路由元信息](#路由元信息)中的示例**。

#### 全局解析守卫

> 2.5.0新增

你可以用 `router.beforeResolve` 注册一个全局守卫。这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。

#### 全局后置钩子

你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 next 函数，也不会改变导航本身：

~~~ javascript
router.afterEach((to, from) => {
    // ...
})
~~~

#### 路由独享守卫

你可以直接在路由的配置对象中定义 `beforeEnter` 守卫：

~~~ javascript
const router = new VueRouter({
    routers: [
        {
            path: './foo',
            component: Foo,
            beforeEnter: (to, from, next) => {
                // ...
            }
        }
    ]
})
~~~

这些守卫和全局前置守卫拥有一样的方法签名(signature)，即方法参数。

#### 组件内守卫

你还可以在组件内部直接定义一下路由导航守卫：

- beforeRouterEnter
- beforeRouterUpdate (2.2新增)
- beforeRouterLeave

~~~ javascript
const Foo = {
    template: '...',
    beforRouterEnter (to, from, next) {
        // 在渲染该组件的对应路由被 confirm 前调用
        // 不能 获取组件实例 'this'
        // 因为当守卫执行前，组件实例还未被创建
    },
    beforeRouterUpdate (to, from, next) {
        // 在当前路由被改变，但该组件被复用时被调用
        // 例如，对于一个带有动态参数的路径：/foo/:id，在/foo/1 和 /foo/2 之间跳转的时候
        // 由于会渲染同样的 Foo 组件，因此组件实例会被复用
        // 可以访问组件实例 ‘this’
    },
    beforeRouterLeave (to, from, next) {
        // 导航离开该组件的对应路由时调用
        // 可以访问组件实例 'this'
    }
}
~~~

`beforeRouterEnter` 守卫**不能**访问 `this`，因为守卫在导航确认前被调用，因此即将登场的新组建还未被创建。

不过，你可以通过传一个回调方法给 next 来访问组件实例，在导航被确认的时候执行回调，且组件实例会作为一个参数传递给回调方法：

~~~ javascript
beforeRouterEnter (to, from, next) {
    next(vm => {
        // 通过 ‘vm’ 访问组件实例
    })
}
~~~

> 注意： `beforeRouterEnter` 是支持给 next 传递回调的位移守卫。对于 `beforeRouterUpdate` 和 `beforeRouterLeave` 来说，`this` 已经可用了，所以不支持传递回调，因为没有必要了。

~~~ javascript
beforeRouterUpdate (to, from, next) {
    // just use ‘this’
    this.name = to.params.name,
    next()
}
~~~

这个离开守卫通常用来禁止用户在还未保存修改前突然离开当前页面(URL地址)。该导航可以通过 `next(false)` 来取消。

~~~ javascript
beforeRouterleave (to, from, next) {
    const answer = window.confirm('Do you really want to leave? you have unsaved changges!')
    if (answer) {
        next()
    } else {
        next(false)
    }
}
~~~

#### 完整的导航解析流程

🦋🍎🐸

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouterLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouterUpdate` 守卫(2.2+)
5. 在路由配置里调用 `beforeEnter` 守卫。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouterEnter`。
8. 调用全局的 `beforeResolve` 守卫(2.5+)。
9. 导航被确认(confirmed)。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 `beforeRouterEnter` 守卫中传给 next 的回调函数。

### 路由元信息

🦋 注意与导航守卫的联系。

vue-router路由元信息简单说，就是通过meta对象中的一些属性设定来判断当前路由是否需要进行相关处理。

在路由列表中，我们可以给每个路由记录配置一个数据字段，一般叫做 `meta`，我么可以设置一些自定义信息，供页面组件或者路由钩子函数使用。例如，我们在浏览一些网站的时候，有时会验证用户是否登录，但并不是所有情况都需要验证的，所以，我们就可以通过在路由记录中添加路由元数据来进行区分。

定义路由的时候可以配置 `meta` 字段：

~~~ javascript
const router = new VueRouter({
    routers: [
        {
            path: '/foo',
            component: Foo,
            children: [
                {
                    path: '/bar',
                    component: Bar,
                    // a meta filed
                    meta: {requiresAuth: true}
                }
            ]
        }
    ]
})
~~~

那么如何访问这个 `meta` 字段呢？

首先，我们称呼 `router` 配置中的每个路由对象为**路由记录**。路由记录是可以嵌套的，因此，当一个路由匹配成功后，它可能匹配多个路由记录。

例如，更具上面的路由配置，`/foo/bar` 这个URL将会匹配父路由记录和子路由记录。

一个路由匹配到的所有路由记录都会暴露为由 `$route` 对象（还有在导航守卫中的路由对象）组成的 `$route.matched` 数组。因此，我们需要遍历该数组来检查路由记录中的 `meta` 字段，在里了我们使用 `some()` 函数来遍历数组，并用到了导航守卫的全局守卫中的`beforeEach` 方法。

~~~ javascript
// 在路由更新之前去遍历路由记录的meta元信息，判断是否存在requiresAuth属性
router.beforeEach((to, from, next) => {
    if (to.matched.some(record => record.meta.requiresAuth)) {
        // 如果存在 requiresAuth 属性，则据需判断是否 logged in，
        // 如果为否，则重定向到登录页面，/login?redirect=to.fillPath
        if (!auth.loggedIn()) {
            next({
                path: '/login',
                query: {redirect: to.fullPath}
            })
        } else {
            next()
        }
    } else {
        // 确保一定要在此处调用 next()方法
        next()
    }
})
~~~

$解析：$

- meta字段就是路由元信息字段，requiresAuth是任意取的字段名称，用来标记这个路由信息是否需要检测，true 表示需要检测；
- `if (to.matched.some(record => record.meta.requiresAuth))`，使用了ES6的箭头函数语法，`to.matched`即为$route对象数组，单个路由对象我们定义为 `record` ，并检测这个路由对象（记录）是否拥有 `meta` 选项，如果有，再检测该选项是否拥有 `requiresAuth`这个属性，并且是否为 `true`，如果满足以上条件，就确定了该 `/foo/bar` 路由；
  - function (i) {return i + 1;} // ES5
  - (i) => i + 1 // ES6
- `if (!auth.loggedIn())`，通过自己封装的检验方法 `auth.loggedIn()`，检验用户是否登录，从而决定渲染下一步操作；
- Array some()方法
  - some() 方法用于检测数组中的元素是否满足指定条件（函数提供）；
  - some() 方法会依次执行数组的每个元素，如果有一个元素满足条件，则表达式返回true，剩余的元素不会再执行检测，如果没有满足条件的元素，则返回false；
  - some() 不会对空数组进行检测，且不会改变原始数组；
  - 语法：array.some(function(currentValue,index,arr),thisValue)
  - 参数说明：
    - currentValue：必选，当前元素的值；
    - index：可选，当前元素的索引值；
    - arr：可选，当前元素属于的数组对象；
    - thisValue：可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。如果省略了 thisValue ，"this" 的值为 "undefined"。
  - 返回值：布尔值。如果数组中有元素满足条件返回 true，否则返回 false。

#### 应用场景

- 在页面跳转之前（使用到beforEach()方法）改变页面标题（title）或者检查用户是否已登录（一般在用户注册、完善用户信息及重置密码等活动后向相关页面跳转时，需要检测），详细信息请查看[这里][6]和[这里][7]；
- 在页面跳转之后用于实现返回到页面之前浏览过的区域或者让页面返回到顶部功能（需要借助afterEach()方法），代码见[这里][8]。

### 过度动效

`<router-view>` 是基本的动态组件，所以我们可以用 `<transition>` 组件给它添加一些过渡效果：

~~~ javascript
<transition>
   <router-view></router-view>
</transition>
~~~

[Transition 的所有功能][13] 在这里同样适用。

#### 单个路由的过度

#### 基于路由的动态过度

### 数据获取

有时候，进入某个路由后，需要从服务器获取数据。例如，在渲染用户数据时，你需要从服务器获取用户数据。我们可以通过以下两种方式来实现：

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据，在数据获取期间显示“加载中”之类的提示。
- **当行完成之前获取**：导航完成前，在路由进入(to)的守卫中获取数据，在数据获取成功后执行导航。

从技术角度将，两种方式都不错——就看你想要的用户体验是哪种。

#### 导航完成后获取数据

#### 导航完成前获取数据

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

### 路由懒加载

当打包构建应用时，JavaScript包会变得非常大，从而影响页面的加载速度。如果我们能够把不同的路由对应的组件分割成不同的代码块，然后当路由被访问时采加载对应的组件，这样就变得高效了。

为了满足上面的要求，需要结合Vue的[异步组件][9]和Webpack 的[代码分割功能][10]。

首先，需要将异步组件定义为一个工厂函数，并且该工厂函数返回一个Promise（该函数返回的Promise应是resolve组件本身）：

~~~ javascript
const Foo = () => Promise.resolve({/*组件定义对象*/})
~~~

第二，在Webpack2中，我们可以使用[动态 import][11]语法来定义代码分块点（split point）：

~~~ javascript
import('./Foo.vue')
~~~

将前面两步相结合，即为如下代码，就定义了一个能够被Webpack自动进行代码分割的异步组件：

~~~ javascript
const Foo = () => import('./Foo.vue')
~~~

然后在路由router实例中直接引用 `Foo`，或者直接在router实例中使用 `import` 语法:

~~~ javascript
const router = new VueRouter({
    routers: [{
        path: '/foo',
        component: () => import('./Foo.vue')
        }
    ]
})
~~~

#### 按组分块

有时候，我们需要将同个路由下的所有组件装入同一个**异步块**(async chunk)中。为此，我们需要使用[**命名块**][12](named chunks)，命名块通过使用一种特殊的 **`注释语法`** 来提供一个**块名称**(chunk name)(webpack版本需要高于2.4)，具体代码如下：

~~~ javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
~~~

这样webpack会将具有同一**块名称**(chunk name)的所有异步模块(组件)装入同一个**异步块**(async chunk)中。

TODO 待续！见[7]

[1]:https://segmentfault.com/a/1190000012578301
[2]:https://github.com/vuejs/vue-router/blob/next/examples/scroll-behavior/app.js
[3]:https://endual.iteye.com/blog/1340359
[4]:https://developer.mozilla.org/zh-CN/docs/Web
[5]:https://ssr.vuejs.org/zh/
[6]:http://www.hangge.com/blog/cache/detail_2130.html
[7]:https://juejin.im/post/5b73a50df265da27f7590737
[8]:https://github.com/vuejs/vue-router/blob/next/examples/scroll-behavior/app.js
[9]:https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6
[10]:https://doc.webpack-china.org/guides/code-splitting-async/#require-ensure-/
[11]:https://github.com/tc39/proposal-dynamic-import
[12]:https://webpack.js.org/guides/code-splitting-async/#chunk-names
[13]:https://cn.vuejs.org/guide/transitions.html
