---
layout: post
title: Vue.js 和 Webpack（一）
date: 2015-08-29 17:58
tag: frontend
---

**转载前请务必先联系邮箱**

最近在把 SISE Game（我们学校的校内游戏直播网站） 从原本的 Ruby on Rails 彻底用 Node.js 重写，  经过一些考虑，决定用 Vue.js 和 Express.js 实现前后端分离的架构，在这几天的重写过程中，积累了对 Webpack 和 Vue.js 的一些新的看法，今天我想先说说 Vue.js。

## About Vue.js
Vue 是个很年轻的 MVVM Library，常常有很多人用 Angular 和 Vue 比较，因为两者都是 MVVM，但实际上，前者是 Framework，而后者是 Library。前者有很陡峭的学习曲线，后者可以很快地掌握运用到项目中去。

Vue 的官方是用 **a library for building modern web interfaces** 来描述自己的。Vue 适合和 React 对比，因为在使用 Vue 的 Components System 开发比较大型的 Single Page Application 的时候，我发现它和 React 有一些相似的地方。如果你赞同 React 的思想，但又不想写 JSX，那么，你就可能需要试试 Vue 了。

一个用 Vue 实现 Data binding 的 Demo：

```html
<!-- index.html -->
<div id="#app">
  <input v-model="msg" />
  <p>{{ msg }}</p>
</div>
<script>
  var app = new Vue({
    el: '#app',
    data: {
      msg: 'hello Vue.js'
    }
  })
</script>
```

## Why Vue.js

前端开发发展到现在，我们做过的很多努力，都是在尝试把开发者从繁琐的 DOM  操作和管理 DOM state 中解放出去，我们希望只需要通过描述数据和行为，DOM 自己就可以发生对应的变化，React 在 View 这一层实现了这一目标，而 MVVM 则是通过 ViewModel 的 Data Binding。React 宣称自己是 View，那么在我看来， Vue 则是 View + ViewModel，并且 Vue 更加 lightweight 和 flexible。

Vue 最让我喜欢的是它的 Components System，利用它可以构建组件化的中大型应用。React 当然也是组件化的，但是 Virtual DOM (JSX) 在一些场景让我很不满意。比如有一次，我用一个使用 React 的项目中，想要在一 `<video>` 里使用 `webkit-playsinline` 这个 attribute，但是 React 不支持，渲染的时候直接被 ignored，我必须手动地操作 DOM 给 `<video>` setAttribute。相反，Vue 的 Components System 当中，写的是真正的 DOM，不需要担心不支持不兼容的各种情况。

容易被用作对比的是 Angular。我第一次听说 Vue 的时候，也是把它当作一个 lightweight 的 Angular alternative. 但是当真正实践使用 Vue 的时候，才发现它和 Angular 有着很大的不同。Angular 是一个 Framework，一旦你使用它，就必须按照它的一套去组织你的项目。以前写 Angular 项目的过程和经历对我个人来说都不太愉快，我更加倾向于 Vue 这种更灵活的方案。

关于 Vue 和其它库和框架的对比，官方也有作者更详细的 [解答](http://vuejs.org/guide/faq.html)（[中文版本](http://cn.vuejs.org/guide/faq.html)）

##Using Vue.js
SISE Game 并不算一个大型的 Web APP，但也规范地使用组件化的开发，整个项目的结构大致如下：

```text
├── app
│   ├── app.js #entry
│   ├── app.vue
│   ├── config.js 
│   ├── filters # 自定义的一些 filters
│   ├── components #各种组件
│   ├── models
│   ├── utils
│   └── views #各种页面的 views
│       ├── home.vue
│       ├── room.vue
│       ├── signin.vue
│       ├── signup.vue
│       └── user.vue
├── bower.json
├── build
├── gulpfile.js
├── index.html
├── node_modules
├── package.json
├── static #静态文件
│   ├── images
│   ├── styles
│   └── swfs
└── webpack.config.js
```

### 组件化
Vue 通过自己的 `.vue` 文件来定义 components，`.vue` 文件里包含组件的模板、逻辑和样式，从而实现组件和组件之间的**分治**，非常易于维护。

```html
<!-- components/user.vue -->
<template>
  <p>Hello {{ name }}</p>
  <button v-on="click: alertName()">alert!</button>
</template>

<script>
  module.exports = {
    data: function(){
      return {
        name: 'Randy'
      }
    },
    methods: {
      alertName: function(){
        alert(this.name);
      }
    }
  }
</script>

<style>
  p{
    color: #69C;
  }
</style>
```

以上就是一个简单的 component 实现，借助 webpack，甚至可以直接在 component 里写 es6、scss 和 jade。

###路由
路由对于 Single Page Application 来说应该算是不可少的东西，Vue 作为一个 Library，它本身并不提供这些组件。目前官方的 vue-router 仍处于 technical preview 的状态，官方也建议可以使用 component 和 Director.js 实现路由，比如：

```html
<div id="app">
  <component is="{{ currentView }}"></component>
</div>
```

```javascript
Vue.component('home', { /* ... */ })
Vue.component('page1', { /* ... */ })
var app = new Vue({
  el: '#app',
  data: {
    currentView: 'home'
  }
})
// Switching pages in your route handler:
app.currentView = 'page1'
```

这样你只需要操作 `app.currentView` 的值就可以实现视图的切换，这一步通常会配合 Director.js 这类的 hash router.

###延伸阅读

- [Vue.js - Component System](http://vuejs.org/guide/components.html)
- [Vue.js - Building Larger Apps](http://vuejs.org/guide/application.html)
- [vue-router](https://github.com/vuejs/vue-router)
- [和 Vue.js 框架的作者聊聊前端框架开发背后的故事](http://teahour.fm/2015/08/16/vuejs-creator-evan-you.html)

##To be continued
