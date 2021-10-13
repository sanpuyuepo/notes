

# Webpack 与模块化开发

## 模块化

前端开发范式之一，一种项目组织方式，通过把复杂的代码按照功能划分为不同的模块单独维护，提高开发效率、降低维护成本

webpack本质上是一个模块化打包工具，官方定义：*静态模块打包器(module bundler)*，设计思想：万物皆模块，实现前端项目的模块化，任何资源都可以作为一个模块，任何模块都可以经过Loader机制处理，最终打包到一起。

## webpack两个核心特性

***Loader机制***和***插件机制***

# Webpack背景介绍

## 模块化的演进过程

1. ### **Stage 1 - 文件划分方式**

    将每个功能及相关状态数据各自单独放到不同的JS文件，约定每个文件为一个独立的模块，通过script标签引入调用。

    ```
    └─ stage-1
        ├── module-a.js
        ├── module-b.js
        └── index.html
    ```

    ```javascript
    // module-a.js 
    function foo () {
       console.log('moduleA#foo') 
    }
    ```

    ```javascript
    // module-b.js 
    var data = 'something'
    ```

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <title>Stage 1</title>
    </head>
    <body>
      <script src="module-a.js"></script>
      <script src="module-b.js"></script>
      <script>
        // 直接使用全局成员
        foo() // 可能存在命名冲突
        console.log(data)
        data = 'other' // 数据可能会被修改
      </script>
    </body>
    </html>
    ```

    - 模块直接在全局工作，大量模块成员污染全局作用域；
    - 没有私有空间，所有模块内的成员都可以在模块外部被访问或者修改；
    - 一旦模块增多，容易产生命名冲突；
    - 无法管理模块与模块之间的依赖关系；
    - 在维护的过程中也很难分辨每个成员所属的模块。

2. ###  **Stage 2 – 命名空间方式**

    在第一阶段的基础上，通过将每个模块“包裹”为一个全局对象的形式实现

    ```javascript
    // module-a.js
    window.moduleA = {
      method1: function () {
        console.log('moduleA#method1')
      }
    }
    ```

    ```javascript
    // module-b.js
    window.moduleB = {
      data: 'something'
      method1: function () {
        console.log('moduleB#method1')
      }
    }
    ```

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="UTF-8">
      <title>Stage 2</title>
    </head>
    <body>
      <script src="module-a.js"></script>
      <script src="module-b.js"></script>
      <script>
        moduleA.method1()
        moduleB.method1()
        // 模块成员依然可以被修改
        moduleA.data = 'foo'
      </script>
    </body>
    </html>
    ```

    解决了第一阶段的命名冲突问题

3. ### **Stage 3 – IIFE**

    使用立即执行函数表达式（IIFE，Immediately-Invoked Function Expression）为模块提供私有空间。具体做法是将每个模块成员都放在一个立即执行函数所形成的私有作用域中，对于需要暴露给外部的成员，通过挂到全局对象上的方式实现

    ```javascript
    // module-a.js
    (function () {
      var name = 'module-a'
      function method1 () {
        console.log(name + '#method1')
      }
      // 私有成员只能在模块成员内通过闭包的形式访问，解决了前面所提到的全局作用域污染和命名冲突的问题
      window.moduleA = {
        method1: method1
      }
    })()
    ```

    ```javascript
    // module-b.js
    (function () {
      var name = 'module-b'
      function method1 () {
        console.log(name + '#method1')
      }
      window.moduleB = {
        method1: method1
      }
    })()
    ```

4. ###  Stage 4 - IIFE 依赖参数

    利用 IIFE 参数作为依赖声明使用，这使得每一个模块之间的依赖关系变得更加明显。

    ```javascript
    // module-a.js
    (function ($) { // 通过参数明显表明这个模块的依赖
      var name = 'module-a'
      function method1 () {
        console.log(name + '#method1')
        $('body').animate({ margin: '200px' })
      }
    
      window.moduleA = {
        method1: method1
      }
    })(jQuery)
    ```

## 模块加载的问题

以上 4 个阶段仍然存在一些没有解决的问题，其中最明显的问题是：模块的加载，都是通过 script 标签的方式直接在页面中引入的这些模块，这意味着模块的加载并不受代码的控制，时间久了维护起来会十分麻烦

解决方法：引入一个 JS 入口文件，其余用到的模块可以通过代码控制，按需加载进来

---

## 模块化规范

### CommonJS 规范

Node.js使用该规范，该规范约定：

1. 一个文件就是一个模块，具有单独的作用域，通过module.exports导出成员，通过require函数载入模块
2. 以同步的方式加载模块，Node.js 执行机制是在启动时加载模块，执行过程中只是使用模块

### AMD规范

 AMD （ Asynchronous Module Definition），异步模块定义规范，相关库：Require.js

约定：模块通过 define() 函数定义，默认可以接收两个参数，第一个参数是一个数组，用于声明此模块的依赖项；第二个参数是一个函数，参数与前面的依赖项一一对应，每一项分别对应依赖项模块的导出成员，这个函数的作用就是为当前模块提供一个私有空间。如果在当前模块中需要向外部导出成员，可以通过 return 的方式实现。

<img src="D:\证件照\Cgq2xl6YeWWAZhc-AAIVA96nDrk023.png" style="zoom:50%;" />

Require.js 还提供了一个 require() 函数用于自动加载模块，用法与 define() 函数类似

<img src="D:\证件照\require.png" style="zoom:50%;" />

---

## 模块化的标准规范

- 在 Node.js 环境中遵循 CommonJS 规范来组织模块。
- 在浏览器环境中遵循 ES Modules 规范。

重点掌握 ES Modules 规范：

- [MDN 官方的详细资料]:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules

- [ECMAScript 官方详细资料]:http://www.ecma-international.org/ecma-262/6.0/#sec-modules

---



## 模块打包工具

模块化的问题与解决：

浏览器兼容问题：编译代码  

频繁的网络请求：将散落的模块再打包到一起

除了JS外，HTML及CSS代码等等也需要被模块化：支持不同种类的前端模块类型，也就是说可以将开发过程中涉及的样式、图片、字体等所有资源文件都作为模块使用

<img src="D:\证件照\Ciqah16YeWaAEqbZAAB2uMwv74E224.png" style="zoom:50%;" />

<img src="D:\证件照\CgoCgV6YeWaAHgm3AAB8JgXpadc131.png" style="zoom:50%;" />

<img src="D:\证件照\Cgq2xl6YeWaARx0OAACCLDANJto655.png" style="zoom:50%;" />



---

# Webpack核心特性

## 使用 Webpack 实现模块化打包

对模块化打包方案或工具的设想或者说是诉求：

- 能够将散落的模块打包到一起；

- 能够编译代码中的新特性；

- 能够支持不同种类的前端资源模块

目前，前端领域有一些工具能够很好的满足以上这 3 个需求，其中最为主流的就是 Webpack、Parcel 和 Rollup，对于webpack：

- Webpack 作为一个模块打包工具，本身就可以解决模块化代码打包的问题，将零散的 JavaScript 代码打包到一个 JS 文件中。

- 对于有环境兼容问题的代码，Webpack 可以在打包过程中通过 Loader 机制对其实现编译转换，然后再进行打包。

- 对于不同类型的前端模块类型，Webpack 支持在 JavaScript 中以模块化的方式载入任意类型的资源文件，例如，我们可以通过 Webpack 实现在 JavaScript 中加载 CSS 文件，被加载的 CSS 文件将会通过 style 标签的方式工作
- Webpack 还具备代码拆分的能力，它能够将应用中所有的模块按照我们的需要分块打包，避免全部代码打包到一起，产生过大的单个文件导致加载慢的问题，可以将初次加载必须的模块打包到一起，其他模块单独打包。当应用工作时，异步加载需要用的模块，实现增量加载。

### Webpack 快速上手

```
└─ 02-configuation
   ├── src
   │   ├── heading.js
   │   └── index.js
   └── index.html
```

```javascript
// ./src/heading.js
// 导出函数
export default () => {
  const element = document.createElement('h2')
  element.textContent = 'Hello webpack'
  element.addEventListener('click', () => alert('Hello webpack'))
  return element
}
```

```javascript
// ./src/index.js
// 导入heading.js，并使用该模块
import createHeading from './heading.js'
const heading = createHeading()
document.body.append(heading)
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Webpack - 快速上手</title>
</head>
<body>
    <!-- type="module" 这种用法是 ES Modules 中提出的标准，用来区分加载的是一个普通 JS 脚本还是一个模块 -->
    <script type="module" src="./src/index.js"></script>
</body>
</html>
```

初始化 package.json 文件，管理 npm 依赖版本

```
$ npm init --yes
```

安装 Webpack 

```
$ npm i webpack webpack-cli --save-dev
```

运行打包命令：Webpack 会自动从 src/index.js 文件开始打包，然后根据代码中的模块导入操作，自动将所有用到的模块代码打包到一起

```
$ npx webpack
```

将 Webpack 命令定义到 npm scripts 中，这样每次使用起来会更加方便

```json
"scripts": {
    "build": "webpack"
  },
```

---

### 配置 Webpack 的打包过程

Webpack 4 以后的版本支持零配置的方式直接启动打包，整个过程会按照约定将 src/index.js 作为打包入口，最终打包的结果会存放到 dist/main.js 

通过配置文件的方式修改 Webpack 的默认配置，在项目的根目录下添加一个 webpack.config.js， 是一个运行在 Node.js 环境中的 JS 文件，也就是说我们需要按照 CommonJS 的方式编写代码，这个文件可以导出一个对象，我们可以通过所导出对象的属性完成相应的配置选项

