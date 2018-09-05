---
layout: post
title: Alan's Blog | 从零开始创建 React 应用
header:  "从零开始创建 React 应用"
date:   2018-09-03 23:21:38 +0800
categories: "技术探究"
---
React 应用的搭建有多种方式，官方文档说明了可以使用的现成工具，在最后还提到可以自己[从零搭建 React 应用](https://reactjs.org/docs/create-a-new-react-app.html#creating-a-toolchain-from-scratch)，本文就针对这一点进行详细的说明，来看一看 React（可以推广到一般的前端应用）是如何搭建起来的。

> 本文翻译自一篇英文博客 《Creating a React App... From Scratch》
> 
> 原文链接：<a href="https://blog.usejournal.com/creating-a-react-app-from-scratch-f3c693b84658" target="_blank">
https://blog.usejournal.com/creating-a-react-app-from-scratch-f3c693b84658
</a>



![react app](https://cdn-images-1.medium.com/max/1600/1*BbK-XuJKmIpB0T8MeTndZQ.png)

React 并非开箱即用（out of the box），它使用一些 node（本教程中讨论的是 v.9.3.0 版本）还不能识别的关键字和语法。解决这个问题还真是需要费一点力气去搭建应用，然而 Facebook 已经提供了一个使构建 React 应用简单的选择，就不需要再为这个问题而费心了不是吗？

问题是，[create-react-app](https://github.com/facebook/create-react-app)将 React 应用运行的许多东西抽取出来，你并不会接触到，至少不需要 eject 和自己手动调试配置项了。可是总会有许许多多的情况下，你想要自定义一些实现，或者至少你想要知道在底层究竟发生了什么。

正如我所说的，在开始一个 React 应用时会遇到两个困难。第一个是 node 并不能处理所有的语法规则（如 [import/export](https://medium.com/@JedaiSaboteur/import-export-babel-and-node-a2e332d15673) 和 jsx)；第二个是你或者需要构建（build）你的文件，或者需要在开发的时候模拟一个服务器工作环境，后者尤为重要。

幸运的是，我们可以使用 [Bable](https://babeljs.io/) 和 [Webpack](https://webpack.js.org/) 处理这些难题。

### 搭建（Setup)

首先，为你的 React 应用新建一个目录。然后使用 `npm init` 初始化项目并使用任意编辑器打开。同时这也是使用 `git init` 的好时机。在你的新项目文件夹中，新建以下的目录结构：

{% highlight plain %}
.
+-- public
+-- src
{% endhighlight %}

往后想一点，我们最终将希望构建我们的应用，并且很可能需要从提交（commit）中排除构建版本和 node 模块，所以我们还要添加 `.gitignore` 文件排除（至少）`node_modules` 和 `dist` 目录。

`public` 目录将处理所有的静态资源（assets），其中最重要的是存放 `index.html` 文件，React 将会使用该文件来渲染你的应用。下面的代码来自 React 文档，只做了非常小的改动。你可以将这些 HTML 代码复制到 `public` 目录里新建的 `index.html` 文件中。

{% highlight html %}
<!-- sourced from https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>React Starter</title>
  </head>
  <body>
    <div id="root"></div>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <script src="../dist/bundle.js"></script>
  </body>
</html>
{% endhighlight %}

最需要注意的就是第 10 行（`<div id="root"></div>`），这是 React 应用嵌入的根部，和第 14 行（`<script src="../dist/bundle.js"></script>`），这里引用了（将要被）构建好的 React 应用。你可以按自己喜欢随意为构建好的脚本起名，本教程将会使用 `bundle.js`。

现在我们建立好了 HTML 页面，可以开始严肃认真起来了。我们将需要建立更多的一些东西。首先，需要确保我们写的代码可以被编译，因此我们需要 Babel。

### Babel

下一步，执行 `npm install --save-dev babel-core@6.26.3 babel-cli@6.26.0 babel-preset-env@1.7.0 babel-preset-react@6.24.1`。

`babel-core` 是主要的 babel 包，我们需要它来对我们的代码进行所有的转换。`babel-cli` 允许你使用命令行来编译文件。`babel-preset-env` 和 `babel-preset-react` 都是转换特殊风格代码的预编译设置（presets）——`env` 可以将 ES6+ 转换为传统的 JavaScript 代码，而 `react` 功能是相同的，只不过使用了 JSX。

在项目的根目录下，新建文件 `.babelrc`。此处我们要告诉 babel 需要使用 `env` 和 `react` 预设。

{% highlight json %}
{
  "presets": ["env", "react"]
}
{% endhighlight %}

如果你仅需要转换特定的特性或 `env` 中没有的特性，Babel 有许许多多可用的插件可供使用。我们暂时不用担心这些，不过你可以在[这里](https://babeljs.io/docs/plugins/)查看他们。

### Webpack

现在我们需要获得并配置 Webpack。我们还需要几个包，你需要他们保存为开发依赖：`npm install --save-dev webpack@4.12.0 webpack-cli@3.0.8 webpack-dev-server@3.1.4 style-loader@0.21.0 css-loader@0.28.11 babel-loader@7.1.4`。

Webpack 使用 [加载器（loaders）](https://webpack.js.org/loaders/)来打包处理不同类型的文件。它也能方便地协同开发环境服务器协同工作，我们使用开发环境服务器来运行开发环境的 React 项目，并且在我们改动（并保存）React 组件后重新加载浏览器页面。但是为了使用这些功能，我们需要配置 Webpack，使用我们的加载器并准备开发环境服务器。

在项目根目录下新建一个文件，命名为 `webpack.config.js`，这个文件导出一个 Webpack 配置对象。

{% highlight javascript %}
const path = require("path");
const webpack = require("webpack");

module.exports = {
  entry: "./src/index.js",
  mode: "development",
  module: {
    rules: [
      {
      	test: /\.(js|jsx)$/,
        exclude: /(node_modules|bower_components)/,
      	loader: 'babel-loader',
      	options: { presets: ['env']}
      },
      {
      	test: /\.css$/,
      	use: ['style-loader', 'css-loader']
      }
    ]
  },
  resolve: { extensions: ['\*', '.js', '.jsx'] },
  output: {
    path: path.resolve(\_\_dirname, "dist/"),
    publicPath: "/dist/",
    filename: "bundle.js"
  },
  devServer: {
    contentBase: path.join(\_\_dirname, "public"),
    port: 3000,
    publicPath: "http://localhost:3000/dist/",
    hotOnly: true
  },
  plugins:[ new webpack.HotModuleReplacementPlugin()]
};
{% endhighlight %}

我们来快速过一遍：[`entry`](https://webpack.js.org/configuration/entry-context/#entry) 告诉 Webpack 应用从哪里开始运行，也是打包文件的起点。下面的一行告诉 Webpack 我们当前工作在开发模式下——当我们运行开发模式服务器时，就不需要添加模式标志（mode flag）了。

[`module`](https://webpack.js.org/configuration/module/) 对象定义了你导出的 JavaScript 模块如何转换，根据给出的 [`rules`](https://webpack.js.org/configuration/module/#module-rules) 数组确定转换那些类型的文件。

我们的第一条规则就是关于转换 ES6 和 JSX 语法的。[`test`](https://webpack.js.org/configuration/module/#rule-test) 和 [`exclude`](https://webpack.js.org/configuration/module/#rule-exclude) 属性是文件匹配条件。当前案例中，它将会匹配 `node_modules` 和 `bower_components` 以外的所有文件。由于我们将会转换 `.js` 和 `.jsx` 文件，需要指示 Webpack 使用 Babel。最后，我们在 [options](https://webpack.js.org/configuration/module/#rule-options-rule-query) 中指明想要使用 `env` 预设。

下一条规则是用于处理 CSS 的。由于没有使用预处理或后处理 CSS，我们只需要确保添加 `style-loader` 和 `css-loader` 到 `use` 属性。`css-loader` 需要 `style-loader` 才能工作。在仅仅使用一个加载器时，`loader` 是 `use` 属性的一个简写形式。

[`resolve`](https://webpack.js.org/configuration/resolve/) 属性让我们指定 Webpack 解析那些扩展名——这让我们能够不需要写扩展名便能导入需要的模块。

[`output`](https://webpack.js.org/configuration/output/) 属性告诉 Webpack 将打包好的代码放置到哪里。属性 [`publicPath`](https://webpack.js.org/configuration/output/#output-publicpath) 指定代码包存放的目录，同时也告诉 `webpack-dev-server` 从哪里获取文件。

属性 [`publicPath`](https://webpack.js.org/configuration/output/#output-publicpath) 是一个帮助我们使用开发环境服务器的特殊属性。它指定目录的公共 URL——至少是 `webpack-dev-server` 知道和关心的。如果设置错误，将会得到 404，因为服务器无法从正确的位置获取文件！

我们在 [`devServer`](https://webpack.js.org/configuration/dev-server/) 属性中配置 `webpack-dev-server`。对于我们的需求来说，不需要太多的配置——只需要静态文件（例如 `index.html`）的存放位置和服务端口就可以了。注意 `devServer` 也有一个 [`publicPath`](https://webpack.js.org/configuration/dev-server/#devserver-publicpath-) 属性，它告诉服务器打包后的文件究竟在哪个位置上。

最后的这一点可能有些困惑——要十分注意：[`output.publicPath`](https://webpack.js.org/configuration/output/#output-publicpath) 和 [`devServer.publicPath`](https://webpack.js.org/configuration/dev-server/#devserver-publicpath-) 是不相同的。请阅读相关词条，两次。

最后，由于我们想使用 [`热模块替换（Hot Module Replacement, HMR）`](https://webpack.js.org/guides/hot-module-replacement/)，以至于不需要反复刷新来看到我们的修改。就这个文件而言，我们要做的就是在 [`plugins`](https://webpack.js.org/configuration/plugins/) 属性中实例化一个新的插件（plugin）实例，并确保在 `devServer` 中设置 `hotOnly` 为 `true`。在 HMR 工作之前我们还需要在 React 中设置一样东西。

至此复杂的搭建工作就完成了，下面让我们让 React 工作起来。

### React

首先，我们还是需要获取两个新包：`react@16.4.1` 和 `react-dom@16.4.1`，安装并保存为一般依赖。

我们需要告诉 React 应用挂载到 DOM 的位置（`index.html` 文件中）。在 `src` 目录中新建一个名为 `index.js` 的文件，这是一个很小的文件，但是在 React 应用中却起到很大的作用。文件内容如下。

{% highlight javascript %}
import React from "react";
import ReactDOM from "react-dom";
import App from "./App.js";
ReactDOM.render(
  <App />,
  document.getElementById("root")
);
{% endhighlight %}

`ReactDOM.render` 函数告诉 React 需要渲染什么，渲染到哪里——在本例中，我们将渲染一个叫做 `App` 的组件（后面将会创建）到 ID 是 root 的 DOM 元素上。

现在，在 `src` 下面再新建一个名为 `App.js` 的文件。如果你使用过 `create-react-app` 创建 React 应用，这部分你肯定非常熟悉。该文件就是一个 React 组件。

{% highlight jsx %}
import React, { Component } from "react";
import "./App.css";

class App extends Component {
  render() {
    return (
      <div className="App">
      	<h1>Hello, World!</h1>
      </div>
    );
  }
}
{% endhighlight %}

我曾经提到过 Webpack 也处理 CSS（并且我们在组件中需要引入它）。让我们在 `src` 目录中添加一个非常简单的样式。

{% highlight css %}
.App {
  margin: 1rem;
  font-family: Arial, Helvetica, sans-serif;
}
{% endhighlight %}

你最终的项目结构应该看起来像下面这样，除非你改变了一些文件的名字：

{% highlight plain %}
.
+-- public
| +-- index.html
+-- src
| +-- App.css
| +-- App.js
| +-- index.js
+-- .babelrc
+--.gitignore
+-- package-lock.json
+-- package.json
+-- webpack.config.js
{% endhighlight %}

现在我们有了一个可以运行的 React 应用了！我们可以在终端执行 `webpack-dev-server --mode developemnt` 命令启动开发环境服务器。我建议将命令放到 `package.json` 中的 `start` 脚本处，可以让你少敲打九个键。

### 完成 HMR

如果你现在启动服务器，你会发现你的改动并没有在客户端产生审核影响，什么原因呢？

HMR 需要知道究竟要替换什么，而我们目前什么也没有给出。对于这一点，我们要使用 React 团队提供的一个包：`react-hot-loader`。

你可以将它作为一般依赖安装——根据文档所示。

*注意：你可以安全地将 react-hot-loader 安装为一般依赖而不是开发依赖，因为它会自动确保在生成环境中不会执行，并且它是最小封装的*

现在，在 App.js 中导入 react-hot-loader，并将导出的对象标记为热重载，修改代码如下。

{% highlight javascript %}
import React, { Component } from "react";
import { hot } from "react-hot-loader";
import "./App.css";

class App extends Component {
  render() {
    return (
      <div className="App">
      	<h1>Hello, World!</h1>
      </div>
    );
  }
}

export default hot(module)(App);
{% endhighlight %}

现在当你运行你的应用时，代码修改在保存后会立刻在客户端更新了。

### 最后的细节

你也许会注意到一些关于运行你的项目的有趣（甚至惊讶）的事情：构建后的文件从未在 `dist` 目录下面出现。**看吧，`webpack-dev-server` 实际上是从内存中提供打包好的文件的——一旦服务停止，他们就没有了。要想真正地构建你的文件，我们要适当地使用 `webpack`——在 `package.json` 中添加一个名为 `build` 的脚本，命令是：`webpack --mode developent`。**你可以把 `developent` 替换成 `production`，但是如果你完全省略 `--mode`，它将会使用前者并且给出一个警告。

以上内容差不多覆盖了渲染一个基本 React 应用需要的所有内容，不需要 `create-react-app` 的帮助。然而还有更多需要添加到实现中的东西，来使项目更加完整——例如图片没有设置给 Webpack 处理，[但是有一个加载器来完成这个任务](https://webpack.js.org/loaders/file-loader/)。这就交给你们自己去实现了。毕竟，如果你不需要或不想提供文件服务，那就只是个累赘了，对吧？

我希望这篇文章可以帮助你稍微了解运行 React 应用所需要的东西，以及底层的基本原理。关于 Babel 和 Webpack 我并没有涉及得太过深入，但是请访问任何出现在文章中的无数链接或者直接访问他们的文档。他们都是非常棒的工具，乍看上去
非常的吓人，但实际上并非如此。他们可以使你的应用更上一层楼。

[请随意查看我在 github 上的实现](https://github.com/paradoxinversion/react-starter)（它更加深入一些），或者[在 twitter 上给我留言](https://twitter.com/JedaiSaboteur)。