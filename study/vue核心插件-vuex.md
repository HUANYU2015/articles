# vuex

## 参考

[知乎][a]

[官网vuex][b]

[a]:https://zhuanlan.zhihu.com/p/25701238
[b]:https://vuex.vuejs.org/zh/

## 安装

我们在使用Vue-Cli这个脚手架工具搭建vue项目时，会自动帮助我们配置好NPM的package.json文件，这个文件里包含了这个Vue项目所用依赖的包。

但是默认情况下，这个脚手架工具并没有为我们将Vuex这个依赖包包含进去，所以需要我们手动`声明`一下我们这个vue项目中需要依赖Vuex这个包。

通常有两种方法：

- 直接修改package.json
- 使用下面的命令声明

~~~ shell
npm install vuex --save
~~~

> 上面的命令表示安装vuex这个包，`--save`表示将这个依赖包与本项目的以依赖关系写入package.json中。

安装好vuex这个依赖包后，我们还需要在项目入口文件`main.js`开头添加一下两行代码：

~~~ javascript
import Vuex from 'vuex'
Vue.use(Vuex)
~~~

第一行使用到了ECMAScript语法的import将vuex包导入进来；
第二行是vue本身的插件注入语法，将插件注入vue的目的是方便我们在组件内部调用它。

## 介绍

### vuex是什么？

vuex是一个专为Vue.js应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所用组件状态，并以相应的规则保证状态以一种**可预测**的方式发生变化。Vuex也集成到Vue的官方调试工具`devtools extension`，提供了诸如零配置的`time-travel`调试、状态快照导入导出等高级调试功能。

即就是，Vuex就是一个用于管理SPA项目中状态的开源产品。

### 什么是状态？为什么要去管理它？

在传统的Vue.js组件开发中，一般可变状态都是分散在各个组件中，如果需要这些状态的值，那就需要分别去不同的组件中取状态值，然后进行状态值的修改，最后还要互相读取对方的状态值。如果他们本身是父子组件，那么还可以通过事件触发或者props属性来传递状态，但是如果是不同的组件，由于Vue.js本身组件之间有组用于，它们无法直接相互通信，所以就需要一些工具如vuex去集中管理和追踪它的变化。一般这些状态以变量的形式保存在内存中，要在页面显示这些状态值，就需要用到Vue.js这种MVVM框架将状态同步到视图中。

### 什么情况下我们应该使用Vuex？

虽然Vuex可以帮助我们管理共享状态，但是也附带了更多的概念和框架。这需要对短期和长期效益进行权衡。

如果你不打算开发大型单页应用（Single Page WebApplication，SPA），使用Vuex可能是繁琐冗余的。所以如果你的应用够简单，并且他们之间并不需要大量频繁的互相读写操作，最好不要使用Vuex，一个简单的`global event bus`就足够你所需了。但是如果你需要构建一个中大型单页应用，你很有可能会考虑如何更好地在组件外部管理状态，Vuex将会成为你自然而然的选择。

## 开始

每一个Vuex应用的核心就是store（仓库）。”store”基本上就是一个容器，它包含着你的应用中的大部分的`状态（state）`。Vuex和单纯的全局对象有以下两点不同：

- Vuex的状态存储是响应式的。当Vue组件从store中读取状态的时候，若store中的状态发生变化，那么相应的组件也会相应地得到高效更新。
- 你不能直接改变store的状态，改变store中的状态的唯一途径就是显式地`提交(commit) mutation`。这样使得我们可以方便地跟踪每一个状态的改变，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

### 最简单的Store

安装Vuex之后，让我们来创建一个store。创建过程直接了当，仅需要提供一个初始state对象和一些mutation：

~~~ javascript
// 如果在模块化构建系统中，请确保在开头调用了Vue.use(Vuex)
const store = new Vuex.Stroe({
    state: {
        count: 0
    },
    mutations: {
        increment (state) {
            state.count++
        }
    }
})
~~~

现在你可以通过`store.state`来获取状态对象，以及通过`store.commit`方法触发状态变更：

~~~ javascript
store.commit('increment')
console.log(store.state.count) // -> 1
~~~

再次强调，我们通过提交`mutation`的方式，而非直接改变`store.state.count`，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。

由于store中的状态时响应式的，在组件中调用store中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的methods中提交mutation。

## 核心概念

### State

#### 单一状态树

