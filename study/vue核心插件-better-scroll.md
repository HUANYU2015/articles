# [better-scroll](https://github.com/ustbhuangyi/better-scroll)

## 介绍

### better-scroll是什么

***
`better-scroll`是一款重点解决移动端（已支持PC）各种滚动场景需求的插件。它的核心是借鉴的`iscroll`的实现，它的API设计基本兼容`iscroll`，在`iscroll`的基础上又扩展了一些`feature`，并做了一些性能优化。

`better-scroll`是基于原生`JS`实现的，不依赖任何框架。它是一款非常轻量级的JS lib。

### 起步

目前最适合移动端开发的前端MVVM框架是`Vue`，并且`better-scroll`可以很好的和vue配合使用。

better-scroll最常见的应用场景是列表滚动，它的HTML结构如下所示：

~~~ html
<div class="wrapper">
  <ul class="content">
    <li>...</li>
    <li>...</li>
    <li>...</li>
    ...
  </ul>
  <!--这里可以放一些其他的DOM，并不会影响滚动-->
</div>
~~~

上面的代码中better-scroll是作用在外层wrapper容器上的，滚动的部分是content元素。这里需要注意的是，`better-scroll只处理容器（wrapper）的第一个子元素（content）的滚动，其他的元素都会被忽略。`

最简单的初始化代码如下：

~~~ javascript
import BScroll from 'better-scroll'
let wrapper = document.querySelector('.wrapper')
let scroll = new BScroll(wrapper, {})
~~~

better-scroll提供了一个`BScroll`的类，我们初始化只需要 new 一个类的实例即可实例化的第一个参数是一个原生的DOM对象，第二个是一些配置参数，具体参考[better-scroll的文档][1]。当然，如果传递的是一个字符串，better-scroll内部会尝试调用querySelector去获取这个DOM对象，所以初始化代码也可以这样：

~~~ javascript
import BScroll from 'better-scroll'
let scroll = new BScroll('.wrapper', {})
~~~

>better-scroll 的初始化时机很重要，因为它在初始化的时候，会计算父元素和子元素的高度和宽度，来决定是否可以纵向和横向滚动。因此，我们在初始化它的时候，必须确保父元素和子元素的内容已经正确渲染了。如果子元素或者父元素 DOM 结构发生改变的时候，必须重新调用 scroll.refresh() 方法重新计算来确保滚动效果的正常。所以同学们反馈的 better-scroll 不能滚动的原因多半是初始化 better-scroll 的时机不对，或者是当 DOM 结构发送变化的时候并没有重新计算 better-scroll。

#### 如何在vue中使用better-scroll

完整代码如下：

~~~ html
<template>
  <div class="wrapper" ref="wrapper">
    <ul class="content">
      <li>...</li>
      <li>...</li>
      ...
    </ul>
  </div>
</template>
~~~

~~~ javascript
<script>
  import BScroll from 'better-scroll'
  export default {
    mounted: function () {
      this.$nextTick(() => {
        this.scroll = new Bscroll(this.$refs.wrapper, {})
      })
    }
  }
</script>
~~~

>vue.js提供了我们一个获取DOM对象的接口-vm.$refs。在这里，我们可以通过`this.$refs.wrapper`访问到这个DOM元素，并且我们在`mounted`这个钩子函数里，使用`this.$nextTick`的回调函数完成对better-scroll的初始化。因为这个时候，wrapper的DOM已经渲染了，我们可以正确计算它以及它内层content的高度，以确保滚动正常。
>
>这里的`this.$nextTick`是一个**异步函数**，使用该函数可以确保DOM已经渲染，具体可见其[内部显示细节][2]，底层用到了`MutationObserver`或者`setTimeout(fn, 0)`。其实我们在这里也可以将`this.$nextTick`替换成`setTimeout(fn, 20)`也是可以的（20ms是一个经验值，每一个Tick约为17ms），这对用户体验而言都是无感的。

#### 异步数据的处理

在我们的实际工作中，列表的数据旺旺都是异步获取却的，因此我们初始化better-scroll的时机需要在数据获取之后进行，d艾玛如下：

~~~ javascript
<template>
  <div class="wrapper" ref="wrapper">
    <ul class="content">
      <li v-for="item in data">{{item}}</li>
    </ul>
  </div>
</template>
<script>
  import BScroll from 'better-scroll'
  export default {
    data() {
      return {
        data: []
      }
    },
    created() {
      requestData().then((res) => {
        this.data = res.data
        this.$nextTick(() => {
          this.scroll = new Bscroll(this.$refs.wrapper, {})
        })
      })
    }
  }
</script>
~~~

