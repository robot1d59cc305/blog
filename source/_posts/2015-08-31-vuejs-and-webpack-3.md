---
layout: post
title: Vue.js 和 Webpack（三）
date: 2015-08-31 15:43
tag: frontend
---

**转载前请务必先联系邮箱**

## Why Vue.js + Webpack

在以往的一些小型的前端项目中，我习惯把逻辑（`scripts`）、视图（`views`）和样式（`styles`）分开在独立的目录当中，保证三者不耦合在一起。但是随着项目越来越大，这样的结构会让开发越来越痛苦，比如要增加或修改某个 `view` 的时候，就要在 `scripts` 和 `sytles` 里找到对应这个 `view` 的逻辑和样式进行修改。

为了避免这样随着项目增大带来的难于维护，我开始尝试**前端组件化**，把 `views` 拆分成不同的组件（component），为单个组件编写对应的逻辑和样式：

```
app/components
├── Chat
│   ├── Chat.jade
│   ├── Chat.js
│   └── Chat.scss
└── Video
    ├── Video.jade
    ├── Video.js
    └── Video.scss
```

这样的开发模式，不仅提高代码的可维护性和可重用性，还有利于团队之间的协作，一个组件由一个人去维护，更好地实现**分治**。幸运的是，随着 React 越来越火，组件化的开发模式也就越来越被接受。

## Using Vue.js + Webpack

在 Vue 中，可以利用一个 `.vue` 文件实现组件化，而不需要对每个组件分别建立 style, scripts 和 view。这样做的好处是使组件能更加直观，而坏处是目前有些 editor 对 `.vue` 的语法支持还是不太好。我用 Atom 写 `.vue` 的时候，`<style>` 的那一块并不能自动补全。不过我个人不依赖 css 的补全，所以没有太大的影响。如果你比较依赖这个，建议你还是把这些代码分离出来。

一个简单的 Vue Component：

```html
<!-- components/sample.vue -->

<template lang="jade">
  .test
    h1 hello {{msg}}
</template>

<script>
module.exports = {
  el: '#app',
  data: {
    msg: 'world'
  }
}
</script>

<style lang="sass">
  .test{
    h1{
      text-align: center;
    }
  }
</style>
```

我们使用 Webpack 就可以自动将 `.vue` 文件编译成正常的 JavaScript 代码，只需要在 Webpack 中配置好 `vue-loader` 即可：

```javascript
// webpack.config.js

module.exports = {
  entry: './app.js',
  output: {
    path: './build',
    filename: 'app.js'
  },
  module: {
    loaders: [
      {
        test: /\.vue$/, loader: 'vue-loader'
      }
    ]
  }
}
```

这样，就可以正常地在文件中 `require()` 所有 `.vue` 文件：

```javascript
module.exports = {
    el: '#app',
    data: {/* ... */},
    components: {
      'sample': require('./components/sample.vue')
    }
  }
```

### css 分离

`vue-loader` 使用 `style-loader` 把 component 当中的样式编译成字符串后插入到 `<head>` 中去。但我们希望把 css 文件独立出去，那么可以使用上一篇文章提到的 `ExtractTextPlugin` 插件，配合 `vue-loader` 的 `withLoaders()` 方法实现生成独立样式文件：

```javascript
// webpack.config.js
var vue = require('vue-loader')
  , ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
  entry: './app.js',
  output: {
    path: './build',
    filename: 'app.js'
  },
  module: {
    loaders: [
      {
        test: /\.vue$/, loader: vue.withLoaders({
          sass: ExtractTextPlugin.extract("css!sass") // 编译 Sass
        })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin('app.css') // 输出到 output path 下的 app.css 文件
  ]
}
```

## 总结
Vue.js 和 Webpack 的结合使用方法写到这里就已经算是写完了，当然，还有很多其它的实践方法，都要靠读者自己去摸索了，毕竟不能顾及到世界上各种各样的业务需求。这个系列仅仅是想给没有使用过 Vue.js 或者 Webpack 的读者一个大概的认识。

最后趁这个机会感慨一下，前端开发是让人感到兴奋的，我以前也写很多有关前端的东西，但从来不愿意称自己为『前端开发者』，是由于自己对前端开发的各种浅见，认为前端开发低端、repetitive、不能成大事。但是经过更加深入的实践，才慢慢发现前端也是工程化的、有学问的、有活力的。我很高兴可以作为一名『前端开发者』，在这里感受日新月异的氛围的技术浪潮。

**系列完结**。

## 延伸阅读

- [前端工程——基础篇](https://github.com/fouber/blog/issues/10)