Vuex使用`单一状态树🌲`，用一个对象就包含了全部应用层级状态。至此它便作为一个“唯一数据源（SSOT）”而存在。这也就意味着每个应用仅仅包含一个store实例。单一状态树🌲让我们能够直接定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。

#### 在Vue组件中获得Vuex状态

那么我们如何在Vue组件中展示状态呢？由于Vuex的状态存储是响应式的，从store实例中读取状态最简单的方法就是在`计算属性`中返回某个状态:

~~~ javascript
// 创建一个Counter组件
const Counter = {
    template: '<div>{{count}}</div>',
    computed: {
        count: function () {
            return store.state.count
        }
    }
}
~~~

每当`store.state.count`变化的时候，都会重新求取计算属性，并触发更新相关联的DOM。
然而，这种模式导致组件依赖全局状态单例。在模块化的构建系统中，在每个需要使用state的组件中需要频繁地导入，并且在测试组件是需要模拟状态。
为了便于组件使用state，Vuex通过`store`选项，提供了一种机制将状态从根组件“注入”到每一个组件中（需要调用`Vue.use(Vuex)`）：

~~~ javascript
import store from './store/index'

const app = new Vue ({
    el: '#app',
    store: store,
    components: {
        counter: Counter
    },
    template: '<div class="app"><counter></counter></div>'
})
~~~

> 注意：一般是在创建`Store`(export default new Vuex.Store)的js文件中引入(import)`Vuex`，并且使用`Vue.use(Vuex)`，然后在程序入口文件(main.js)中引入(import)`store`文件，并在应用根实例中利用store选项添加，如上面的代码。

通过在根实例中注册`store`选项，该store实例会注入到根组件下的所用子组件中，且子组件能通过`this.$store`访问到。那么`Counter`的实现可以修改为：

~~~ javascript
const Counter = {
    template: '<div>{{count}}</div>',
    computed: {
        count: function () {
            return this.$store.state.count
        }
    }
}
~~~

#### [mapState辅助函数][1]
[1]:https://juejin.im/post/5ae433ab518825671a6388d5

当一个组件需要获取多个状态的时候，`将这些状态都声明为计算属性会有些重复和冗余`。为了解决这个问题，我们可以使用`mapState`辅助函数帮助我们生成计算属性：

~~~ javascript
import {mapState} from 'vuex'
export default {
    // ...
    computed: mapState ({
        // 箭头函数可是代码更加简练
        count: state => state.count,
        // 传字符串参数等同于 `state => state.count`
        countAlias: 'count',

        //  为了能够使用`this`获取局部状态，必须使用常规函数
        countPlusLocalState (state) {
            return state.count + this.localCount
        }
    })
}
~~~

当映射的计算属性的名称与state的子节点名称相同时，我们也可以给`mapState`传一个字符串数组：

~~~ javascript
computed: mapState ([
    // 映射this.count 为 store.state.count(或者$store.state.count)
    'count'
])
~~~

或者映射方式也可以写成对象形式的：

~~~ javascript
computed: mapState ({
    count: 'count'
})
~~~

> 注意：上面代码与本小节第一段代码mapState辅助函数写法类似，不同的只是此处映射的`状态名称(state属性名称)`与`computed属性名称`相同。

#### 对象展示运算符

`mapState`函数返回值是一个对象。那么我们如何`将它与局部计算属性混合使用`呢？通常，我们需要使用一个工具函数将多个对象合并为一个，以使得我们可以将最终对象传给`computed`属性。但是自从有了``对象展开运算符``（现在处于ECMSCript提案stage-4阶段），那么代码可以大大简化：

~~~ javascript
computed：{
    localComputed: function () {
        // ...
    }
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState ({
        // ...
        count: 'count'
    })
}
~~~

> 注意：虽然将所有的状态放到Vuex会使状态变化更显式和易调用，但是会是代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是将其作为组件的局部状态使用。

### Getter

> 这个获取器和一些后端开发中模型层ORM中的获取器其实是差不多的功能。
>
> 比如说后端返回给我们的是一个int类型的时间戳，我们想把这个时间戳转换成正常人类可读的文本型时间表现形式（比如说2017年3月11日 12:43:31），那么我们就得在所有获取该状态的代码中增加一个转换函数。
>
> 可是现在有了状态获取器之后，我们可以统一将这个时间戳转字符串的函数写在获取器里面，要调用的时候就直接调用获取器就好了。
>
> 还有一些其他场景也可以使用获取器，比如说像错误码这种东西一般都是一个数字码对应一个文字形式的错误原因，我们也可以通过获取器来实现通过错误码拉取错误原因的功能。
>
> 使用获取器的方法则是直接调用Vuex实例的getter下的各种函数即可。