```javascript
// ./webpack.config.js
const path = require('path')

module.exports = {
  entry: './src/main.js',  // entry 属性: 指定 Webpack 打包的入口文件路径
  // output 属性: 设置文件输出位置，必须是一个对象
  output: {
    filename: 'bundle.js', 
    path: path.join(__dirname, 'output')
  }
}
```

更多配置见官方文档https://www.webpackjs.com/configuration/

### 配置文件支持智能提示

通过 import 的方式导入 Webpack 模块中的 Configuration 类型，然后根据类型注释的方式将变量标注为这个类型，这样我们在编写这个对象的内部结构时就可以有正确的智能提示

```javascript
// ./webpack.config.js
import { Configuration } from 'webpack'
/**
 * @type {Configuration}
 */
const config = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  }
}
module.exports = config
```

需要注意的是：我们添加的 import 语句只是为了导入 Webpack 配置对象的类型，这样做的目的是为了标注 config 对象的类型，从而实现智能提示。在配置完成后一定要记得注释掉这段辅助代码，因为在 Node.js 环境中默认还不支持 import 语句

### Webpack 工作模式

Webpack 4 新增了一个工作模式的用法，通过mode属性配置 ，简化了 Webpack 配置的复杂程度。可以理解为针对不同环境的几组预设配置：

- production 模式下，启动内置优化插件，自动优化打包结果，打包速度偏慢；

- development 模式下，自动优化打包速度，添加一些调试过程中的辅助插件；

- none 模式下，运行最原始的打包，不做任何额外处理。

修改 Webpack 工作模式的方式有两种：

- 通过 CLI --mode 参数传入；

- 通过配置文件设置 mode 属性。

### 打包结果运行原理

学习 Webpack 打包后生成的 bundle.js 文件，深入了解 Webpack 是如何把这些模块合并到一起，而且还能正常工作的

vscode 折叠代码：Ctrl-k+Ctrl-0 ， 展开代码：Ctrl-k + Ctrl-j

生产的bundle.js文件：

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);
// ./src/index.js

const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])()
document.body.append(heading)

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
// ./src/heading.js
/* harmony default export */ __webpack_exports__["default"] = (() => {
    const element = document.createElement('h2')
    element.textContent = 'Hello webpack'
    element.addEventListener('click', () => alert('Hello webpack'))
    return element
});

/***/ })
/******/ ]);
```

折叠后：

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210424174059720.png" alt="image-20210424174059720" style="zoom:80%;" />

整体生成的代码其实就是一个立即执行函数（webpack5 为箭头函数形式），这个函数是 Webpack 工作入口（webpackBootstrap），它接收一个 modules 参数，调用时传入了一个数组

展开这个数组，里面的元素均是参数列表相同的函数。这里的函数对应的就是我们源代码中的模块，也就是说每个模块最终被包裹到了这样一个函数中，从而实现模块私有作用域

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210424174535266.png" alt="image-20210424174535266" style="zoom:80%;" />

展开 Webpack 工作入口函数:

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210424180150259.png" alt="image-20210424180150259" style="zoom:100%;" />

最开始定义了一个 installedModules 对象用于存放或者缓存加载过的模块。紧接着定义了一个 require 函数，顾名思义，这个函数是用来加载模块的。再往后就是在 require 函数上挂载了一些其他的数据和工具函数，这些暂时不用关心。

这个函数执行到最后调用了 require 函数，传入的模块 id 为 0，开始加载模块。模块 id 实际上就是模块数组的元素下标，也就是说这里开始加载源代码中所谓的入口模块

---

## 通过 Loader 实现特殊资源加载

### 如何加载资源模块

 Webpack 实现不同种类资源模块（，如CSS文件、图片等）加载的核心就是 Loader。

Webpack 是用 Loader（加载器）来处理每个模块的，而内部默认的 Loader 只能处理 JS 模块，如果需要加载其他类型的模块就需要配置不同的 Loader。

<img src="D:\证件照\Ciqah16gAM2AVBOyAACbAmBWOWM473.png" style="zoom:75%;" />

### 加载器的使用方式

不同的loader需要通过npm分别安装，如CSS-loader：

```
npm install css-loader --save-dev
```

安装完后需要在webpack.confige.js配置文件中添加对应的配置

```javascript
// ./webpack.config.js
module.exports = {
  entry: './src/main.css',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/, // 根据打包过程中所遇到文件路径匹配是否使用这个 loader
        use: 'css-loader' // 指定具体的 loader
      }
    ]
  }
}
```

配置对象的 module 属性中添加一个 rules 数组，用于配置模块规则（配置 loader、解析器等选项），每个规则对象都需要设置两个属性

- test：正则表达式，匹配打包过程中遇到的文件路径
- use：指定匹配到的文件需要使用的loader

### 样式模块加载的问题

css-loader 的作用是将 CSS 模块转换为一个 JS 模块，具体的实现方法是将我们的 CSS 代码 push 到一个数组中，这个数组是由 css-loader 内部的一个模块提供的，但是整个过程并没有任何地方使用到了这个数组。

需要在 css-loader 的基础上再使用一个 **style-loader**，**将 css-loader 中所加载到的所有样式模块，通过创建 style 标签的方式添加到页面上**

一旦配置多个 Loader，执行顺序是从后往前执行的

### 通过 JS 加载资源模块

在 JS 代码中通过 import 语句去加载 CSS 文件

<img src="D:\证件照\CgoCgV6gAY2AMxjdAACTY2OYqEw950.png" style="zoom:80%;" />

 通过 JS 代码去加载的 CSS 模块，css-loader 和 style-loader 仍然可以正常工作。因为 Webpack 在打包过程中会循环遍历每个模块，然后根据配置将每个遇到的模块交给对应的 Loader 去处理，最后再将处理完的结果打包到一起

### 为什么要在 JS 中加载其他资源

Webpack 为什么要在 JS 中载入 CSS 呢？不是应该将样式和行为分离么？

 Webpack 不仅建议在 JavaScript 中引入 CSS，还建议在代码中引入当前业务所需要的任意资源文件。通过 JavaScript 代码去引入资源文件，或者说是建立 JavaScript 和资源文件的依赖关系，具有明显的优势，易于后期维护，因为所有资源的加载都是由 JS 代码控制。

其他常见Loader：

| 名称           | 链接                                             |
| -------------- | ------------------------------------------------ |
| file-loader    | https://webpack.js.org/loaders/file-loader       |
| url-loader     | https://webpack.js.org/loaders/url-loader        |
| babel-loader   | https://webpack.js.org/loaders/babel-loader      |
| style-loader   | https://webpack.js.org/loaders/style-loader      |
| css-loader     | https://webpack.js.org/loaders/css-loader        |
| sass-loader    | https://webpack.js.org/loaders/sass-loader       |
| postcss-loader | https://webpack.js.org/loaders/postcss-loader    |
| eslint-loader  | https://github.com/webpack-contrib/eslint-loader |
| vue-loader     | https://github.com/vuejs/vue-loader              |

---

### 开发一个 Loader

Loader 作为 Webpack 的核心机制，内部的工作原理却非常简单

开发一个可以加载 markdown 文件的加载器，以便可以在代码中直接导入 md 文件。markdown 一般是需要转换为 html 之后再呈现到页面上的，所以需要markdown 文件的加载器导入 md 文件后，直接得到 markdown 转换后的 html 字符串

Webpack 的 Loader 都需要导出一个函数，这个函数就是 Loader 对资源的处理过程，它的输入就是加载到的资源文件内容，输出就是加工后的结果，通过 source 参数接收输入，通过返回值输出

在Webpack 配置文件中添加一个加载器规则：

```javascript
module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.md$/,
                // 直接使用相对路径
                use: './markdown-loader'
            }
        ]
    }
```

<u>Webpack 加载资源文件的过程类似于一个工作管道，可以在这个过程中依次使用多个 Loader，但是最终这个管道结束过后的结果必须是一段标准的 JS 代码字符</u>

![](D:\证件照\CgoCgV6gA8SAfv7-AAA9hfxlofw372.png)

#### 在 Loader 的最后返回一段 JS 代码字符串

```javascript
// ./markdown-loader.js
module.exports = source => {
  // 加载到的模块内容 => '# About\n\nthis is a markdown file.'
  console.log(source)
  // 返回值就是最终被打包的内容
  // return 'hello loader ~'
  return 'console.log("hello loader ~")'
}
```

![image-20210424224126148](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210424224126148.png)

安装一个能够将 Markdown 解析为 HTML 的模块，叫作 **marked**

```
npm install --save-dev marked
```

在 markdown-loader.js 中导入这个模块，然后使用这个模块去解析我们的 source

```javascript
const marked = require('marked')

module.exports = source => {
    // 1. 将 markdown 转换为 html 字符串
    const html = marked(source)
    // 将 html 字符串拼接为一段导出字符串的 JS 代码
    const code = `module.exports = ${JSON.stringify(html)}`
    return code
}
```

如果直接拼接，HTML 中的换行和引号就都可能会造成语法错误，所以先通过 JSON.stringify() 将字段字符串转换为标准的 JSON 字符串，然后再参与拼接

除了 module.exports 这种方式，Webpack 还允许我们在返回的代码中使用 ES Modules 的方式导出，例如，我们这里将 module.exports 修改为 export default，然后运行打包，结果同样是可以的，Webpack 内部会自动转换 ES Modules 代码

```javascript
const marked = require('marked')