>这里的 requestData 是伪代码，作用就是发起一个 http 请求从服务端获取数据，并且这个函数返回的是一个 promise（实际项目中我们可能会用 `axios` 或者 `vue-resource`）。我们获取到数据的后，需要通过异步的方式再去初始化 better-scroll，因为 Vue 是数据驱动的， Vue 数据发生变化（`this.data = res.data`）到页面重新渲染是一个异步的过程，我们的初始化时机是要在 DOM 重新渲染后，所以这里用到了 this.$nextTick，当然替换成 `setTimeout(fn, 20)` 也是可以的。
>
>为什么这里在 created 这个钩子函数里请求数据而不是放到 `mounted` 的钩子函数里？因为 requestData 是发送一个网络请求，这是一个异步过程，当拿到响应数据的时候，Vue 的 DOM 早就已经渲染好了，但是数据改变 —> DOM 重新渲染仍然是一个异步过程，所以即使在我们拿到数据后，也要异步初始化 better-scroll。

#### 数据的动态更新

#### scroll组件的抽象和封装

### 滚动原理

很多人已经用过 better-scroll，我收到反馈最多的问题是：

>better-scroll 初始化了， 但是没法滚动。

不能滚动是现象，我们得搞清楚这其中的根本原因。在这之前，我们先来看一下浏览器的滚动原理： 浏览器的滚动条大家都会遇到，当页面内容的高度超过视口高度的时候，会出现纵向滚动条；当页面内容的宽度超过视口宽度的时候，会出现横向滚动条。也就是当我们的视口展示不下内容的时候，会通过滚动条的方式让用户滚动屏幕看到剩余的内容。