有时候我么需要从store中派生出一些状态，例如对列表进行过滤并计数：

~~~ javascript
computed: {
    doneTodosCount: function () {
        return this.$store.state.todos.filfter(todo => todo.done).length
    }
}
~~~

Vuex中的getters类似于vue组件中的computed选项的作用，抽离DOM中的数据计算。
Vuex允许我们在store中定义“getter”（可以认为是store的计算属性）。就像计算属性一样，getter的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
Getter接收state作为其第一个参数：

~~~ javascript
const store = new Vuex.Store ({
    state: {
        todos: [
            {id: 1, text: '...', done: true},
            {id: 2, text: '...', done: false}
        ]
    },
    // vuex中的getters类似于vue组件中的computed选项的作用，抽离DOM中的数据计算，
    getters: {
        doneTodos: state => {
            return state.todos.filter(todo => todo.done)
        },
        // 寻常写法
        doneTodosCount: function (state) {
            return state.todos.filter(todo => todo.done).length
        }
    }
})
~~~

#### 通过属性访问

Getter会暴露为`store.getters`对象，你可以以属性的形式访问这些值：

~~~ javascript
store.getters.doneTodos // -> [{id: 1, test: '...', done: true}]
~~~

Getter也可以接收其他getter作为第二各参数：

~~~ javascript
getters: {
    // ...
    doneTodoCount: (state, getters) => {
        return getters.doneTodos.length
    }
}
~~~

~~~ javascript
store.getters.doneTodoCount // -> 1
~~~

我们便很容易地在任何组件中使用它：

~~~ javascript
computed: {
    doneTodoCount: function () {
        return this.$store.getters.doneTodoCount
    }
}
~~~

> 注意：getter在通过属性访问时是作为Vue的响应系统的一部分`缓存`其中的。

#### 通过方法访问

你也可以通过让getter返回一个函数，来实现给getter传参。在你对store里的数组进行查询时非常有用。

~~~ javascript
getters: {
    // ...
    getTodoById: (state) => (id) => {
        return state.todos.find(todo => todo.id === id)
    }
}
~~~

~~~ javascript
store.getters.getTodoById(2) // -> {id: 2, text: '...', done: false}
~~~

> 注意：getter在通过方法访问时，每次都会进行调用，而不会缓存结果。

#### mapGetters辅助函数

`mapGetters`辅助函数仅仅是将store中的getter映射到局部计算属性上：

~~~ javascript
import {mapGetters} from 'vuex'
export default {
    // ...
    computed: {
        // 使用对象展开运算符将getter混入computed对象中
        ...mapGetters ([
            'doneTodoCount',
            'anotherGetter',
            // ...
        ])
    }
}
~~~

如果你想将一个getter属性另取一个名称，可使用对象像是重写上面代码：

~~~ javascript
mapGetters ({
    // 把`this.doneCount`映射为`this.$store.getters.doneTodosCount`
    doneCount: 'doneTodosCount'
})
~~~

### Mutation

> 这个Mutations其实国内目前也没有比较好的翻译，通常我们都是直接称Mutations。
>
> 我们前面只讲了可以通过调用Vuex的实例的state属性或者getter获取器来读取状态。但是没讲到如何修改状态。
>
> 官方文档中已经讲了需要先在Vuex实例的Mutations下编写对应的修改函数来修改状态。并且要修改的时候，要通过Vuex实例的commit方法来提交修改。也就是说任何对state状态的修改操作都必须写在Mutations中，并且还得用Vuex实例的commit来提交修改操作，并且由于Mutations函数可以传入参数，所以commit同理也可以传入参数。
>
> 这个时候可能有一些同学就会提问了，前面既然讲到了读取可以直接访问Vuex实例的state属性，为什么修改却不能直接去操作Vuex实例的state呢？官方文档和其他高手的回答是：
>
>> 再次强调，我们通过提交 mutation 的方式，而非直接改变 store.state.count，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。
>
> 相当于我们通过一个Mutations函数可以显式的在代码中告诉开发者，我们这个SPA中究竟会对状态进行哪些操作，操作方式是什么。并且在后期我们使用一些辅助开发工具，可以保存状态的快照，就像git或者svn一样可以回滚状态。如果你还是有点不明白，总之你就按照官方文档说的去做吧，等开发一段时间之后会慢慢明白作者的良苦用心的。
>
> 还有一个问题就是为什么状态修改的提交必须通过Vuex实例的commit方法提交呢？为什么不能直接调用Mutations函数呢？除了上面官方文档中提到的原因，网上还有高手解释了：因为Vue.js的状态修改其实是在内部有一个修改队列，通过commit的方式提交修改可以保证状态的修改是有序的。

