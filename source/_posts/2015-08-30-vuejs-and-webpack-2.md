---
layout: post
title: Vue.js 和 Webpack（二）
date: 2015-08-30 00:31
tag: frontend
---

**转载前请务必先联系邮箱**

## About Webpack

与其费周章说明 Webpack 是什么东西，倒不如先说说不用 Webpack 以前的一些现实。

我们在写前端 JavaScript 的时候，通常是写在多个 `.js` 文件里，通过闭包避免全局变量污染，然后一股脑地用 `<script>` 引入。

```html
<body>
  ...
  <script src="a.js"></script>
  <script src="b.js"></script>
  <script src="c.js"></script>
</body>
```

出于性能上的追求，我们会应该把 `a.js` `b.js` `c.js` 合并为同一个文件 `bundle.js` 来减少请求数量，变成：

```html
<body>
  ...
  <script src="bundle.js"></script>
</body>
```

使用 Gulp/Grunt 等自动化构建工具很容易可以实现这样的 concat，但是很快我们就会发现，单纯的 concat 并不是一个好的方案，因为代码文件之间的依赖关系不明确，这样一来，有时不得不花一些时间去组织 concat 的顺序。我们很希望像写 Node 一样模块化地去写前端 JavaScript。

又有些时候，在两个不同的页面当中我们常常会共用一些代码，单纯的 concat 会增加很多不必要的体积。

所以  ，我们理想的情况是，可以在前端优雅地写符合模块规范（AMD, UMD, CommonJS）的代码并且自动打包，最好还能自动把重用的文件分离出来。

嘿，Webpack 就很擅长做这种事。

## Using Webpack

Webpack 兼容所有模块规范（如果你不知道到底用哪一种，就用 CommonJS）。

配置 webpack 比较简单，你需要定义入口文件和 bundle 输出的目录：

```javascript
// webpack.config.js
module.exports = {
  entry: './app.js',
  output: {
    path: './build',
    filename: 'bundle.js'
  }
}
```

这样，你就能在前端这样去写 JavaScript：

```javascript
// /app.js
var Vue = require('vue');
var app = new Vue({/*...*/})
```

这是 CommonJS 的写法，如果你写过 Node.js，应该对这种写法相当熟悉。这时运行 `$ webpack` ，webpack 会自动根据入口文件 `app.js` 中的依赖关系来打包成单个 js 文件，输出到配置文件中指定的 output path 中。

webpack 也可以通过 plugin 自动分析重用的模块并且打包成单独的文件：

```javascript
// webpack.config.js
var webpack = require('webpack'),
  CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin

module.exports = {
  entry: './app.js',
  output: {
    path: './build',
    filename: 'bundle.js'
  },
  plugins: [
    new CommonsChunkPlugin('vendor.js')
  ]
}
```
### 多入口文件
webpack 的一个特色是可以指定多个入口文件，最后打包成多个 bundle。比如说 Timeline page 和 Profile page 是不同的页面，我们不希望两个页面的 js 被打包在一起，这时我们就可以为 timeline 和 profile 两个页面定义不同的入口：

```javascript
// webpack.config.js
module.exports = {
  entry: {
    timeline: './timeline.js',
    profile: './profile.js'
  },
  output: {
    path: './build',
    filename: '[name].bundle.js'
  },
  plugins: [
    new CommonsChunkPlugin('vendor.js')
  ]
}
```

最后会被分别打包成 `timeline.bundle.js` 和 `profile.bundle.js`。

###loader

webpack 神奇的地方在于，任何的文件都能被 `require()`。依靠各种 loader，使你可以直接 `require()` 样式、图片等静态文件。这些静态文件最后都会被处理（比如 scss pre-process 和图片的压缩）和打包在配置好的 output path 中。

```scss
#container{
  background-image: url(require('./images/background.png'));
  p{
    color: #69C;
  }
}
```

```javascript
// app.js
require('./styles/app.scss')
// blablabla....
```

你可以像上面这样在 JavaScript 中引入 scss （和在样式中引入图片），只要你配置好处理 scss 的 loader：

```javascript
module.exports = {
  entry: './app.js',
  output: {
    path: './build',
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.(css|scss)$/,
        loader: ExtractTextPlugin.extract('style','css!sass')
      },
      {
        test: /\.(png|jpg)$/,
        loader: 'url?limit=8192' // 图片低于 8MB 时转换成 inline base64，非常神奇！
      }
    ]
  }
}
```

css 默认被编译到 JavaScript 中成为字符串后再被插入到 `<style>` 中，我个人建议使用 `ExtractTextPlugin` 这个插件把 css 分离出去。

## 总结
webpack 是一个十分好用的模块打包工具，使用它更加利于实现前端开发工程化。

很多人认为 webpack 可以取代 Gulp/Grunt 等构建工具，其实不然。Webpack 仅仅是**顺便**替构建工具分担了一些预编译预处理的工作，而构建工作不仅仅只有预编译啊。

### 延伸阅读

- [Webpack Getting Started](http://webpack.github.io/docs/tutorials/getting-started/)
- [Webpack 入门指迷](http://segmentfault.com/a/1190000002551952)
- [How Instagram.com Works](https://www.youtube.com/watch?v=VkTCL6Nqm6Y)

##To be continued