better-scroll 也是一样的原理，我们可以用一张图更直观的感受一下：
![image1](https://github.com/HUANYU2015/articles/blob/master/better-scroll.png)

绿色部分为 wrapper，也就是父容器，它会有固定的高度。黄色部分为 content，它是父容器的第一个子元素，它的高度会随着内容的大小而撑高。那么，当 content 的高度不超过父容器的高度，是不能滚动的，而它一旦超过了父容器的高度，我们就可以滚动内容区了，这就是 better-scroll 的滚动原理。

## 安装

### NPM

better-scroll托管在NPM上，执行如下命令安装：

~~~ xml
npm install better-scroll --save
~~~

接下来就是在带，啊中引入了，webpack等构件工具都支持从node_modules里一如代码：

~~~ javascript
import BScroll from 'better-scroll'
~~~

如果是ES5语法，引入方式见如下代码：

~~~ javascript
var BScroll = require ('better-scroll')
~~~

### script加载

better-scroll也支持直接用script加载的方式，加载后会在window上挂载一个BScroll的对象。

## 选项/基础

better-scroll 支持很多参数配置，可以在初始化的时候传入第二个参数，比如：

~~~ javascript
let scroll = new BScroll('.wrapper',{
    scrollY: true,
    click: true
})
~~~

这样就实现了一个纵向可点击的滚动效果。better-scroll 支持的参数非常多，可以修改它们去实现更多的 feature。通常你可以不改这些参数（列出不建议修改的参数），better-scroll 已经为你实现了最佳效果，接下来我们来列举 better-scroll 支持的参数。

### startX

- 类型：Number
- 默认值：0
- 作用：横轴方向初始化位置

### startY

- 类型：Number
- 默认值：0
- 作用：纵轴方向初始化位置

### scrollX

- 类型：Boolean
- 默认值：false
- 作用：当设置为true时，可以开启横向滚动。
- 备注：当设置`eventPassthrough`为`horizontal`的时候，该配置无效。

### scrollY

- 类型：Boolean
- 默认值：true
- 作用：当设置为true时，可以开启纵向滚动。
- 备注：当设置`eventPassthrough`为`vertical`的时候，该配置无效。

### freeScroll

### directionLockThreshold

### eventPassthrough

- 类型：String
- 默认值：‘’
- 可选值：‘vertical’、‘horizontal’
- 作用：有时候我们使用better-scroll在某个方向模拟滚动的时候，希望在另一个方向保留原生的滚动（比如轮播图，我们希望横向模拟横向滚动，而纵向的滚动还是保留原生滚动，那么我们将`eventPassthrough`设置为`vertical`；相应地，如果我们希望保留横向原生滚动，可以将`eventPassthrough`设置为`horizontal`）。
- 备注：`eventPassthrough`的设置会导致其他一些选项配置无效。


### click

- 类型：Boolean
- 默认值：false
- 作用：better-scroll默认会阻止浏览器的`原生click事件`。当设置为true时，better-scroll会派发一个`click事件`，我们给派发的event参数加一个私有属性`_constructed`，值为true。

### dbclick

- 类型：Boolean | Object
- 默认值：false
- 作用：派发双击点击事件。当配置成true时，默认两次点击的延迟时间为300ms，如果配置成对象，则可以修改delay（延时时长）。

~~~ javascript
bdclick: {
  delay: 300
}
~~~

## 选项/高级

better-scroll还支持一些高级配置，来实现一些特殊的feature。

### sccrollbar

- 类型：Boolean | Object
- 默认值：false
- 作用：默认为false，当设置为true或者一个如下所示的Object时，都可开启滚动条。

~~~ javascript
scrollbar: {
  fade: true,
  interactive: false // 1.8.0新增
}
~~~

fade为true时表示当滚动停止时，滚动条是否具有渐隐效果，interactive表示滚动条是否可以交互。

## 方法/通用

better-scroll 提供了很多灵活的 API，当我们基于 better-scroll 去实现一些 feature 的时候，会用到这些 API，了解他们会有助于开发更加复杂的需求。

### refresh()

- 参数：无
- 返回值：无
- 作用：重新计算 better-scroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常。

## 方法/定制

better-scroll还提供了一些定制的方法，专门用来实现某一个feature所用。

### goToPage(x, y, time, easing)

- 参数
  - {Number}x横轴的页数
  - {Number}y纵轴的页数
  - {Number}time动画执行的时间
  - {Object}easing缓动函数，一般不建议修改，如果你想修改，可参考源码中ease.js里的写法。
- 返回值：无
- 作用：当我们做slide组件的时候，slide通常会分成多个页面。调用此方法可以滚动到指定的页面。

## 事件

better-scroll 除了提供了丰富的 API 调用，还提供了一些事件，方便和外部做交互。你可以利用他们实现一些更高级的 feature。

### beforeScrollStart

- 参数：无
- 触发时机：滚动开始之前

### scrollStart

- 参数：无
- 触发时机：滚动开始时

### scroll

- 参数：{Object}{x, y}滚动的实时坐标
- 触发时机：滚动过程中，具体时机取决于选项中的`probeType`

### scrollCancal

- 参数：无
- 触发时机：滚动被取消时

### scrollEnd

- 参数：{Object} {x, y} 滚动结束的位置坐标
- 触发时机：滚动结束。

### touchEnd

- 参数：{Object} {x, y} 位置坐标
- 触发时机：鼠标/手指离开。

### flick

- 参数：无
- 触发时机：轻拂时。

### refresh

- 参数: 无
- 触发时机：refresh 方法调用完成后。

### destroy

- 参数：无
- 触发时机：destroy 方法调用完成后。

### pullingDown

- 参数：无
- 触发时机：在一次下拉刷新的动作后，这个时机一般用来去后端请求数据。

### pullingUp

- 参数：无
- 触发时机：在一次上拉加载的动作后，这个时机一般用来去后端请求数据。

### zoomStart

- 参数：无
- 触发时机：缩放开始时。

### zoomEnd

- 参数：无
- 触发时机：缩放结束后。

## 属性

有时候我们想基于 better-scroll 做一些扩展，需要对 better-scroll 的一些属性有所了解，下面介绍几个常用属性。

### x

- 类型：Number
- 作用：scroll横轴坐标

### y

- 类型：Number
- 作用：scroll纵轴坐标

### maxScrollX

- 类型：Number
- 作用：scroll 最大横向滚动位置。
- 备注：scroll 横向滚动的位置区间是 0 - maxScrollX，并且 maxScrollX 是`负值`。

### maxScrollY

- 类型：Number
- 作用：scroll 最大纵向滚动位置。
- 备注：scroll 纵向滚动的位置区间是 0 - maxScrollY，并且 maxScrollY 是`负值`。

### movingDirectionX

- 类型：Number
- 作用：判断 scroll 滑动过程中的方向（左右）。
- 备注：-1 表示从左向右滑，1 表示从右向左滑，0 表示没有滑动。

### movingDirectionY

- 类型：Number
- 作用：判断 scroll 滑动过程中的方向（上下）。
- 备注：-1 表示从上往下滑，1 表示从下往上滑，0 表示没有滑动。

### directionX

类型：Number
作用：判断 scroll 滑动结束后相对于开始滑动位置的方向（左右）。
备注：-1 表示从左向右滑，1 表示从右向左滑，0 表示没有滑动。

### directionY

- 类型：Number
- 作用：判断 scroll 滑动结束后相对于开始滑动位置的方向（上下）。
- 备注：-1 表示从上往下滑，1 表示从下往上滑，0 表示没有滑动。

### enabled

- 类型：Boolean,
- 作用：判断当前 scroll 是否处于启用状态。

### isInTransition

- 类型：Boolean,
- 作用：判断当前 scroll 是否处于滚动动画过程中。
- 备注：当开启 CSS3 Transition 动画时判断该值。

### isAnimating

- 类型：Boolean,
- 作用：判断当前 scroll 是否处于滚动动画过程中。
- 备注：当开启 JS Animation 动画时判断该值。

[1]:https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/options.html
[2]:https://github.com/vuejs/vue/blob/dev/src/core/util/env.js#L66-L122

[Gitbook](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/#better-scroll%20%E6%98%AF%E4%BB%80%E4%B9%88)
[知乎](https://zhuanlan.zhihu.com/p/27407024)
[github](https://ustbhuangyi.github.io/better-scroll/doc/zh-hans/options.html#click)