更改Vuex的store的状态的唯一办法是`提交`(commit)`mutation`。Vuex中的mutation非常类似于事件：每个mutation都有一个字符串的`事件类型(type)`和一个`回调函数(handle)`。这个回调函数就是我们实际进行状态更改的地方，并且它会接收state作为第一个参数：

~~~ javascript
const store = new Vuex.Store ({
    state: {
        count: 1
    },
    mutations: {
        increment: function (state) {
            // 变更状态
            state.count++
        }
    }
})
~~~

你不能直接调用一个`mutation handler`。这个选项更像是事件注册：当触发一个类型为`increment`的mutation时，调用此函数。要唤醒一个mutation handler，你需要以相应的type调用`store.commit`方法：

~~~ javascript
store.commit('increment')
~~~

#### 提交载荷(Payload)

你可以向`store.commit`传入额外的参数，即mutation的`载荷(payload)`：

~~~ javascript
// ...
mutations: {
    increment: function (state, n) {
        state.count += n
    }
}
~~~

~~~ javascript
store.commit('increment', 10)
~~~

在大多数情况下，载荷应该是一个对象，这样可以包含多个字段并且记录的mutation会更易读：

~~~ javascript
// ...
mutations: {
    increment: function (state, payload) {
        state.count += payload.amount
    }
}
~~~

~~~ javascript
store.commit('increment', {
    amount: 10
})
~~~

#### 对象风格的提交方式

提交mutation的另一种方式是直接使用包含`type`属性的对象：

~~~ javascript
store.commit({
    type: 'increment',
    amount: 10
})
~~~

当使用对象风格的提交方式，整个对象都会被作为载荷传给mutation函数，因此handler保持不变：

~~~ javascript
mutations: {
    increment: function (state, payload) {
        state.count += payload.cmount
    }
}
~~~

#### Mutation需遵守Vue的响应规则

既然Vuex的store中的状态时响应式的，那么当我们变更状态时，监视状态的Vue组件也会自动更新。这也意味着Vuex中的mutation也需要与使用Vue一样遵守一些注意事项：

- 最好提前在你的store中初始化好所有所需属性
- 当需要在对象上添加新属性时，你应该
  - 使用`Vue.set(obj, 'newProp', 123)`，或者
  - 以新对象替换劳对象。例如，利用stage-3的对象展开运算符，我们可以这样写：

 ~~~ javascript
 state.obj = {...state.obj, newProp: 123}
 ~~~

#### 使用常量替代Mutation事件类型

使用常量替代mutation事件类型在各种Flux实现中是常见的模式。这样可以使`linter`之类的工具发挥作用，同时把这些常量放在单独的文件可以让你的代码合作者对整个APP包含的mutation一目了然：

~~~ javascript
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
~~~

~~~ javascript
// store.js
import Vuex from 'vuex'
import {SOME_MUTATION} from './mutation-types'

const store = new Vuex.Store({
    state: {...},
    mutations: {
        // 我们可以使用ES2015风格的计算属性命名功能来使用一个常量作为函数名
        [SOME_MUTATION] (state) {
            // mutate state
        }
    }
})
~~~

用不用常量取决于你——在需要多人协作的大型项目中，帮助很大。

#### Mutation必须是同步函数

mutation必须是`同步函数`。为什么？

~~~ javascript
mutations: {
    someMutation: function (state) {
        api.callAsynMethod(() => {
            state.count++
        })
    }
}
~~~

现在考虑，我们正在debug一个app并且观察devtool中的mutation日志。每一条mutation被记录，devtool都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中mutation中的异步函数中的回调导致无法完成这样的操作：因为当mutation触发的时候，回调函数还未被调用，devtool无法确定回调函数实际被调用的时机——`实质上任何在回调函数中进行的状态改变都是不可追踪的`。

