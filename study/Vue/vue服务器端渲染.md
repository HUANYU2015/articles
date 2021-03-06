# Vue.js 服务器端渲染指南

$介绍$

## 什么是服务器端渲染(SS)？

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出Vue组件，进行生成DOM和操作DOM的活动。然而，**也可以将同一个组件渲染为服务器端的HTML字符串，将它们直接发送到浏览器，最后将这些`静态标记`“激活”为客户端上完全可交互的应用程序**。

服务器端渲染的Vue.js应用程序也可以被认为是“同构”或“通用”，因为应用程序的大部分代码都是可以在浏览器和客户端上运行的。

## 为什么使用服务器端渲染(SSR)？

相比传统SPA(单页应用程序 Single-Page Application)，服务器端渲染(SSR)的优势主要在于：

- 更好的SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。</br>请注意，截至目前，Google和Bing可以很好地对同步javascript应用程序进行索引。在这里，同步是关键。如果你的应用程序初始展示loading菊花图，然后通过ajax获取内容，抓取工具并不会等待异步完成后再行抓取内容。所以，如果SEO对你的站点至关重要，而你的页面又是异步获取内容，则你可能需要使用服务器端渲染(SSR)解决此问题。
- 更快的内容到达时间(time-to-content)，特别是在网络缓慢或者设备运行缓慢的情况下。无需等待所有的JavaScript都完成下载并执行，才显示`服务器端的标记`，所以你的用户将会更快速地看到完整渲染的页面。通常可以产生更好的用户体验，并且对于那些 内容到达时间(time-to-content)与转化率直接相关的应用程序而言，服务器端渲染(SSR)至关重要。

使用SSR还需要做以下权衡：

- 开发条件限制。</br>浏览器特定的代码，只能在某些生命周期钩子函数中使用；一些外部扩展库可能需要特殊处理，才能在使用服务器端渲染的应用程序中运行。
- 构建设置和部署需要满足更多要求。</br>服务器渲染应用程序，需要处于Node.js server 运行环境，这与可以部署在任何静态文件服务器上的完全静态的单页应用程序(SPA)不同。
- 更多的服务器端负载。</br>在Node.js中渲染完整的应用程序，显然会比仅仅提供静态文件的server占用更多的CPU资源(CPU_intensive - CPU集合)，因此如果你的应用程序是在高流量环境下使用的，需要准备相应的服务器负载，以及采用缓存策略。

在对你的应用程序使用服务器端渲染 (SSR) 之前，你应该问的第一个问题是，是否真的需要它。这主要取决于内容到达时间 (time-to-content) 对应用程序的重要程度。例如，如果你正在构建一个内部仪表盘，初始加载时的额外几百毫秒并不重要，这种情况下去使用服务器端渲染 (SSR) 将是一个小题大作之举。然而，内容到达时间 (time-to-content) 要求是绝对关键的指标，在这种情况下，服务器端渲染 (SSR) 可以帮助你实现最佳的初始加载性能。

## 服务端渲染 vs 预渲染(SSR vs Prerendering)

如果你的服务器端渲染只是用来改善少数营销页面(例如 `/`， `/about`，`/contact` 等)的SEO，那么你可能无需使用web 服务器实时动态变异HTML，而是使用预渲染方式，在构建时(build time)简单地生成针对特定路由的静态HTML文件。设置预渲染的优点是，它可以做的更简单，并且可以将你的前端作为一个完全静态的站点。

如果你在使用 webpack，你可以使用 [`prerender-spa-plugin`][1] 轻松地添加预渲染。

## 关于本指南

本指南专注于，使用 Node.js server 的服务器端单页面应用程序渲染。**将 Vue 服务器端渲染 (SSR) 与其他后端设置进行混合使用，是后端自身集成 SSR 的话题**，我们会在 [专门章节][2] 中进行简要讨论。

本指南将会非常深入，并且假设你已经熟悉 Vue.js 本身，并且具有 Node.js 和 webpack 的相当不错的应用经验。如果你倾向于使用提供了平滑开箱即用体验的更高层次解决方案，你应该去尝试使用 Nuxt.js。它建立在同等的 Vue 技术栈之上，但抽象出很多模板，并提供了一些额外的功能，例如静态站点生成。但是，如果你需要更直接地控制应用程序的结构，[**Nuxt.js**][3] 并不适合这种使用场景。无论如何，阅读本指南将更有助于更好地了解一切如何运行。

当你阅读时，参考官方 [HackerNews Demo][4] 将会有所帮助，此示例使用了本指南涵盖的大部分技术。

$基本用法$

## 安装

## 渲染一个Vue实例

## 与服务器集成

## 使用一个页面模板

## 模板插值

$编写通用代码$

$源码结构$

$路由和代码分割$

$数据预取和状态$

$客户端激活(client-side-hydration)$

$Bundle Renderer指引$

$构建管理$

$CSS管理$

$Head管理$

$缓存$

$流式渲染(Streaming)$

$在非Node.js环境中使用$


[1]:https://github.com/chrisvfritz/prerender-spa-plugin
[2]:https://ssr.vuejs.org/zh/guide/non-node.html
[3]:https://nuxtjs.org/
[4]:https://github.com/vuejs/vue-hackernews-2.0/