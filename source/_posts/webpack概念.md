---
title: webpack 概念
date: 2018-07-28 11:26:31
tags:
---

# webpack

## 概念

### 入口（entry）

可以配置`entry` 来指定一个入口起点(或多个入口起点)。默认值为`./src`

```javascript
module.exports={
    entry: './path/to/my/entry/file.js'
}
```

多入口

```javascript
// 应用场景  分离应用程序和第三方库入口
const config = {
 	entry: {
        app: './src/app.js',
        vendors: './src/vendors'
 	}
};

// 多页面应用程序
const config = {
    entry: {
        pageOne: './src/pageOne/index.js',
        pageTwo: './src/pageTwo/index.js',
        pageThree: './src/pageThree/index.js'
    }
}
```



### 出口（output）

`output`属性告诉`webpack` 在哪里输出它所创建的bundle，以及如何命名这些文件。默认为`./dist`

**注意** 即使可以存在多个入口，但只指定一个输出配置

配置output属性的最低要求是

1. filename     用于输出文件的文件名
2. 目标输出目录的path的绝对路径

```javascript
const path = require('path');
module.exports={
    entry: './path/to/my/entry/file.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'my-first-webpack.bundle.js'
    }
};
```

多个入口起点

```javascript
const config = {
    entry: {
        app: './src/app.js',
   		search: './src/search.js'
    },
    output: {
        filename: '[name].js', // 根据不同的名称区分
        path: __dirname + '/dist'
    }
}
// 写入到硬盘：./dist/app.js, ./dist/search.js
```

以下是使用 CDN 和资源 hash 的复杂示例：

```javascript
output: {
  path: "/home/proj/cdn/assets/[hash]",
  publicPath: "http://cdn.example.com/assets/[hash]/"
}
在编译时不知道最终输出文件的 publicPath 的情况下，publicPath 可以留空，并且在入口起点文件运行时动态设置。如果你在编译时不知道 publicPath，你可以先忽略它，并且在入口起点设置 __webpack_public_path__


__webpack_public_path__ = myRuntimePublicPath

// 剩余的应用程序入口
```



### loader

`loader`让`webpack`能够去处理那些非`javascript`文件，loader可以将所有类型文件转换为webpack能够处理的有效模块，然后利用webpack打包能力，对它们进行处理，

注意，loader 能够 `import` 导入任何类型的模块（例如 `.css` 文件），这是 webpack 特有的功能，其他打包程序或任务执行器的可能并不支持。我们认为这种语言扩展是有很必要的，因为这可以使开发人员创建出更准确的依赖关系图。

在更高层面，在 webpack 的配置中 **loader** 有两个目标：

1. `test`属性，用于标识出应该被对应的loader进行转换的某个或某些文件
2. `use`属性，表示进行转换时，应该使用哪个loader

```javascript
const path = require('path');

const config = {
    entry: './bundle.js',
    module: {
        rules: [
            {test: /\.txt$/, use: 'raw-loader'}
        ]
    }
};

module.exports = config;
```

以上配置中，对一个单独的module对象定义了`rules`属性，里面包含必须属性： `test`和`use`

在`require()` / `import`语句中被解析为`.txt`的路径时，在你对它打包之前，先使用`rew-loader`转换一下

重要的是要记得，**在 webpack 配置中定义 loader 时，要定义在 module.rules 中，而不是 rules**。然而，在定义错误时 webpack 会给出严重的警告。为了使你受益于此，如果没有按照正确方式去做，webpack 会“给出严重的警告”

在你的应用中，有三种使用loader的方式：

1. 配置 （推荐）： 在webpack.config.js文件中指定loader
2. 内联：在每个import语句中显式指定loader
3. CLI：在shell命令中指定它们

**配置**

`module.rules`允许你在webpack配置中指定对各loader。这是展示loader的一种简明方式，并且有助于使代码变得简洁

```javascript
module: {
    rules: [
        {
            test: '/\.css$/',
            use: [
                {loader: 'style-loader'},
                {
                    loader: 'css-loader',
                    option: {
                        modules: true
                    }
                }
            ]
        }
    ]
}
```

**内联**

可以在 `import` 语句或任何等效于 "import" 的方式中指定 loader。使用 `!` 将资源中的 loader 分开。分开的每个部分都相对于当前目录解析

```javascript
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

通过前置所有规则及使用 `!`，可以对应覆盖到配置中的任意 loader。

选项可以传递查询参数，例如 `?key=value&foo=bar`，或者一个 JSON 对象，例如 `?{"key":"value","foo":"bar"}`。

**CLI**

```javascript
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

**loader特性**

1. lodaer支持链式传递，能够对资源使用流水线，一组链式的loader将按照`相反的顺序`执行，loader链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript
2. loader 可以是同步的，也可以是异步的。
3. loader 运行在 Node.js 中，并且能够执行任何可能的操作
4. loader 接收查询参数。用于对 loader 传递配置
5. loader 也能够使用 `options` 对象进行配置。
6. 除了使用 `package.json` 常见的 `main` 属性，还可以将普通的 npm 模块导出为 loader，做法是在 `package.json` 里定义一个 `loader` 字段。
7. 插件(plugin)可以为 loader 带来更多特性。
8. loader 能够产生额外的任意文件