module.exports = source => {
    // 1. 将 markdown 转换为 html 字符串
    const html = marked(source)
    // 将 html 字符串拼接为一段导出字符串的 JS 代码
    // const code = `module.exports = ${JSON.stringify(html)}`
    const code = `export default ${JSON.stringify(html)}`
    return code
}
```

#### 多个 Loader 的配合

 markdown-loader 中直接返回 HTML 字符串，然后交给下一个 Loader 处理

```javascript
// ./markdown-loader.js
const marked = require('marked')

module.exports = source => {
  // 1. 将 markdown 转换为 html 字符串
  const html = marked(source)
  return html
}
```

再安装一个处理 HTML 的 Loader，叫作 html-loader

```javascript
module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.md$/,
                // 直接使用相对路径
                use: [
                    'html-loader',
                    './markdown-loader'
                ]
            }
        ]
    }
```

---

## **利用插件机制横向扩展 Webpack 的构建能力**

Webpack 插件机制的目的是为了增强 Webpack 在项目自动化构建方面的能力。

Loader 就是负责完成项目中各种各样资源模块的加载，从而实现整体项目的模块化，而 Plugin 则是用来解决项目中除了资源模块打包以外的其他自动化工作

几个插件最常见的应用场景：

- 实现自动在打包之前清除 dist 目录（上次的打包结果）；
- 自动生成应用所需要的 HTML 文件；
- 根据不同环境为代码注入类似 API 地址这种可能变化的部分；
- 拷贝不需要参与打包的资源文件到输出目录；
- 压缩 Webpack 打包完成后输出的文件；
- 自动发布打包结果到服务器实现自动部署。

---



### clean-webpack-plugin

自动在打包之前清除 dist 目录（上次的打包结果）

安装：

```
npm install clean-webpack-plugin --save-dev
```

在 webpack.config.js 配置文件中配置：

```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin()
  ]
}
```

---

###  html-webpack-plugin

用于生成 HTML 的插件, 自动生成使用打包结果(在 HTML 中自动注入 Webpack 打包生成的 bundle)的 HTML

不同于 clean-webpack-plugin，html-webpack-plugin 插件默认导出的就是插件类型，不需要再解构内部成员:

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin()
  ]
}
```

- 对于生成的 HTML 文件，页面 title 必须要修改；
- 很多时候还需要我们自定义页面的一些 meta 标签和一些基础的 DOM 结构

如果只是简单的自定义，我们可以通过修改 HtmlWebpackPlugin 的参数来实现:

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
        title: 'Webpack Plugin Sample'
    })
  ]
}
```

如果要对 HTML 进行大量的自定义，更好的做法是在源代码中添加一个用于生成 HTML 的模板，让 html-webpack-plugin 插件根据这个模板去生成页面文件，具体做法是：

在src 目录下新建一个 index.html 文件作为 HTML 文件的模板，然后根据需要在这个文件中添加相应的元素。对于模板中动态的内容，可以使用 Lodash 模板语法输出，模板中可以通过 htmlWebpackPlugin.options 访问这个插件的配置数据

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>

<body>
    <div class="container">
        <h1>页面上的基础结构</h1>
        <div id="root"></div>
    </div>
</body>

</html>
```

有了模板文件过后，回到配置文件中，通过 HtmlWebpackPlugin 的 <u>**template 属性**</u>指定所使用的模板

```javascript
plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
        title: 'Webpack Plugin Sample',
        template: './src/index.html'     
    })
  ]
```

如果需要同时输出多个 HTML 文件，其实也非常简单，配置文件中通过 HtmlWebpackPlugin 创建的对象就是用于生成 index.html 的，完全可以再创建一个新的实例对象，用于创建额外的 HTML 文件。

```javascript
plugins: [
    new CleanWebpackPlugin(),
    // 用于生成 index.html
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      template: './src/index.html'
    }),
    // 用于生成 about.html
    new HtmlWebpackPlugin({
      filename: 'about.html'
    })
  ]
```

对于同时输出多个 HTML，一般我们还会配合 Webpack 多入口打包的用法，这样就可以让不同的 HTML 使用不同的打包结果

---



### copy-webpack-plugin

不需要参与构建的静态文件，最终也需要发布到线上，例如网站的 favicon、robots.txt 等

一般建议，把这类文件统一放在项目根目录下的 public 或者 static 目录中，Webpack 在打包时一并将这个目录下所有的文件复制到输出目录

```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin')
plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Webpack Plugin Sample',
      template: './src/index.html'
    }),
    new CopyWebpackPlugin({
        // 这个插件类型的构造函数需要我们传入一个字符串数组，用于指定需要拷贝的文件路径。它可以是一个通配符，也可以是一个目录或者文件的相对路径。
        // 这里传入的是 public 目录，表示将这个目录下所有文件全部拷贝到输出目录中
        patterns: ['public'] // 需要拷贝的目录或者路径通配符
    })
  ]
```

---



### 开发一个插件

Webpack 的插件机制就是最常见的**<u>钩子机制</u>**

钩子机制有点类似于 Web 中的事件。在 Webpack 整个工作过程会有很多环节，为了便于插件的扩展，Webpack 几乎在每一个环节都埋下了一个钩子。在开发插件的时候，通过往这些不同节点上挂载不同的任务，就可以轻松扩展 Webpack 的能力。

<img src="https://s0.lgstatic.com/i/image3/M01/16/A6/Ciqah16mU4KAX07hABBXsBqlv1U403.gif" alt="4.gif" style="zoom:80%;" />

预先定义好的钩子，参考官方文档的 API：