#### 在组件中提交Mutation

你可以在组件中使用`this.$store.commit('xxx')`提交mutation，或者使用`mapMutations`辅助函数将组件中的methods映射为`store.commit`调用（需要在根节点注入`store`）。

~~~ javascript
import {mapMutations} from 'vuex'
export default {
    // ...
    methods: {
        ...mapMutations([
            'increment', // 将'this.increment'映射为`this.$store.commit('increment')`

            // `mapMutations` 也支持载荷：
            'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
        ]),
        ...mapMutations({
            add: 'increment' // 将`this.add()`映射为`this.$store.commit('increment')`
        })
    }
}
~~~

#### 下一步：Action

`在mutation中混合异步调用会导致你的程序很难调试`。例如，当你调用了两个包含异步回调的mutation来改变状态，你无法确定什么时候回调以及先回调哪个？进而需要区分mutation和action这两个概念。现在，我们已经知道了，在Vuex中，`mutation都是同步事务`：

~~~ javascript
store.commit('increment')
// 任何由“increment”导致的状态变更都应该在此刻完成。
~~~

为了处理异步操作，我们需要学习`Action`。

### Action

> 前面提到了Mutations中可以对状态进行操作，但是忘记告诉各位同学，Mutations中对状态的操作只能是同步操作，不能是异步操作。
>
> 如果这个时候我们有一个对状态的修改操作是异步的怎么办呢？
>
> 首先看看什么是异步操作？比如说ajax就可以选择是否发起异步请求，发起异步请求之后，我们就需要在回调函数里面进行请求结果的处理。
>
> 其实异步的状态修改本质上还是通过几个同步操作组合的，所以我们还是得先声明好mutation同步操作方法，然后在actions中进行异步操作。
>
> 前面讲到了mutation是用commit来提交操作的，那么actions是怎么提交的呢？官方文档中说了actions是使用Vuex实例的dispatch方法来提交（其实说分发会更加准确）的。

Action类似于mutation，不同之处在于：

- Action提交的是mutation，而不是直接变更状态
- Action可以包含任意异步操作

让我们来注册一个简单的action：

~~~ javascript
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment: function (state) {
            state.count++
        }
    },
    actions: {
        increment: function (context) {
            context.commit('increment')
        }
    }
})
~~~

Action函数接收一个与store实例具有相同方法和属性的context对象，因此你可以调用`context.commit`提交一个mutation，或者通过`context.state`和`context.getters`来获取state和getters。后面我们介绍到`Mudules`时，你就知道`context`对象为什么不是`store`实例本身了。

实践中，我们会经常用到ES2015的**解构函数**来简化代码（特别是我们需要调用commit很多次的时候）：

~~~ javascript
actions: {
    increment ({commit}) {
        commit('increment')
    }
}
~~~

#### 分发Action

Action通过`store,dispatch`方法触发：

~~~ javascript
store.dispatch('increment')
~~~

似乎直接分发mutation会更方便，但是，由于mutation必须要同步执行，而Action就不受约束，所以我们可以在action内部执行异步操作：

~~~ javascript
actions: {
    incrementAsync: function (context) {
        setTimeout(() => {
            context.commit('increment')
        }, 1000)
    }
}
~~~

或者，

~~~ javascript
actions: {
    incrementAsync ({commit}) {
        setTimeout(()=> {
            commint('increment')
        }, 1000)
    }
}
~~~

Action同样支持载荷方式和对象方式进行分发mutation：

~~~ javascript
// 以载荷形式分发
store.dispatch('incrementAsync', {
    amount: 10
})

// 以对象像形式分发
store.dispatch({
    type: 'incrementAsync',
    amount: 10
})
~~~

来看一个更加实际的购物车例子，涉及调用`异步API和分发多重mutation`：

~~~ javascript
actions: {
    checkout ({commit, state}, products) {
        // 把当前购物车的商品备份起来
        const savedCartItems = [...state.cart.added]
        // 发出结账请求，然后乐观地清空购物车
        commit(types.CHECKOUT_REQUEST)
        // 购物API接收一个成功回调和一个失败回调
        shop.buyProducts(
            products,
            // 成功操作
            () => commit(types.CHECKOUT_SUCCESSS),
            // 失败操作
            () => commit(types.CHECKOUT_FAILURE, savedCartItems)
        )
    }
}
~~~