loader 遵循标准的[模块解析](https://www.webpackjs.com/concepts/module-resolution/)。多数情况下，loader 将从[模块路径](https://www.webpackjs.com/concepts/module-resolution/#module-paths)（通常将模块路径认为是 `npm install`, `node_modules`）解析。

loader 模块需要导出为一个函数，并且使用 Node.js 兼容的 JavaScript 编写。通常使用 npm 进行管理，但是也可以将自定义 loader 作为应用程序中的文件。按照约定，loader 通常被命名为 `xxx-loader`（例如 `json-loader`）

### 插件(plugins)

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。[插件接口](https://www.webpackjs.com/api/plugins)功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要`require()`它，然后把它添加到`plugins`数组中。多数插件可以通过选项(option)自定义。你可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用`new`操作符来创建它的一个实例

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');

const config = {
    module: {
        rules: [
            {test: /.txt$/,use: 'raw-loader'}
        ]
    },
    plugins: [
        new webpack.optimize.UglifyJsPlugin(),
        new HtmlWebpackPlugin({template: './src/inde.html'})
    ]
};

module.exports = config;
```

webpack插件是一个具有`apply`属性的javascript对象

**用法**

由于插件可以携带参数/选项，你必须在webpack配置中，向`plugins`属性传入`new`实例。

webpack.config.js

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过npm安装
const webpack = require('webpack'); // 访问内置的插件
const path = require('path');

const config = {
    entry: './path/to/my/entry/file.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, '/dist')
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                use: 'babel-loader'
            },
		]
    },
    plugins: [  // 向plugins传入new实例
        new webpack.optimize.UglifyJsPlugin(),
        new HtmlWebpackPlugin({template: './src/index.html'})
    ]
};
module.exports = config;
```

### 模式

通过选择 `development`或 `production`之中的一个，来设置`mode`参数，你可以启用相应模式下的webpack内置的优化

```javascript
module.exports = {
    mode: 'production'
}
```

development

会将 `process.env.NODE_ENV` 的值设为 `development`。启用 `NamedChunksPlugin` 和 `NamedModulesPlugin`。

production

会将 `process.env.NODE_ENV` 的值设为 `production`。启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `UglifyJsPlugin`

记住，只设置 `NODE_ENV`，则不会自动设置 `mode`。

```javascript
// mode: development
// webpack.development.config.js
module.exports = {
+ mode: 'development'
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

```javascript
// mode: production
// webpack.production.config.js
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}
```

### 配置

webpack的配置文件，是导出一个对象的javascript文件。此对象，由webpack根据对象定义的属性进行解析

因为webpack配置是标准的Node.js CommonJS模块，所以

1. 通过`require(...)`导入其他文件
2. 通过`require(...)`使用npm的工具函数
3. 使用javascript控制流表达式，例如 `?:`操作符
4. 对常用值使用常量或变量
5. 编写并执行函数来生成部分配置

避免一下做法：

1. 在使用 webpack 命令行接口(CLI)（应该编写自己的命令行接口(CLI)，或[使用 `--env`](https://www.webpackjs.com/configuration/configuration-types/)）时，访问命令行接口(CLI)参数
2. 导出不确定的值（调用 webpack 两次应该产生同样的输出文件）
3. 写很长的配置（应该将配置拆分为多个文件）

**基本配置**

webpack.config.js

```javascript
var path = require('path');

module.exports = {
    mode: 'development',
    entry: 'main.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
};
```

### 模块

在模块化编程中，开发者将程序分解成离散功能块，并称之为 模块

1. ES2015 `import`语句
2. CommonJS `require`语句
3. AMD `define` 和 `require`语句
4. css/sass/less文件中的 `@import`语句
5. 样式（`url(...)`）或HTML文件（`<img src=...>`）中的图片链接

webpack 通过 *loader* 可以支持各种语言和预处理器编写模块。*loader* 描述了 webpack **如何**处理 非 JavaScript(non-JavaScript) _模块_，并且在*bundle*中引入这些_依赖_。

### 构建目标

因为服务器和浏览器代码都可以用javascript编写，所以webpack提供了多种构建目标(target)，你可以在你的webpack配置中设置

```javascript
module.exports = {
    target: 'node'
};
```

多个target

webpack不支持向`target`传入多个字符串，你可以通过打包两份分离的配置来创建同构的库：

```javascript
var path = require('path');

var serverConfig = {
    target: 'node',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'lib.node.js'
    }
};

var clientConfig = {
    target: 'web',  // 默认为'web', 可省略
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'lib.js'
    }
}

module.exports = [serverConfig, clientConfig];

```

### 模块热替换 (hot module replacement)

模块热替换功能会在应用程序过程中替换，添加或删除模块，而无需要重新加载整个页面。主要通过以下几种方式：

1. 保留在完全重新加载页面时丢失的应用程序状态
2. 只更新变更内容，以节省宝贵的开发时间
3. 调整样式更加加速，几乎相当于在浏览器调试器中更改样式

...待续