- [Compiler Hooks](https://webpack.js.org/api/compiler-hooks/)；
- [Compilation Hooks](https://webpack.js.org/api/compilation-hooks/)；
- [JavascriptParser Hooks](https://webpack.js.org/api/parser/)。

需求：开发一个插件，能够自动清除 Webpack 打包结果中的注释，这样一来， bundle.js 将更容易阅读

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425003648964.png" alt="image-20210425003648964" style="zoom:80%;" />

首先明确：插件必须是一个函数或者是一个包含 apply 方法的对象。

定义一个类型，在这个类型中定义 apply 方法。然后在使用时，再通过这个类型来创建一个实例对象去使用这个插件。

所以，定义一个RemoveCommentsPlugin类型，在该类型中定义一个apply方法，该方法在Webpack启动时调用，接受一个compiler对象参数，compiler对象是Webpack中最核心的对象，包含所有配置信息，通过该对象去注册钩子函数：

```javascript
// ./remove-comments-plugin.js
class RemoveCommentsPlugin {
  apply (compiler) {
    console.log('RemoveCommentsPlugin 启动')
    // compiler => 包含了我们此次构建的所有配置信息
  }
}

module.exports=RemoveCommentsPlugin
```

还需要明确我们这个任务的执行时机，也就是到底应该把这个任务挂载到哪个钩子上。

需求是删除 bundle.js 中的注释，也就是说只有当 Webpack 需要生成的 bundle.js 文件内容明确过后才可能实施。

据 API 文档中的介绍，找到一个叫作 emit 的钩子，这个钩子会在 Webpack 即将向输出目录输出文件时执行，非常符合我们的需求

通过 compiler 对象的 **hooks 属性**访问到 **emit 钩子**，再通过 **tap 方法**注册一个钩子函数，这个方法接收两个参数：

- 第一个是插件的名称，我们这里的插件名称是 RemoveCommentsPlugin；
- 第二个是要挂载到这个钩子上的函数；

根据 API 文档中的提示，这里我们在这个函数中接收一个 compilation 对象参数，这个对象可以理解为此次运行打包的上下文，所有打包过程中产生的结果，都会放到这个对象中。

我们可以使用这个对象中的 assets 属性获取即将写入输出目录的资源文件信息，它是一个对象，我们这里通过 for in 去遍历这个对象，其中键就是每个文件的名称，我们尝试把它打印出来，

```javascript
class RemoveCommentsPlugin {
    apply(compiler) {
        console.log('RemoveCommentsPlugin 启动');
        // compiler => 包含了我们此次构建的所有配置信息
        compiler.hooks.emit.tap('RemoveCommentsPlugin', compilation => {
            // compilation => 可以理解为此次打包的上下文
            for (const name in compilation.assets) {
                // 先判断文件名是不是以 .js 结尾，因为 Webpack 打包还有可能输出别的文件，而我们的需求只需要处理 JS 文件
                
                console.log(name) // 输出文件名称
                console.log(compilation.assets[name].source()) // 输出文件内容
            }
        })
    }
}

module.exports=RemoveCommentsPlugin
```

***插件都是通过往 Webpack 生命周期的钩子中挂载任务函数实现的***

---

# **Webpack 运行机制与核心工作原理**

## 工作过程简介

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425011854542.png" alt="image-20210425011854542" style="zoom:50%;" />

Webpack 在整个打包的过程中：

- 通过 Loader 处理特殊类型资源的加载，例如加载样式、图片；
- 通过 Plugin 实现各种自动化的构建任务，例如自动压缩、自动发布。

具体打包过程：

1. Webpack 启动后，根据配置，找到项目中的某个指定文件（一般这个文件都会是一个 JS 文件）作为入口
2. 根据入口文件代码中出现的 import（ES Modules）或者是 require（CommonJS）之类的语句，解析推断出来这个文件所依赖的资源模块
3. 分别去解析每个资源模块的依赖，周而复始，最后形成整个项目中所有用到的文件之间的依赖关系树
4. 遍历（递归）这个依赖树，找到每个节点对应的资源文件
5. 根据配置选项中的 Loader 配置，交给对应的 Loader 去加载这个模块，将加载的结果放入 bundle.js（打包结果）中，从而实现整个项目的打包

![](D:\证件照\Ciqc1F6pSHiAbuTBACPS6wVVqZw547.gif)

![](D:\证件照\Ciqc1F6pUBeAfHWtAG70TcGBhSM152.gif)

依赖模块中无法通过 JavaScript 代码表示的资源模块，例如图片或字体文件，一般的 Loader 会将它们单独作为资源文件拷贝到输出目录中，然后将这个资源文件所对应的访问路径作为这个模块的导出成员暴露给外部。



---

## 工作原理剖析



### 一、Webpack CLI

### 二、创建 Compiler 对象

### 三、开始构建

### 四、make 阶段



---

# Webpack 高阶

## **使用 Dev Server 提高本地开发效率**

### Webpack 自动编译

每次修改完代码，都是通过命令行手动重复运行 Webpack 命令，从而得到最新的打包结果，那么这样的操作过程根本没有任何开发体验可言。

针对上述这个问题，我们可以使用 Webpack CLI 提供的另外一种 watch 工作模式来解决

watch模式下，Webpack 完成初次构建过后，项目中的源文件会被监视，一旦发生任何改动，Webpack 都会自动重新运行打包任务

用法：启动 Webpack 时，添加一个 --watch 的 CLI 参数

```
webpack --watch
```

BrowserSync 就可以帮我们实现文件变化过后浏览器自动刷新的功能，使用 BrowserSync 工具替换 serve 工具，启动 HTTP 服务

```
# 可以先通过 npm 全局安装 browser-sync 模块，然后再使用这个模块
$ npm install browser-sync --global
$ browser-sync dist --watch

# 或者也可以使用 npx 直接使用远端模块
$ npx browser-sync dist --watch
```

Webpack 监视源代码变化，自动打包源代码到 dist 中，而 dist 中文件的变化又被 BrowserSync 监听了，从而实现自动编译并且自动刷新浏览器的功能，整个过程由两个工具分别监视不同的内容。

watch 模式 + BrowserSync 虽然也实现了我们的需求，但是这种方法有很多弊端：

- 操作烦琐，我们需要同时使用两个工具，那么需要了解的内容就会更多，学习成本大大提高；
- 效率低下，因为整个过程中， Webpack 会将文件写入磁盘，BrowserSync 再进行读取。过程中涉及大量磁盘读写操作，必然会导致效率低下。

所以这只能算是“曲线救国”，并不完美，我们仍然需要继续改善。

---

#### Webpack Dev Server

提供了一个开发服务器，并且将自动编译和自动刷新浏览器等一系列对开发友好的功能全部集成在了一起，是一个高度集成的工具，使用起来十分的方便

webpack-dev-server 同样也是一个独立的 npm 模块，所以我们需要通过 npm 将 webpack-dev-server 作为项目的开发依赖安装。安装完成过后，这个模块为我们提供了一个叫作 webpack-dev-server 的 CLI 程序，我们同样可以直接通过 npx 直接去运行这个 CLI，或者把它定义到 npm scripts 中

运行 webpack-dev-server 这个命令时，它内部会启动一个 HTTP Server，为打包的结果提供静态文件服务，并且自动使用 Webpack 打包我们的应用，然后监听源代码的变化，一旦文件发生变化，它会立即重新打包，大致流程如下：

<img src="D:\证件照\CgqCHl6ykrKAKqeOAABe6Avstu0065.png" style="zoom:80%;" />

webpack-dev-server 为了提高工作速率，它并没有将打包结果写入到磁盘中，而是暂时存放在内存中，内部的 HTTP Server 也是从内存中读取这些文件的。这样一来，就会减少很多不必要的磁盘读写操作，大大提高了整体的构建效率



---

#### 配置选项

Webpack 配置对象中可以有一个叫作 devServer 的属性，专门用来为 webpack-dev-server 提供配置，

```JavaScript
devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000
```



---

#### 静态资源访问

通过 Webpack 打包能够输出的文件都可以直接被访问到。但是如果你还有一些没有参与打包的静态文件也需要作为开发服务器的资源被访问，那你就需要额外通过配置“告诉” webpack-dev-server

通过这个 devServer 对象的 contentBase 属性指定额外的静态资源路径。这个 contentBase 属性可以是一个字符串或者数组，也就是说你可以配置一个或者多个路径。

---

#### Proxy 代理

由于 webpack-dev-server 是一个本地开发服务器，所以我们的应用在开发阶段是独立运行在 localhost 的一个端口上，而后端服务又是运行在另外一个地址上。但是最终上线过后，我们的应用一般又会和后端服务部署到同源地址下。

这样就会出现一个非常常见的问题：在实际生产环境中能够直接访问的 API，回到我们的开发环境后，再次访问这些 API 就会产生跨域请求问题。

可以用跨域资源共享（CORS）解决这个问题。确实如此，如果我们请求的后端 API 支持 CORS，那这个问题就不成立了。但是并不是每种情况下服务端的 API 都支持 CORS。如果前后端应用是同源部署，也就是协议 / 域名 / 端口一致，那这种情况下，根本没必要开启 CORS，所以跨域请求的问题仍然是不可避免的。

解决这种开发阶段跨域请求问题最好的办法，就是在开发服务器中配置一个后端 API 的代理服务，也就是把后端接口服务代理到本地的开发服务地址。

<u>**webpack-dev-server 支持直接通过配置的方式，添加代理服务**</u>

假定 GitHub 的 API 就是我们应用的后端服务，那我们的目标就是将 GitHub API 代理到本地开发服务器中。如：

<img src="C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425024007721.png" alt="image-20210425024007721" style="zoom:80%;" />

在 devServer 配置属性中添加一个 **proxy 属性**，这个属性值需要是一个对象，对象中的每个属性就是一个代理规则配置。

属性的名称是需要被代理的请求路径前缀，一般为了辨别，我都会设置为 /api。值是所对应的代理规则配置，我们将代理目标地址设置为 https://api.github.com，

```javascript
// ./webpack.config.js
module.exports = {
  // ...
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.github.com'
      }
    }
  }
}
```

此时我们请求 http://localhost:8080/api/users ，就相当于请求了 https://api.github.com/api/users

而真正希望请求的地址是 https://api.github.com/users，所以对于代理路径开头的 /api要重写掉。可以添加一个 **pathRewrite 属性**来<u>实现代理路径重写</u>，重写规则就是把路径中开头的 /api 替换为空，pathRewrite 最终会以正则的方式来替换请求路径。

```javascript
// ./webpack.config.js
module.exports = {
  // ...
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.github.com',
        pathRewrite: {
          '^/api': '' // 替换掉代理地址中的 /api
        }
      }
    }
  }
}
```

除此之外，还需设置一个 **changeOrigin 属性**为 true。这是因为默认代理服务器会以实际在浏览器中请求的主机名，也就是 localhost:8080 作为代理请求中的主机名。而一般服务器需要根据请求的主机名判断是哪个网站的请求，localhost:8080 这个主机名，对于 GitHub 的服务器来说，肯定无法正常请求，所以需要修改。

将代理规则配置的 changeOrigin 属性设置为 true，就会以实际代理请求地址中的主机名去请求，也就是我们正常请求这个地址的主机名是什么，实际请求 GitHub 时就会设置成什么

```JavaScript
// ./webpack.config.js
module.exports = {
  // ...
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.github.com',
        pathRewrite: {
          '^/api': '' // 替换掉代理地址中的 /api
        },
        changeOrigin: true // 确保请求 GitHub 的主机名就是：api.github.com
      }
    }
  }
```

此时，我们就可以回到代码中使用代理后的本地同源地址去请求后端接口，而不必担心出现跨域问题了。

---

## **配置 Webpack SourceMap 的最佳实践**

通过构建或者编译之类的操作，我们将开发阶段编写的源代码转换为能够在生产环境中运行的代码，这种进步同时也意味着我们实际运行的代码和我们真正编写的代码之间存在很大的差异

在这种情况下，如果需要调试我们的应用，或是应用运行的过程中出现意料之外的错误，那我们将无从下手。因为无论是调试还是报错，都是基于构建后的代码进行的，我们只能看到错误信息在构建后代码中具体的位置，却很难直接定位到源代码中对应的位置。

### Source Map 简介

source map的作用：映射转换后的代码与源代码之间的关系。一段转换后的代码，通过转换过程中生成的 Source Map 文件就可以逆向解析得到对应的源代码。

---

### Webpack 中配置 Source Map

在webpack配置文件中，通过 devtool 属性来配置开发过程中的辅助工具，也就是与 Source Map 相关的一些功能。我们可以先将这个属性设置为 source-map，

```JavaScript
// ./webpack.config.js
module.exports = {
  devtool: 'source-map' // source map 设置
}
```

打包：生成 bundle.js 的 Source Map 文件，与此同时 bundle.js 中也会通过注释引入这个 Source Map 文件

![image-20210425025734792](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425025734792.png)

通过 serve 工具把打包结果运行起来，然后打开浏览器，再打开开发人员工具，此时我们就可以直接定位到错误所在的位置了。当然如果需要调试，这里也可以直接调试源代码

 Webpack 支持的 Source Map 模式有很多种。每种模式下所生成的 Source Map 效果和生成速度都不一样。显然，效果好的一般生成速度会比较慢，而生成速度快的一般就没有什么效果，

Webpack 中的 devtool 配置，除了可以使用 source-map 这个值，它还支持很多其他的选项，

![](D:\证件照\Ciqc1F65B2aAGTvVAANPGIkqtEY706.png)

---

#### Eval 模式

eval 是 JavaScript 中的一个函数，可以用来运行字符串中的 JavaScript 代码。例如下面这段代码，字符串中的 console.log("foo~") 就会作为一段 JavaScript 代码被执行：

```JavaScript
const code = 'console.log("foo~")'
eval(code) // 将 code 中的字符串作为 JS 代码执行
```

![image-20210425093327087](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425093327087.png)

默认通过eval执行的代码运行在一个临时的环境中，如上面的 VM126：1

可以通过 sourceURL 来声明这段代码所属文件路径，在 eval 函数执行的字符串代码中添加一个注释，注释的格式：# sourceURL=./path/to/file.js，这样的话这段代码就会执行在指定路径下

![image-20210425093742432](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425093742432.png)

 Webpack 的配置文件中，将 devtool 属性设置为 eval，打包过后，找到生成的 bundle.js 文件，你会发现每个模块中的代码都被包裹到了一个 eval 函数中，而且每段模块代码的最后都会通过 sourceURL 的方式声明这个模块对应的源文件路径，

![](D:\证件照\Ciqc1F65BaCAaChSAALB1bStzyo434.png)

一旦出现错误，浏览器的控制台就可以定位到具体是哪个模块中的代码，具体效果如下

![](D:\证件照\CgqCHl65BauAc85YAAFTP1qAxo4213.png)

但是当你点击控制台中的文件名打开这个文件后，看到的却是打包后的模块代码，而并非我们真正的源代码，

![image-20210425095239564](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425095239564.png)

在 eval 模式下，Webpack 会将每个模块转换后的代码都放到 eval 函数中执行，并且通过 sourceURL 声明对应的文件路径，这样浏览器就能知道某一行代码到底是在源代码的哪个文件中

在 eval 模式下并不会生成 Source Map 文件，所以它的构建速度最快，但是缺点同样明显：它只能定位源代码的文件路径，无法知道具体的行列信息

---

### 案例准备工作

为了可以更好地对比不同模式的 Source Map 之间的差异，这里我们使用一个新项目，同时创建出不同模式下的打包结果，通过具体实验来横向对比它们之间的差异

```
└─ 07-devtool-diff
   ├── src
   │   ├── heading.js
   │   └── main.js
   ├── package.json
   └── webpack.config.js
```

Webpack 的配置文件除了可以导出一个配置对象，还可以导出一个数组，数组中每一个元素就是一个单独的打包配置，那这样就可以在一次打包过程中同时执行多个打包任务

```JavaScript
const config = [{
        entry: './src/main.js',
        output: {
            filename: 'output1.js'
        }
    },
    {
        entry: './src/main.js',
        output: {
            filename: 'output2.js'
        }
    }
]
module.exports = config;
```

这么配置的话，再次打包就会有两个打包子任务工作，我们的 dist 中生成的结果也就是两个文件

![image-20210425101308157](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425101308157.png)

回到配置文件，遍历刚才定义的allDevtoolModes， 为每一个模式单独创建一个打包配置

![](D:\证件照\CgqCHl65BgCAbZVrAAUumD0yd4g992.png)



---

### 不同模式的对比

eval-source-map 的模式，这个模式也是使用 eval 函数执行模块代码，不过这里有所不同的是，eval-source-map 模式除了定位文件，还可以定位具体的行列信息。相比于 eval 模式，它能够生成 Source Map 文件，

![](D:\证件照\Ciqc1F65LV2ABnPSAAOQByCZ1X8805.gif)

cheap-eval-source-map 的模式。根据这个模式的名字就能推断出一些信息，它就是在 eval-source-map 基础上添加了一个 cheap，也就是便宜的，或者叫廉价的。用计算机行业的常用说法，就是阉割版的 eval-source-map，因为它虽然也生成了 Source Map 文件，但是这种模式下的 Source Map 只能定位到行，而定位不到列，所以在效果上差了一点点，但是构建速度会提升很多，

![](D:\证件照\CgqCHl65LXKAL1X9AAQvU6NQre0545.gif)

cheap-module-eval-source-map 的模式。慢慢地我们就发现 Webpack 中这些模式的名字不是随意的，好像都有某种规律。这里就是在 cheap-eval-source-map 的基础上多了一个 module，

![](D:\证件照\Ciqc1F65LYKAb35fAAWpO16gIpE536.gif)

这种模式同样也只能定位到行，它的特点相比于 cheap-eval-source-map 并不明显 ，如果你没有发现差异，可以再去看看上一种模式，仔细做一个对比，相信对比之后你会发现，cheap-module-eval-source-map 中定位的源代码与我们编写的源代码是一模一样的，而 cheap-eval-source-map 模式中定位的源代码是经过 ES6 转换后的结果

![](D:\证件照\Ciqc1F65BjSAE-KLAAFGXAIp67I615.png)

cheap-source-map 模式，这个模式的名字中没有 eval，意味着它没用 eval 执行代码，而名字中没有 module，意味着 Source Map 反推出来的是 Loader 处理后的代码，有 cheap 表示只能定位源代码的行号

几个特殊一点的模式，我们单独介绍一下：

- inline-source-map 模式

它跟普通的 source-map 效果相同，只不过这种模式下 Source Map 文件不是以物理文件存在，而是以 data URLs 的方式出现在代码中。我们前面遇到的 eval-source-map 也是这种 inline 的方式。

- hidden-source-map 模式

在这个模式下，我们在开发工具中看不到 Source Map 的效果，但是它也确实生成了 Source Map 文件，这就跟 jQuery 一样，虽然生成了 Source Map 文件，但是代码中并没有引用对应的 Source Map 文件，开发者可以自己选择使用。

- nosources-source-map 模式：

在这个模式下，我们能看到错误出现的位置（包含行列位置），但是点进去却看不到源代码。这是为了保护源代码在生产环境中不暴露。

**开发过程中（开发环境），建议选择 cheap-module-eval-source-map**，原因有以下三点：

- 我使用框架的情况会比较多，以 React 和 Vue.js 为例，无论是 JSX 还是 vue 单文件组件，Loader 转换后差别都很大，我需要调试 Loader 转换前的源代码。
- 一般情况下，我编写的代码每行不会超过 80 个字符，对我而言能够定位到行到位置就够了，而且省略列信息还可以提升构建速度。
- 虽然在这种模式下启动打包会比较慢，但大多数时间内我使用的 webpack-dev-server 都是在监视模式下重新打包，它重新打包的速度非常快。

**发布前的打包，也就是生产环境的打包，建议选择 none，它不会生成 Source Map**。原因很简单：

- 首先，Source Map 会暴露我的源代码到生产环境。如果没有控制 Source Map 文件访问权限的话，但凡是有点技术的人都可以很容易的复原项目中涉及的绝大多数源代码，这非常不合理也不安全，我想很多人可能都忽略了这个问题。
- 其次，调试应该是开发阶段的事情，你应该在开发阶段就尽可能找到所有问题和隐患，而不是到了生产环境中再去全民公测。如果你对自己的代码实在没有信心，我建议你选择 nosources-source-map 模式，这样出现错误可以定位到源码位置，也不至于暴露源码。

Source Map 并不是 Webpack 特有的功能，它们两者的关系只是：Webpack 支持 Source Map。大多数的构建或者编译工具也都支持 Source Map。



---

## **模块支持热替换（HMR）机制**

### 自动刷新的问题

每次修改完代码，Webpack 都可以监视到变化，然后自动打包，再通知浏览器自动刷新，一旦页面整体刷新，那页面中的任何操作状态都将会丢失

更好的办法自然是能够实现在页面不刷新的情况下，代码也可以及时的更新到浏览器的页面中，重新执行，避免页面状态丢失。针对这个需求，Webpack 同样可以满足

---

### 模块热替换（HMR）

HMR 全称 Hot Module Replacement，翻译过来叫作“模块热替换”或“模块热更新

Webpack 中的模块热替换，指的是我们可以在应用运行过程中，实时的去替换掉应用中的某个模块，而应用的运行状态不会因此而改变， 例如我们在应用运行过程中修改了某个模块，通过自动刷新会导致整个应用的整体刷新，那页面中的状态信息都会丢失；而如果使用的是 HMR，就可以实现只将修改的模块实时替换至应用中，不必完全刷新整个应用，

有了 HMR 支持后，例如在页面中随意添加一些内容，也就是为页面制造一些运行状态，然后我们回到开发工具中，再来尝试修改文本的样式，保存过后页面并没有整体刷新，而且我们能立即看到最新的样式。



---

#### 开启 HMR

HMR 已经集成在了 webpack 模块中

使用这个特性最简单的方式就是，在运行 webpack-dev-server 命令时，通过 --hot 参数去开启这个特性

也可以在配置文件中通过添加对应的配置来开启这个功能：

- 首先需要将 devServer 对象中的 hot 属性设置为 true；
- 然后需要载入一个插件，这个插件是 webpack 内置的一个插件，所以我们先导入 webpack 模块，有了这个模块过后，这里使用的是一个叫作 HotModuleReplacementPlugin 的插件。

```JavaScript
// ./webpack.config.js
const webpack = require('webpack')
module.exports = {
  // ...
  devServer: {
    // 开启 HMR 特性，如果资源不支持 HMR 会 fallback 到 live reloading
    hot: true
    // 只使用 HMR，不会 fallback 到 live reloading
    // hotOnly: true
  },
  plugins: [
    // ...
    // HMR 特性所需要的插件
    new webpack.HotModuleReplacementPlugin()
  ]
}

```



---

#### HMR 的疑问

HMR 并不像 Webpack 的其他特性一样可以开箱即用，需要有一些额外的操作，Webpack 中的 HMR 需要我们手动通过代码去处理，当模块更新过后该，如何把更新后的模块替换到页面中

Q1：为什么我们开启 HMR 过后，样式文件的修改就可以直接热更新？

A1：这是因为样式文件是经过 Loader 处理的，在 style-loader 中就已经自动处理了样式文件的热更新，所以就不需要我们额外手动去处理了。

Q2：为什么样式就可以自动处理，而脚本就需要手动处理？

A2：因为样式模块更新过后，只需要把更新后的 CSS 及时替换到页面中，它就可以覆盖掉之前的样式，从而实现更新。而 JavaScript 模块是没有任何规律的，你可能导出的是一个对象，也可能导出的是一个字符串，还可能导出的是一个函数，使用时也各不相同。所以 Webpack 面对这些毫无规律的 JS 模块，根本不知道该怎么处理更新后的模块，也就无法直接实现一个可以通用所有情况的模块替换方案。

Q3：那可能还有一些平时使用 vue-cli 或者 create-react-app 这种框架脚手架工具的人会说，“我的项目就没有手动处理，JavaScript 代码照样可以热替换，也没你说的那么麻烦”。

A3：这是因为你使用的是框架，使用框架开发时，我们项目中的每个文件就有了规律，例如 React 中要求每个模块导出的必须是一个函数或者类，那这样就可以有通用的替换办法，所以这些工具内部都已经帮你实现了通用的替换操作，自然就不需要手动处理了。

---

#### HMR APIs & JS 模块热替换 & 热替换的状态保持

**<u>HotModuleReplacementPlugin</u>** 为我们的 JavaScript 提供了一套用于处理 HMR 的 API，我们需要在我们自己的代码中，使用这套 API 将更新后的模块替换到正在运行的页面中

对于开启 HMR 特性的环境中，我们可以访问到全局的 module 对象中的 hot 成员，这个成员是一个对象，这个对象就是 HMR API 的核心对象，它提供了一个 accept 方法，用于注册当某个模块更新后的处理函数。

**accept 方法**第一个参数接收的就是所监视的依赖模块路径，第二个参数就是依赖模块更新后的处理函数

 **JS 模块热替换** ：

```JavaScript
// ./src/index.js
import './style.css'
import createHeading from './heading.js'
import createEditor from './editor.js'
import about from './about.md'

console.log('app starts running~');
console.log(about)

const heading = createHeading()
document.body.appendChild(heading)

const editor = createEditor();
document.body.appendChild(editor);

// 记录下来每次热替换创建的新元素，以便于下一次热替换时的操作
let lastHeading = heading
let lastEditor = editor

if (module.hot) {
    module.hot.accept('./heading.js', () => {
        // 当 ./heading.js 更新，自动执行此函数
        console.log('heading 更新了～～')
        document.body.removeChild(lastHeading) // 移除之前创建的元素
        lastHeading = createHeading() // 用新模块创建新元素
        document.body.appendChild(lastHeading)
    })
    
    // 热替换的状态保持：用户在界面上输入一些内容（形成页面操作状态），如 input , 热替换操作在替换时把状态保留下来 
    module.hot.accept('./editor.js', () => {
        // 当 ./editor.js 更新，自动执行此函数
        // 临时记录更新前编辑器内容
        const value = lastEditor.value
        document.body.removeChild(lastEditor) // 移除之前创建的元素
        lastEditor = createEditor() // 用新模块创建新元素
        // 还原编辑器内容
        lastEditor.value = value
        document.body.appendChild(lastEditor)
    })
}
```

为什么 Webpack 需要我们自己处理 JS 模块的热更新：

因为不同的模块有不同的情况，不同的情况，在这里处理时肯定也是不同的。就好像，我们这里是一个文本编辑器应用，所以需要保留状态，如果不是这种类型那就不需要这样做。所以说 Webpack 没法提供一个通用的 JS 模块替换方案

---

#### 图片模块热替换

同样通过 module.hot.accept 注册这个图片模块的热替换处理函数，在这个函数中，我们只需要重新给图片元素的 src 设置更新后的图片路径就可以了。因为图片修改过后图片的文件名会发生变化，而这里我们就可以直接得到更新后的路径，所以重新设置图片的 src 就能实现图片热替换

---

#### 常见问题

1. <u>如果处理热替换的代码（处理函数）中有错误，结果也会导致自动刷新</u>。

    - 原因: HMR 过程报错导致 HMR 失败，HMR 失败过后，会自动回退到自动刷新，页面一旦自动刷新，控制台中的错误信息就会被清除，这样的话，如果不是很明显的错误，就很难被发现
    - 解决：使用 hotOnly 的方式来解决，因为现在使用的 hot 方式，如果热替换失败就会自动回退使用自动刷新，而 hotOnly 的情况下并不会使用自动刷新，配置文件中，将 devServer 中的 hot 等于 true 修改为 hotOnly 等于 true

2. <u>使用了 HMR API 的代码，在没有开启 HMR 功能的情况下运行 Webpack 打包，此时运行环境中就会报出 Cannot read property 'accept' of undefined</u> 

    - 原因： module.hot 是 HMR 插件提供的成员，没有开启这个插件，自然也就没有这个对象

    - 解决：先判断是否存在这个对象，再去使用

        ![image-20210425235814463](C:\Users\ajiai\AppData\Roaming\Typora\typora-user-images\image-20210425235814463.png)

3. <u>在代码中写了很多与业务功能本身无关的代码，会不会对生产环境有影响？</u>

    不会对生产环境有任何影响

---

关于框架的 HMR，因为在大多数情况下是开箱即用的，所以这里不做过多介绍，详细可以参考：

- [React HMR 方案](https://github.com/gaearon/react-hot-loader)；
- [Vue.js HMR 方案](https://vue-loader.vuejs.org/guide/hot-reload.html)。

---

## **Webpack 高级特性**

 Tree Shaking 和 sideEffects。它们都属于 Webpack 打包结果优化的必备特性，

### Tree Shaking

Tree Shaking ，“摇树”。伴随着摇树的动作，树上的枯树枝和树叶就会掉落下来。

Webpack中的 Tree-shaking 也是同样的道理，不过通过 Tree-shaking “摇掉”的是代码中那些没有用到的部分，这部分没有用的代码更专业的说法应该叫作未引用代码（dead-code）。

使用 Webpack 生产模式打包的优化过程中，就使用自动开启这个功能，以此来检测我们代码中的未引用代码，然后自动移除它们

Webpack 的 Tree-shaking 特性在**生产模式**下会**<u>自动开启</u>**

---

#### 开启 Tree Shaking

在其他模式下，如何一步一步手动开启 Tree-shaking。通过这个过程，还可以顺便了解 Tree-shaking 的工作过程和 Webpack 其他的一些优化功能。

在 Webpack 的配置文件中，在配置对象中添加一个 optimization 属性，这个属性用来集中配置 Webpack 内置优化功能，它的值也是一个对象， optimization 对象中我们可以先开启一个 usedExports 选项，表示在输出结果中只导出外部使用了的成员

对于这种未引用代码，如果我们开启压缩代码功能，就可以自动压缩掉这些没有用到的代码

```JavaScript
 optimization: {
        // 模块只导出被使用的成员
        usedExports: true,
        // 压缩输出结果
        minimize: true
    },
```

#### 合并模块（扩展）

除了 usedExports 选项之外，我们还可以使用一个 concatenateModules 选项继续优化输出。

普通打包只是将一个模块最终放入一个单独的函数中，如果我们的模块很多，就意味着在输出结果中会有很多的模块函数。

**concatenateModules** 配置的作用就是尽可能将所有模块合并到一起输出到一个函数中，这样既提升了运行效率，又减少了代码的体积

配置文件中，在 optimization 属性中开启 concatenateModules

```javascript
 optimization: {
        // 模块只导出被使用的成员
        usedExports: true,
        // 压缩输出结果
        minimize: true
        concatenateModules: true,
    },
```

此时 bundle.js 中就不再是一个模块对应一个函数了，而是把所有的模块都放到了一个函数中,这个特性又被称为 Scope Hoisting，也就是作用域提升，它是 Webpack 3.0 中添加的一个特性。

如果再配合 minimize 选项，打包结果的体积又会减小很多

#### 结合 babel-loader 的问题

早期的 Webpack 发展非常快，那变化也就比较多，所以当我们去找资料时，得到的结果不一定适用于当前我们所使用的版本。而 Tree-shaking 的资料更是如此，很多资料中都表示“*为 JS 模块配置 babel-loader，会导致 Tree-shaking 失效*”。

针对这个问题，首先你需要明确一点：

**Tree-shaking 实现的前提是 ES Modules**，也就是说：最终**交给 Webpack 打包的代码，必须是使用 ES Modules 的方式来组织的模块化**。

Webpack 在打包所有的模块代码之前，先是将模块根据配置交给不同的 Loader 处理，最后再将 Loader 处理的结果打包到一起

很多时候，我们为了更好的兼容性，会选择使用 [babel-loader](https://github.com/babel/babel-loader) 去转换我们源代码中的一些 ECMAScript 的新特性。而 Babel 在转换 JS 代码时，很有可能处理掉我们代码中的 ES Modules 部分，把它们转换成 CommonJS 的方式

当然了，Babel 具体会不会处理 ES Modules 代码，取决于我们有没有为它配置使用转换 ES Modules 的插件。

很多时候，我们为 Babel 配置的都是一个 preset（预设插件集合），而不是某些具体的插件。例如，目前市面上使用最多的 [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)，这个预设里面就有[转换 ES Modules 的插件](https://babeljs.io/docs/en/babel-plugin-transform-modules-commonjs)。所以当我们使用这个预设时，代码中的 ES Modules 部分就会被转换成 CommonJS 方式。那 Webpack 再去打包时，拿到的就是以 CommonJS 方式组织的代码了，所以 Tree-shaking 不能生效。

在最新版本（8.x）的 babel-loader 中，已经自动帮我们关闭了对 ES Modules 转换的插件,所以**经过 babel-loader 处理后的代码默认仍然是 ES Modules，那 Webpack 最终打包得到的还是 ES Modules 代码，Tree-shaking 自然也就可以正常工作了**。

最新版本的 babel-loader 并不会导致 Tree-shaking 失效。如果你不确定现在使用的 babel-loader 会不会导致这个问题，最简单的办法就是在配置中将 @babel/preset-env 的 modules 属性设置为 false，确保不会转换 ES Modules，也就确保了 Tree-shaking 的前提。

---

### sideEffects

Webpack 4 中新增了一个 sideEffects 特性，它允许我们通过配置标识我们的代码是否有副作用，从而提供更大的压缩空间。

TIPS：模块的副作用指的就是模块执行的时候除了导出成员，是否还做了其他的事情。

这个特性一般只有我们去开发一个 npm 模块时才会用到





---

### Code Splitting（分块打包）

#### All in One 的弊端

通过 Webpack 实现前端项目整体模块化的优势固然明显，但是它也会存在一些弊端：它最终会将我们所有的代码打包到一起。试想一下，如果我们的应用非常复杂，模块非常多，那么这种 All in One 的方式就会导致打包的结果过大，甚至超过 4～5M。

在绝大多数的情况下，应用刚开始工作时，并不是所有的模块都是必需的。如果这些模块全部被打包到一起，即便应用只需要一两个模块工作，也必须先把 bundle.js 整体加载进来，而且前端应用一般都是运行在浏览器端，这也就意味着应用的响应速度会受到影响，也会浪费大量的流量和带宽。

所以这种 All in One 的方式并不合理，更为合理的方案是**把打包的结果按照一定的规则分离到多个 bundle 中，然后**根据**应用的运行需要按需加载**。这样就可以降低启动成本，提高响应速度。

Web 应用中的资源受环境所限，太大不行，太碎更不行。因为我们开发过程中划分模块的颗粒度一般都会非常的细，很多时候一个模块只是提供了一个小工具函数，并不能形成一个完整的功能单元。

如果我们不将这些资源模块打包，直接按照开发过程中划分的模块颗粒度进行加载，那么运行一个小小的功能，就需要加载非常多的资源模块。

再者，目前主流的 HTTP 1.1 本身就存在一些缺陷，例如：

- 同一个域名下的并行请求是有限制的；
- 每次请求本身都会有一定的延迟；
- 每次请求除了传输内容，还有额外的请求头，大量请求的情况下，这些请求头加在一起也会浪费流量和带宽。

综上所述，模块打包肯定是必要的，但当应用体积越来越大时，我们也要学会变通。

---

#### Code Splitting

为了解决打包结果过大导致的问题，Webpack 设计了一种分包功能：Code Splitting（代码分割）。

Code Splitting 通过把项目中的资源模块按照我们设计的规则打包到不同的 bundle 中，从而降低应用的启动成本，提高响应速度。

Webpack 实现分包的方式主要有两种：

- 根据业务不同配置多个打包入口，输出多个打包结果；
- 结合 ES Modules 的动态导入（Dynamic Imports）特性，按需加载模块。

##### 多入口打包

多入口打包一般适用于传统的多页应用程序，最常见的划分规则就是一个页面对应一个打包入口，对于不同页面间公用的部分，再提取到公共的结果中

一般 entry 属性中只会配置一个打包入口，如果我们需要配置多个入口，可以把 entry 定义成一个对象。

> 注意：这里 entry 是定义为对象而不是数组，如果是数组的话就是把多个文件打包到一起，还是一个入口。

入口配置为多入口形式，那输出文件名也需要修改，因为两个入口就有两个打包结果，不能都叫 bundle.js。我们可以在这里使用 [name] 这种占位符来输出动态的文件名，[name] 最终会被替换为入口的名称。

通过 HtmlWebpackPlugin 的 chunks 属性来设置。我们分别为两个页面配置使用不同的 chunk，

> TIPS：每个打包入口都会形成一个独立的 chunk（块）

```javascript
// ./webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: {
    index: './src/index.js',
    album: './src/album.js'
  },
  output: {
    filename: '[name].bundle.js' // [name] 是入口名称
  },
  // ... 其他配置
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/index.html',
      filename: 'index.html',
      chunks: ['index'] // 指定使用 index.bundle.js
    }),
    new HtmlWebpackPlugin({
      title: 'Multi Entry',
      template: './src/album.html',
      filename: 'album.html',
      chunks: ['album'] // 指定使用 album.bundle.js
    })
  ]
}
```

多入口打包本身非常容易理解和使用，但是它也存在一个小问题，就是不同的入口中一定会存在一些公共使用的模块，如果按照目前这种多入口打包的方式，就会出现多个打包结果中有相同的模块的情况。

还需要把这些公共的模块提取到一个单独的 bundle 中。Webpack 中实现公共模块提取非常简单，我们只需要在优化配置中开启 splitChunks 功能就可以了

```javascript
 optimization: {
    splitChunks: {
      // 自动提取所有公共模块到单独 bundle
      chunks: 'all'
    }
  }
```

在 optimization 属性中添加 splitChunks 属性，那这个属性的值是一个对象，这个对象需要配置一个 chunks 属性，我们这里将它设置为 all，表示所有公共模块都可以被提取。

---

#### 动态导入

除了多入口打包的方式，Code Splitting 更常见的实现方式还是结合 ES Modules 的动态导入特性，从而实现按需加载，在应用运行过程中，需要某个资源模块时，才去加载这个模块。这种方式极大地降低了应用启动时需要加载的资源体积，提高了应用的响应速度，同时也节省了带宽和流量。

Webpack 中支持使用动态导入的方式实现模块的按需加载，而且所有动态导入的模块都会被自动提取到单独的 bundle 中，从而实现分包

可以通过代码中的逻辑去控制需不需要加载某个模块，或者什么时候加载某个模块。而且我们分包的目的中，很重要的一点就是让模块实现按需加载，从而提高应用的响应速度

<img src="D:\证件照\CgqCHl7Ez--AA39kACvspSgItKU114.gif" style="zoom:80%;" />

```
├── src
│   ├── album
│   │   ├── album.css
│   │   └── album.js
│   ├── common
│   │   ├── fetch.js
│   │   └── global.css
│   ├── posts
│   │   ├── posts.css
│   │   └── posts.js
│   ├── index.html
│   └── index.js
├── package.json
└── webpack.config.js
```

文章列表对应的是这里的 posts 组件，而相册列表对应的是 album 组件。我在打包入口（index.js）中同时导入了这两个模块，然后根据页面锚点的变化决定显示哪个组件，核心代码如下

```javascript
// ./src/index.js
import posts from './posts/posts'
import album from './album/album'

const update = () => {
  const hash = window.location.hash || '#posts'
  const mainElement = document.querySelector('.main')
  mainElement.innerHTML = ''
  if (hash === '#posts') {
    mainElement.appendChild(posts())
  } ese if (hash === '#album') {
    mainElement.appendChild(album())
  }
}
window.addEventListener('hashchange', update)
update()
```

在这种情况下，就可能产生资源浪费。试想一下：如果用户只需要访问其中一个页面，那么加载另外一个页面对应的组件就是浪费。

如果我们采用动态导入的方式，就不会产生浪费的问题了，因为所有的组件都是惰性加载，只有用到的时候才会去加载。具体实现代码如下：

```javascript
// ./src/index.js
// import posts from './posts/posts'
// import album from './album/album'
const update = () => {
  const hash = window.location.hash || '#posts'
  const mainElement = document.querySelector('.main')
  
  mainElement.innerHTML = ''
  if (hash === '#posts') {
    // mainElement.appendChild(posts())
    import('./posts/posts').then(({ default: posts }) => {
      mainElement.appendChild(posts())
    })
  } else if (hash === '#album') {
    // mainElement.appendChild(album())
    import('./album/album').then(({ default: album }) => {
      mainElement.appendChild(album())
    })
  }
}
window.addEventListener('hashchange', update
update()
```

P.S. 为了动态导入模块，可以将 import 关键字作为函数调用。当以这种方式使用时，import 函数返回一个 Promise 对象。这就是 ES Modules 标准中的 [Dynamic Imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports)。

这里我们先移除 import 这种静态导入，然后在需要使用组件的地方通过 import 函数导入指定路径，那这个方法返回的是一个 Promise。在这个 Promise 的 then 方法中我们能够拿到模块对象。由于我们这里的 posts 和 album 模块是以默认成员导出，所以我们需要解构模块对象中的 default，先拿到导出成员，然后再正常使用这个导出成员。



整个过程我们无需额外配置任何地方，只需要按照 ES Modules 动态导入的方式去导入模块就可以了，Webpack 内部会自动处理分包和按需加载。

如果你使用的是 Vue.js 之类的 SPA 开发框架的话，那你项目中路由映射的组件就可以通过这种动态导入的方式实现按需加载，从而实现分包。

##### 魔法注释

默认通过动态导入产生的 bundle 文件，它的 name 就是一个序号，这并没有什么不好，因为大多数时候，在生产环境中我们根本不用关心资源文件的名称。

但是如果你还是需要给这些 bundle 命名的话，就可以使用 Webpack 所特有的魔法注释去实现。具体方式如下：

```javascript
// 魔法注释
import(/* webpackChunkName: 'posts' */'./posts/posts')
  .then(({ default: posts }) => {
    mainElement.appendChild(posts())
  })
```

所谓魔法注释，就是在 import 函数的形式参数位置，添加一个行内注释，这个注释有一个特定的格式：webpackChunkName: ''，这样就可以给分包的 chunk 起名字了。

除此之外，魔法注释还有个特殊用途：如果你的 chunkName 相同的话，那相同的 chunkName 最终就会被打包到一起，例如我们这里可以把这两个 chunkName 都设置为 components，然后再次运行打包，那此时这两个模块都会被打包到一个文件中，

---

## **优化 Webpack 的构建速度和打包结果**

生产环境和开发环境有很大的差异，在生产环境中我们强调的是以更少量、更高效的代码完成业务功能，也就是注重运行效率。而开发环境中我们注重的只是开发效率。

### 不同环境下的配置

创建不同环境配置的方式主要有两种：

- 在配置文件中添加相应的判断条件，根据环境不同导出不同配置；
- 为不同环境单独添加一个配置文件，一个环境对应一个配置文件。

1. 在配置文件中添加判断的方式：

    Webpack 配置文件还支持导出一个函数，然后在函数中返回所需要的配置对象。这个函数可以接收两个参数，第一个是 env，是我们通过 CLI 传递的环境名参数，第二个是 argv，是运行 CLI 过程中的所有参数。具体代码如下：

    ```javascript
    // ./webpack.config.js
    module.exports = (env, argv) => {
      return {
        // ... webpack 配置
      }
    }
    ```

    先将不同模式下公共的配置定义为一个 config 对象，具体代码如下：

    ```JavaScript
    // ./webpack.config.js
    module.exports = (env, argv) => {
      const config = {
        // ... 不同模式下的公共配置
      }
      return config
    }
    ```

    然后通过判断，再为 config 对象添加不同环境下的特殊配置。具体如下

    ```JavaScript
    // ./webpack.config.js
    module.exports = (env, argv) => {
      const config = {
        // ... 不同模式下的公共配置
      }
    
      if (env === 'development') {
        // 为 config 添加开发模式下的特殊配置
        config.mode = 'development'
        config.devtool = 'cheap-eval-module-source-map'
      } else if (env === 'production') {
        // 为 config 添加生产模式下的特殊配置
        config.mode = 'production'
        config.devtool = 'nosources-source-map'
      }
    
      return config
    }
    ```

    通过这种方式完成配置过后，在命令行终端执行 webpack 命令时就可以通过 --env 参数去指定具体的环境名称，从而实现在不同环境中使用不同的配置。

    这种方式是 Webpack 建议的方式。

    也可以直接定义环境变量，然后在全局判断环境变量，根据环境变量的不同导出不同配置。

2. 为不同环境单独添加一个配置文件：

    通过判断环境名参数返回不同配置对象的方式只适用于中小型项目，因为一旦项目变得复杂，我们的配置也会一起变得复杂起来。所以对于大型的项目来说，还是建议使用不同环境对应不同配置文件的方式来实现。

    一般在这种方式下，项目中最少会有三个 webpack 的配置文件。其中两个用来分别适配开发环境和生产环境，另外一个则是公共配置。因为开发环境和生产环境的配置并不是完全不同的，所以需要一个公共文件来抽象两者相同的配置。具体配置文件结构如下：

    ```
    ├── webpack.common.js ···························· 公共配置
    ├── webpack.dev.js ······························· 开发模式配置
    └── webpack.prod.js ······························ 生产模式配置
    ```

    在项目根目录下新建一个 webpack.common.js，在这个文件中导出不同模式下的公共配置；然后再来创建一个 webpack.dev.js 和一个 webpack.prod.js 分别定义开发和生产环境特殊的配置。

    在不同环境的具体配置中我们先导入公共配置对象，可以使用 `Object.assign` 方法把公共配置对象复制到具体环境的配置对象中，并且同时去覆盖其中的一些配置。具体如下：

    ```JavaScript
    // ./webpack.common.js
    module.exports = {
      // ... 公共配置
    }
    // ./webpack.prod.js
    const common = require('./webpack.common')
    module.exports = Object.assign(common, {
      // 生产模式配置
    })
    // ./webpack.dev.js
    const common = require('./webpack.common')
    module.exports = Object.assign(common, {
      // 开发模式配置
    })
    ```

     Object.assign 方法，会完全覆盖掉前一个对象中的同名属性。这个特点对于普通值类型属性的覆盖都没有什么问题。但是像配置中的 plugins 这种数组，我们只是希望在原有公共配置的插件基础上添加一些插件，那 Object.assign 就做不到了。

    所以我们需要更合适的方法来合并这里的配置与公共的配置。你可以使用 [Lodash](http://lodash.com/) 提供的 merge 函数来实现，不过社区中提供了更为专业的模块 [webpack-merge](https://github.com/survivejs/webpack-merge)，它专门用来满足我们这里合并 Webpack 配置的需求。

    ```javascript
    // ./webpack.common.js
    module.exports = {
      // ... 公共配置
    }
    // ./webpack.prod.js
    const merge = require('webpack-merge')
    const common = require('./webpack.common')
    module.exports = merge(common, {
      // 生产模式配置
    })
    // ./webpack.dev.jss
    const merge = require('webpack-merge')
    const common = require('./webpack.common')
    module.exports = merge(common, {
      // 开发模式配置
    })
    ```

    分别配置完成过后，命令行终端打包，通过 --config 参数来指定我们所使用的配置文件路径例如：

    ```
    webpack --config webpack.prod.js
    ```

---

### 生产模式下的优化插件

学习 production 模式下几个主要的优化功能，顺便了解一下 Webpack 如何优化打包结果

#### Define Plugin

作用：为代码中注入全局成员，在 production 模式下，默认通过这个插件往代码中注入了一个 process.env.NODE_ENV。很多第三方模块都是通过这个成员去判断运行环境，从而决定是否执行例如打印日志之类的操作。

DefinePlugin 是一个内置的插件，所以我们先导入 webpack 模块，然后再到 plugins 中添加这个插件。这个插件的构造函数接收一个对象参数，对象中的成员都可以被注入到代码中。具体代码如下：

```JavaScript
// ./webpack.config.js
const webpack = require('webpack')
module.exports = {
  // ... 其他配置
  plugins: [
    new webpack.DefinePlugin({
        // 传入一个字符串字面量语句
        API_BASE_URL: '"https://api.example.com"'
        // 这里有一个非常常用的小技巧，如果我们需要注入的是一个值，就可以通过 JSON.stringify 的方式来得到表示这个值的字面量。这样就不容易出错了。
        // API_BASE_URL: JSON.stringify('https://api.example.com')
    })
  ]
}
```

---

#### Mini CSS Extract Plugin

CSS 文件的打包，一般我们会使用 style-loader 进行处理，这种处理方式最终的打包结果就是 CSS 代码会内嵌到 JS 代码中

mini-css-extract-plugin 是一个可以将 CSS 代码从打包结果中提取出来的插件，它的使用非常简单，同样也需要先通过 npm 安装一下这个插件

```javascript
// ./webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 'style-loader', // 将样式通过 style 标签注入
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin()
  ]
}
```

先导入这个插件模块，导入过后我们就可以将这个插件添加到配置对象的 plugins 数组中了。这样 Mini CSS Extract Plugin 在工作时就会自动提取代码中的 CSS 了。

除此以外，Mini CSS Extract Plugin 还需要我们使用 MiniCssExtractPlugin 中提供的 loader 去替换掉 style-loader，以此来捕获到所有的样式。

这样的话，打包过后，样式就会存放在独立的文件中，直接通过 link 标签引入页面。

不过这里需要注意的是，如果你的 CSS 体积不是很大的话，提取到单个文件中，效果可能适得其反，因为单独的文件就需要单独请求一次。个人经验是 如果单个CSS 文件超过 200KB 才需要考虑是否提取出来，作为单独的文件。

---

#### Optimize CSS Assets Webpack Plugin

使用了 Mini CSS Extract Plugin 过后，样式就被提取到单独的 CSS 文件中了，但是样式文件并没有被压缩。

因为Webpack 内置的压缩插件仅仅是针对 JS 文件的压缩，其他资源文件的压缩都需要额外的插件。

Webpack 官方推荐了一个 [Optimize CSS Assets Webpack Plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) 插件。我们可以使用这个插件来压缩我们的样式文件。

```JavaScript
// ./webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin(),
    new OptimizeCssAssetsWebpackPlugin()
  ]
}
```

不过这里还有个额外的小点，可能你会在这个插件的官方文档中发现，文档中的这个插件并不是配置在 plugins 数组中的，而是添加到了 optimization 对象中的 minimizer 属性中，那这是为什么呢？

其实也很简单，如果我们配置到 plugins 属性中，那么这个插件在任何情况下都会工作。而配置到 minimizer 中，就只会在 minimize 特性开启时才工作。

所以 Webpack 建议像这种压缩插件，应该我们配置到 minimizer 中，便于 minimize 选项的统一控制。

但是这么配置也有个缺点，此时我们再次运行生产模式打包，打包完成后再来看一眼输出的 JS 文件，此时你会发现，原本可以自动压缩的 JS，现在却不能压缩了。这是因为我们设置了 minimizer，Webpack 认为我们需要使用自定义压缩器插件，那内部的 JS 压缩器就会被覆盖掉。我们必须手动再添加回来。

内置的 JS 压缩插件叫作 terser-webpack-plugin，我们回到命令行手动安装一下这个模块，再手动添加这个模块到 minimizer 配置当中，这样的话，我们再次以生产模式运行打包，JS 文件和 CSS 文件就都可以正常压缩了

```JavaScript
// ./webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
const TerserWebpackPlugin = require('terser-webpack-plugin')
module.exports = {
  mode: 'none',
  entry: {
    main: './src/index.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  optimization: {
    minimizer: [
      new TerserWebpackPlugin(),
      new OptimizeCssAssetsWebpackPlugin()
    ]
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin()
  ]
}
```

