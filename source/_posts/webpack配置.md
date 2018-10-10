---
title: webpack配置
date: 2018-07-30 08:47:43
tags:
---

# webpack安装

要安装最新版或特定版本，请运行以下命令之一：

```
npm install --save-dev webpack
npm install --save-dev webpack@<version> // 安装指定版本
```

使用webpack4+版本，还需要安装CLI

```
npm install --save-dev webpack-cli
```

一般会在package.json写执行的webpack

```json
"scripts": {
    "start": "webpack --config webpack.config.js"
}
```

全局安装

```
npm install --global webpack
```

# loader加载

### 加载css

为了从 JavaScript 模块中 `import` 一个 CSS 文件，你需要在 [`module` 配置中](https://www.webpackjs.com/configuration/module) 安装并添加 [style-loader](https://www.webpackjs.com/loaders/style-loader) 和 [css-loader](https://www.webpackjs.com/loaders/css-loader)

```
npm install --save-dev style-loader css-loader
```

### 加载图片  压缩和优化你的图像。查看 [image-webpack-loader](https://github.com/tcoopman/image-webpack-loader) 和 [url-loader](https://www.webpackjs.com/loaders/url-loader)

```
npm install --save-dev file-loader
```

### 加载字体

```
npm install --save-dev file-loader
```

### 加载数据

此外，可以加载的有用资源还有数据，如 JSON 文件，CSV、TSV 和 XML。类似于 NodeJS，JSON 支持实际上是内置的，也就是说 `import Data from './data.json'` 默认将正常运行。要导入 CSV、TSV 和 XML，你可以使用 [csv-loader](https://github.com/theplatapi/csv-loader) 和 [xml-loader](https://github.com/gisikw/xml-loader)

```
npm install --save-dev csv-loader xml-loader
```

收到

```javascript
const CleanWebpackPlugin = require('clean-webpack-plugin'); // 清除dist 文件夹
plugins: [
        //每次构建前清理 / dist 文件夹
        new CleanWebpackPlugin(['dist']),
    ],
```

当 webpack 打包源代码时，可能会很难追踪到错误和警告在源代码中的原始位置.为了更容易地追踪错误和警告，JavaScript 提供了 [source map](http://blog.teamtreehouse.com/introduction-source-maps) 功能，将编译后的代码映射回原始源代码

```javascript
const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
+   devtool: 'inline-source-map',  // 仅解释说明，不要用于生产环境
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Development'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

每次要编译代码时，手动运行 `npm run build` 就会变得很麻烦。

webpack 中有几个不同的选项，可以帮助你在代码发生变化后自动编译代码：

1. webpack's Watch Mode
2. webpack-dev-server
3. webpack-dev-middleware

观察者模式

```javascript
// package.json
{
    "name": "development",
    "version": "1.0.0",
    "description": "",
    "main": "webpack.config.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
+     "watch": "webpack --watch", // 添加--watch 进行观察
      "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "clean-webpack-plugin": "^0.1.16",
      "css-loader": "^0.28.4",
      "csv-loader": "^2.1.1",
      "file-loader": "^0.11.2",
      "html-webpack-plugin": "^2.29.0",
      "style-loader": "^0.18.2",
      "webpack": "^3.0.0",
      "xml-loader": "^1.2.1"
    }
  }
// 唯一的缺点是，为了看到修改后的实际效果，你需要刷新浏览器
```

使用webpack-dev-server  推荐使用

`webpack-dev-server` 为你提供了一个简单的 web 服务器，并且能够实时重新加载(live reloading)

```
npm install --save-dev webpack-dev-server // 先安装webpack-dev-server
```

修改配置文件，告诉开发服务器(dev server)，在哪里查找文件

```javascript
const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    devtool: 'inline-source-map',
+   devServer: {
+     contentBase: './dist'
+   },
    plugins: [
      new CleanWebpackPlugin(['dist']),
      new HtmlWebpackPlugin({
        title: 'Development'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };

//以上配置告知 webpack-dev-server，在 localhost:8080 下建立服务，将 dist 目录下的文件，作为可访问文件
```

让我们添加一个 script 脚本，可以直接运行开发服务器(dev server)

```javascript
// package.json 
{
    "name": "development",
    "version": "1.0.0",
    "description": "",
    "main": "webpack.config.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "watch": "webpack --watch",
+     "start": "webpack-dev-server --open",
      "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "clean-webpack-plugin": "^0.1.16",
      "css-loader": "^0.28.4",
      "csv-loader": "^2.1.1",
      "file-loader": "^0.11.2",
      "html-webpack-plugin": "^2.29.0",
      "style-loader": "^0.18.2",
      "webpack": "^3.0.0",
      "xml-loader": "^1.2.1"
    }
  }
```

tree shaking

*tree shaking* 是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)

math.js

```javascript
export function square(x) {
  return x * x;
}

export function cube(x) {
  return x * x * x;
}
```

math.js导出两函数

index.js

```javascript
import { cube } from './math.js'; // 只导入了cube方法  没有square方法
console.log(5 cubed is equal to ' + cube(5))
```

但是在打包过后的bundle.js中还是存在square方法

```javascript
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* unused harmony export square */
/* harmony export (immutable) */ __webpack_exports__["a"] = cube;
function square(x) {
  return x * x;
}

function cube(x) {
  return x * x * x;
}
```

所以要去掉这些没有用到部分，通过`package.json`的`sideEffects`属性来实现

```javascript
{
  "name": "your-project",
  "sideEffects": false
}
//我们就可以简单地将该属性标记为 false，来告知 webpack，它可以安全地删除未用到的 export 导出。

// 如果你的代码确实有一些副作用，那么可以改为提供一个数组
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```

压缩输出

从 webpack 4 开始，也可以通过 `"mode"` 配置选项轻松切换到压缩输出，只需设置为 `"production"`。

webpack-merge区分生成环境和开发环境