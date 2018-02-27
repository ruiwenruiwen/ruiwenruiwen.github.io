---
title: webpack 初探
date: 2017-07-28 21:07:35
tags: webpack
---

本篇是对于 wabpack 学习的笔记。由于 webpack 官方文档略微有点难读懂，所以这几天我一边看文档一边试着用 webpack 做了一个轮播组件。本篇博文主要是用来记录我在做这个组件的时候遇到的问题以及学到的内容。

<!--more-->

## webpack 结构

webpack 是一个模块打包机，它分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（scss，TypeScript等），将其打包为合适的格式以供浏览器使用。它还可以将按需加载的模块进行代码分隔，等到实际需要的时候再异步加载。

webpack 的配置主要是在 `webpack.config.js` 以及 `packge.json` 中。接下来就记录一下我在配置自己的 webpack 项目中学习到的东西。（所用的版本是 webpack 3.xxx ）

### 入口起点与出口

首先， webpack 需要一个入口文件，这是所有依赖关系的入口，webpack 从这个入口开始静态解析，分析模块之间的依赖关系。这个入口文件内可以用 AMD , require JS 等方式引入其他的文件。

webpack 还需要一个出口，表示文件打包后存放在哪里，以及打包后的文件名是什么。

具体配置如下：

```javascript
module.exports = {
    entry: __dirname +  "/src/core.js",        //唯一入口文件的写法
    output: {
        path: path.join(__dirname, '/build'),  //打包后文件存放的文件夹
        filename: 'demo.js'                     //打包后输出文件的文件名
    }
}
```

上述入口起点的写法是只有一个入口文件的写法。根据官方文档，webpack 还可以有多个入口文档，并且可以像上述出口设置一般，用对象的方法表示。

### loader

loader 是对应用程序中资源文件进行转换。通过不同的 loader ，webpack 可以对将各种资源文件转换成新的浏览器能够直接使用的文件。

要使用这些不同的 loader ，需要下载。比如：

+ `npm install css-loader style-loader --save-dev`：安装 babel-loader 进行转码
+ `npm install babel-preset-es2015 --save-dev`：设置转码规则，使不能直接运行的代码（如 es6 ）转成浏览器可以运行的代码
+ `npm install css-loader style-loader --save-dev`：css-loader会遍历css文件，找到所有的 url(...) 并且处理。style-loader 会把所有的样式插入到你页面的一个 style tag 中。

倘若是代码中使用了 `scss`，`less` 等 css 预处理器，或者是用了 `react`， `express` 等模块，也需要下载相应的 loader 。

loader 的配置包含以下几个方面：

+ `test`：一个匹配loaders所处理的文件的拓展名的正则表达式（必须）
+ `loader`：loader的名称（必须）
+ `include/exclude`：手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）
+ `query`：为 loaders 提供额外的设置选项（可选）

在这次的练习中，我的配置如下：

```javascript
module: {
    rules: [
        {
            test:/\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            query: {
			    presets: ['es2015']
			}
        },
        {
            test:/\.css$/,
            loader: 'style-loader!css-loader?modules'
        }
    ]
},
```

需要注意的是：如果将 css 的 loader 设置成 `loader: 'style-loader!css-loader?modules` 那么打包过后的 css 选择器将会被编码成与 css 文件中定义的不同。如果是 `loader: 'style-loader!css-loader` 则不会有这种情况。这是因为 modules 表示打开 css modules 功能，一般来说css的作用域都是全局的，我们常在母版页里面添加了多个样式文件，后面的样式文件会覆盖前面的样式文件，常常给我们的调试带来麻烦。而 css modules 通过 import 引入了本地作用域。这样能够避免命名空间冲突。因此会出现看似 css 选择器乱码的现象。

### plugins

插件也是 webpack 重要的一部分。他的目的在于解决 loader 无法实现的事情。

由于 plugin 可以携带参数/选项，在配置插件的时候，需要向 plugins 属性传入 new 实例。如下所示：

```javascript
plugins: [
    new HtmlWebpackPlugin({
        template: __dirname + "/src/index.html"
    }),
    new webpack.HotModuleReplacementPlugin()
],
```

在此介绍上述两个常用插件：

+ `HtmlWebpackPlugin`：这个插件用来简化创建服务于 webpack bundle 的 HTML 文件，尤其是对于在文件名中包含了 hash 值，而这个值在每次编译的时候都发生变化的情况。既可以让这个插件来帮助你自动生成 HTML 文件，也可以使用 lodash 模板加载生成的 bundles，或者自己加载这些 bundles 。
+ `HotModuleReplacementPlugin`：这是大名鼎鼎的热模块替换。在某些场景下，可以在不刷新页面的情况下让代码生效，并且可以即时反应到浏览器上。

```javascript
devServer: {
    contentBase: path.join(__dirname, "src"),
    historyApiFallback: true,
    inline: true，  //使用 inline 模式，将webpack-dev-sever的客户端入口添加到包(bundle)中
    port: 3000     //打开的端口  不写此行则打开的为默认的 8080 端口
}
```

webpack-dev-server 是 webpack 官方提供的一个小型 Express 服务器。使用它可以为webpack打包生成的资源文件提供web服务。

webpack-dev-server 主要提供两个功能：
+ 为静态文件提供服务
+ 自动刷新和热替换(HMR)

## 运行

经过以上的配置后，若要运行还需要在 `packet.json` 中经过一下配置：

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack -p",                         //生产环境使用
    "dev": "webpack-dev-server --progress"         //开发环境使用
}
```

`packet.json` 中的 `script` 对象可以用来写一些脚本命令。

现在，可以在命令行通过以下命令：
+ `npm run dev`：启动开发环境，并且支持热更新。
+ `npm run build`：打包生产环境的代码。-p 参数会开启生产环境模式。在这个模式下 webpack 会将代码做压缩等优化。

## 总结

webpack 是我目前学习的唯一的一个前端工程化的工具，也许这篇博文说的不大详尽，也不能解决我目前遇到的所有问题。不过我还是希望先做一个笔记，方便以后的学习。