注意我们正在进行一系列的异步操作，并且通过提交mutation来记录action产生的副作用（即状态的变更）。

#### 在组件中分发Action

你在组件中使用`this.$store.dispatch('xxx')`分发action，或者使用`mapActions`辅助函数将组件的methods映射为`store.dispatch`调用（需要先在根节点注入`store`）。

~~~ javascript
import {mapActions} from 'vuex'

export default {
    // ...
    methods: {
        ...mapActions([
            'increment', // 将'this.increment()'映射为`this.$store.dispatch('increment')`
            // `mapActions`也支持载荷：
            'incrementBy' // 将`this.incrementBy()`映射为`this.$store.dispatch('incrementBy')`
        ])，
        ...mapActions({
            add: 'increment' // 将`this.add()`映射为`this.$store.dispatch('increment')`
        })
    }
}
~~~

#### 组合Action

Action通常是异步的，那么如何知道action什么时候结束呢？更重要的是，我们如何才能组合多个action，以及处理更加复杂的异步流程？

首先，你需要明白`store.dispatch`可以处理`被触发的action的处理函数返回的**Promise**`，并且`store.dispatch`仍旧返回Promise:

~~~ javascript
actions: {
    actionA ({commit}) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                commit('someMutation'),
                resolve(),
            }, 1000)
        })
    }
}
~~~

现在你可以：

~~~ javascript
store.dispatch('actionA').then(() => {
    // ...
})
~~~

在另外一个action中也可以：

~~~ javascript
actions: {
    // ...
    actionB ({dispatch, commit}) {
        return dispatch('actionA').then(() => {
            commit('someOtherMutation')
        })
    }
}
~~~

最后，如果我们利用`async/await`，我们可以如下组合action：

~~~ javascript
// 假设 getData() 和 getOtherData() 返回的是Promise
actions：{
    async actionA ({commit}) {
        commit('gotData', await getData())
    },
    async actionB ({dispatch, commit}) {
        await dispatch('actionA') // 等待actionA完成
        commit('gotOtherData', await getOtherData())
    }
}
~~~

> 注意：一个`store.dispatch`在不同模块中可以触发多个action函数。在这种情况下，只有当所有触发函数完成后，返回的`Promise`才会执行。

### Module

> 如果你的SPA项目非常的庞大，那么状态可能本身还需要进行分模块分类管理，这个时候就需要用到模块了。
>
> 前面提到过actions方法在声明的时候需要带上一个context参数，原因在这里可以得到解答：
>> 对于模块内部的 action，context.state 是局部状态，根节点的状态是 context.rootState

由于应用构建方式使用的是单一状态树，应用的所有状态都会集中到一个比较大的`store`对象中。当应用变得非常复杂时，store对象就可能变得相当臃肿。

为了解决以上问题，Vuex允许我们将store分割成`模块（module）`。每个模块拥有自己的state、mutation、action和getter，甚至是嵌套子模块——从上至下以同样的方式进行分割：

~~~ javascript
const moduleA = {
    state: {...},
    mutations: {...},
    actions: {...},
    getters: {...}
}

const moduleB = {
    state: {...},
    mutations: {...},
    actions: {...},
    getters: {...}
}

const store = new Vuex.Store({
    modules: {
        a: moduleA,
        b: moduleB
    }
})
store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
~~~

#### 模块的局部状态

#### 命名空间

#### 模块动态注册

#### 模块重用

## 项目结构

~~~ javascript
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
~~~

## 插件

## 严格模式

> 在严格模式下，`状态发生改变如果不是由mutation函数引起的，将会抛出错误`。这能保证所有的状态变更都能被调试工具跟踪到。
> 前面提到了state的修改需要通过提交Mutations或者分发Actions来实现，但是事实上我们可以直接修改state吗？当然可以，但是在开发阶段，为了尽可能防止开发者直接修改，就可以通过严格模式来检测这种错误的修改方式，并且抛出异常。
>
>> 注意：`不要在发布环境下启用严格模式`！严格模式会深度监测状态树来检测不合规的状态变更——`请确保在发布环境下关闭严格模式，以避免性能损失`。
>
> 因此不要在生产环境下开启严格模式导致性能损失。

## 表单处理

## 测试

## 热重载