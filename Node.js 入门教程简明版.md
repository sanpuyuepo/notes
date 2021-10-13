

# Node.js 入门教程简明版

## Node.js简介

1. 开源与跨平台的JavaScript运行时环境，一个底层平台
2. 使用V8引擎
3. 运行于单个进程，无需为每个请求创建新线程
4. 提供异步的I/O原生功能
5. 拥有大量的库：npm仓库

------

## 与浏览器的异同

相同：均使用 JavaScript 作为其编程语言

不同：

1. Node.js使用单一语言JavaScript编程前端和后端
2. 浏览器中操作的时DOM和WebAPI，Node.js中不存在浏览器提供的对象，如document、window等
3. 浏览器中不存在Node.js模块提供的api，如文件系统访问功能
4. Node.js可以控制运行环境，可以使用最新标准的ECMAScript来编写代码
5. Node.js使用CommonJS模块系统，即在Node.js中使用require()，浏览器中使用import

------

## 命令行运行Node.js脚本

```powershell
node app.js
```

------

## 终止Node.js应用程序

控制台中：使用`ctrl-c`

代码中：使用`process`模块提供的方法或属性

1. process.exit()方法，可以传入一个整数，向操作系统发送退出码

    ```javascript
    process.exit(1)
    ```

     默认情况下，退出码为 `0`，表示成功。有关退出码的信息，详见 http://nodejs.cn/api/process.html#process_exit_codes

2. 设置process.exitCode属性，程序结束时，Node.js 会返回该退出码。

    ```javascript
    process.exitCode = 1;
    ```

3.   process.on()方法发送SIGTERM 信号

    ```javascript
    const express = require('express')
    
    const app = express()
    
    app.get('/', (req, res) => {
      res.send('你好')
    })
    
    const server = app.listen(3000, () => console.log('服务器已就绪'))
    // 向该命令()发送 SIGTERM 信号，并使用进程的信号处理程序进行处理
    process.on('SIGTERM', () => {
      server.close(() => {
        console.log('进程已终止')
      })
    })
    ```

------

## 读取环境变量

`process` 核心模块提供了 `env` 属性，该属性承载了在启动进程时设置的所有环境变量

```javascript
process.env.NODE_ENV // "development"
```

------

## Node.js REPL

Node.js REPL(Read Eval Print Loop:交互式解释器) 表示一个电脑的环境，类似 Window 系统的终端或 Unix/Linux shell

node 命令省略文件名，进入REPL模式：

```powershell
PS D:\MyProject\node> node
Welcome to Node.js v12.18.2.      
Type ".help" for more information.
> console.log('testing')
testing
undefined
>
```

`undefined`是运行 `console.log()` 的返回值。

1. tab键自动补全

2. 输入 JavaScript 类的名称，添加一个点号并按下 `tab`，REPL 会打印可以在该类上访问的所有属性和方法

    ```powershell
    > String.
    String.__defineGetter__      String.__defineSetter__      String.__lookupGetter__      String.__lookupSetter__      String.__proto__             String.hasOwnProperty        String.isPrototypeOf
    String.propertyIsEnumerable  String.toLocaleString        String.valueOf
    
    String.apply                 String.arguments             String.bind                  String.call                  String.caller                String.constructor           String.toString
    
    String.fromCharCode          String.fromCodePoint         String.length                String.name                  String.prototype             String.raw
    ```

3. 输入 `global.` 并按下 `tab`，可以检查可以访问的全局变量

4. ## 点命令：以点号 `.` 开头

    - `.help`
    - `.editor`: 进入编辑模式，编写JavaScript代码，ctrl-D运行代码
    - `.break`: 相当于ctrl-C
    - `.clear`: 将 REPL 上下文重置为空对象并清除当前正在输入的表达式
    - `.load`: 加载 JavaScript 文件
    - `.save`: 将在 REPL 会话中输入的所有内容保存到文件（需指定文件名）
    - `.exit`: 退出 REPL

------

## 命令行接收参数

使用`node`命令执行程序时，可以传入任意数量的参数（参数可以是独立的，也可以具有键和值），如：

```powershell
node app.js joe
node app.js name=joe
```

传入之后怎么获取呢？

使用内置的process对象：

```javascript
// process.argv属性, 是一个参数数组, 
// 数组第一项: node命令完整路径
// 第二项：正被执行的文件的完整路径
process.argv.forEach((val, index) => {
  console.log(`${index}: ${val}`)
})

/*
排除前两项
*/
const args = process.argv.slice(2)
```

独立参数：

```powershell
node app.js joe
```

独立参数获取：

```javascript
const args = process.argv.slice(2)
args[0]
```

具有键和值的参数：

```powershell
node app.js name=joe
```

具有键和值的参数获取：

```javascript
// args[0] 是 name=joe，需要对其进行解析。 最好的方法是使用 minimist 库，该库有助于处理参数：
const args = require('minimist')(process.argv.slice(2))
args['name'] //joe
```

使用minimist库时，需要在每个参数名称之前使用双破折号

```powershell
node app.js --name=joe
```

------

## 输出到命令行

Node.js 提供了 `console` 模块，本上与浏览器中的 `console` 对象相同

1. `console.log()`

    ```javascript
    const x = 'x'
    const y = 'y'
    console.log(x, y)
    
    console.log('我的%s已经%d岁', '猫', 2)
    ```

2. `console.clear()`: 清除控制台

3. `console.count()`: 元素计数

    ```javascript
    const x = 1
    const y = 2
    const z = 3
    console.count(
      'x 的值为 ' + x + ' 且已经检查了几次？'
    )
    console.count(
      'x 的值为 ' + x + ' 且已经检查了几次？'
    )
    console.count(
      'y 的值为 ' + y + ' 且已经检查了几次？'
    )
    ```

4. `console.trace()`: 打印堆栈踪迹

    ```javascript
    const function2 = () => console.trace()
    const function1 = () => function2()
    function1()
    ```

5. `time()`/`timeEnd()`: 计算函数运行所需时间

    ```javascript
    const doSomething = () => console.log('测试')
    const measureDoingSomething = () => {
      console.time('doSomething()')
      //做点事，并测量所需的时间。
      doSomething()
      console.timeEnd('doSomething()')
    }
    measureDoingSomething()
    ```

控制台中创建进度条 : 使用 [Progress](https://www.npmjs.com/package/progress) ，npm install progress

------

## 从命令行接收输入

Node.js 提供了 [`readline` 模块](http://nodejs.cn/api/readline.html)来执行以下操作：每次一行地从可读流（例如 `process.stdin` 流，在 Node.js 程序执行期间该流就是终端输入）获取输入

```javascript
const readline = require('readline').createInterface({
    input: process.stdin,
    output: process.stdout
})
// question() 方法会显示第一个参数（即问题），并等待用户的输入
readline.question(`你叫什么名字?`, name => {
    console.log(`你好 ${name}!`)
    readline.close()
})
```

---

## exports导入导出

```javascript
const library = require('./library')
```

1. 将对象赋值给 `module.exports`（这是模块系统提供的对象），这会使文件只导出该对象:

    ```javascript
    const car = {
      brand: 'Ford',
      model: 'Fiesta'
    }
    
    module.exports = car
    
    //在另一个文件中
    
    const car = require('./car')
    ```

2. 将要导出的对象添加为 `exports` 的属性。这种方式可以导出多个对象、函数或数据:

    ```javascript
    const car = {
      brand: 'Ford',
      model: 'Fiesta'
    }
    
    exports.car = car
    
    // 或者
    exports.car = {
      brand: 'Ford',
      model: 'Fiesta'
    }
    
    // 另一个文件中
    const items = require('./items')
    items.car
    // 或
    const car = require('./items').car
    ```

**`module.exports` 和 `export` 之间有什么区别？**

前者公开了它指向的对象。 后者公开了它指向的对象的属性

---

## npm 简介

Node.js 标准的软件包管理器

1. **安装**：**npm install**: 

    - `--save` 安装并添加条目到 `package.json` 文件的 dependencies。
    - `--save-dev` 安装并添加条目到 `package.json` 文件的 devDependencies。

    

    默认情况下，当输入 `npm install` 命令时，软件包会被安装到当前文件树中的 `node_modules` 子文件夹下，并在当前文件夹中存在的 `package.json` 文件的 `dependencies` 属性中添加软件包的 条目

    使用 `-g` 标志可以执行全局安装，全局的位置到底在哪里？

    使用`npm root -g`查看

    

    <u>通常</u>，<u>所有的软件包都应本地安装</u>

    <u>但是某些软件包最好在全局安装</u>，如：npm, vue-cli, grunt-cli等

    # 查看 npm 包安装的版本:

    - **npm list**  

        查看所有已安装的 npm 软件包（包括它们的依赖包）的最新版本

    - **npm list --depth=0**

        仅获取顶层的软件包（基本上就是告诉 npm 要安装并在 `package.json` 中列出的软件包

    - **npm list [package_name]** 

        指定名称来获取特定软件包的版本

    - **npm view [package_name] version**

        查看软件包在 npm 仓库上最新的可用版本

    # 安装 npm 包的旧版本

    - ```powershell
        npm install <package>@<version>
        ```

    - 列出软件包所有的以前的版本

        ```powershell
        npm view <package> versions
        ```

2. **更新**：**npm update**

    如果有新的次版本或补丁版本，并且输入了 `npm update`，则已安装的版本会被更新，并且 `package-lock.json` 文件会被新版本填充。

    `package.json` 则保持不变。

    若要发觉软件包的新版本，则运行 `npm outdated`。

    若要将所有软件包更新到新的主版本，则全局地安装 `npm-check-updates` 软件包

    ```powershell
    npm install -g npm-check-updates
    ```

    然后运行：

    ```powershell
    ncu -u
    ```

    以升级 `package.json` 文件的 `dependencies` 和 `devDependencies` 中的所有版本，以便 npm 可以安装新的主版本

3. **卸载**：**npm uninstall**

    - 本地卸载：

        ```powershell
        npm uninstall <package-name>
        ```

        如果使用 `-S` 或 `--save` 标志，则此操作还会移除 `package.json` 文件中的引用

        如果程序包是开发依赖项（列出在 `package.json` 文件的 devDependencies 中），则必须使用 `-D` 或 `--save-dev` 标志从文件中移除

        ```powershell
        npm uninstall -S <package-name>
        npm uninstall -D <package-name>
        ```

        

    - 全局卸载：

        ```powershell
        npm uninstall -g <package-name>
        ```

        

4. **运行**：**npm run**

    package.json 文件支持一种用于指定命令行任务（可通过使用以下方式运行）的格式

    ```powershell
    npm run <task-name>
    ```

    ```json
    {
      "scripts": {
        "watch": "webpack --watch --progress --colors --config webpack.conf.js",
        "dev": "webpack --progress --colors --config webpack.conf.js",
        "prod": "NODE_ENV=production webpack -p --config webpack.conf.js",
      },
    }
    ```

    ```powershell
    $ npm run watch
    $ npm run dev
    $ npm run prod
    ```

5. **使用或执行 npm 安装的软件包**

- require导入

    ```powershell
    npm install lodash
    ```

    ```javascript
    const _ = require('lodash')
    ```

- 如果软件包是可执行文件: 

    可执行文件放到 `node_modules/.bin/` 文件夹, 使用`npx`来运行它

6. **npm 依赖与开发依赖**
    - 当使用 `npm install <package-name>` 安装 npm 软件包时，是将其安装为依赖项。该软件包会被自动地列出在 package.json 文件中的 `dependencies` 列表下（在 npm 5 之前：必须手动指定 `--save`）
    - 当添加了 `-D` 或 `--save-dev` 标志时，则会将其安装为开发依赖项（会被添加到 `devDependencies` 列表）。开发依赖是仅用于开发的程序包，在生产环境中并不需要。 例如测试的软件包、webpack 或 Babel。
    - 当投入生产环境时，如果输入 `npm install` 且该文件夹包含 `package.json` 文件时，则会安装它们，因为 npm 会假定这是开发部署。需要设置 `--production` 标志（`npm install --production`），以避免安装这些开发依赖项。

---

## package.json

`package.json` 文件是项目的清单，用于工具的配置中心， `npm` 和 `yarn` 存储所有已安装软件包的名称和版本的地方

示例：

```json
{
  "name": "test-project", 
  "version": "1.0.0", 
  "description": "A Vue.js project", 
  "main": "src/main.js",
  "private": true,
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "unit": "jest --config test/unit/jest.conf.js --coverage",
    "test": "npm run unit",
    "lint": "eslint --ext .js,.vue src test/unit",
    "build": "node build/build.js"
  },
  "dependencies": {
    "vue": "^2.5.2"
  },
  "devDependencies": {
    "autoprefixer": "^7.1.2",
    "babel-core": "^6.22.1",
    "babel-eslint": "^8.2.1",
    "babel-helper-vue-jsx-merge-props": "^2.0.3",
    "babel-jest": "^21.0.2",
    "babel-loader": "^7.1.1",
    "babel-plugin-dynamic-import-node": "^1.2.0",
    "babel-plugin-syntax-jsx": "^6.18.0",
    "babel-plugin-transform-es2015-modules-commonjs": "^6.26.0",
    "babel-plugin-transform-runtime": "^6.22.0",
    "babel-plugin-transform-vue-jsx": "^3.5.0",
    "babel-preset-env": "^1.3.2",
    "babel-preset-stage-2": "^6.22.0",
    "chalk": "^2.0.1",
    "copy-webpack-plugin": "^4.0.1",
    "css-loader": "^0.28.0",
    "eslint": "^4.15.0",
    "eslint-config-airbnb-base": "^11.3.0",
    "eslint-friendly-formatter": "^3.0.0",
    "eslint-import-resolver-webpack": "^0.8.3",
    "eslint-loader": "^1.7.1",
    "eslint-plugin-import": "^2.7.0",
    "eslint-plugin-vue": "^4.0.0",
    "extract-text-webpack-plugin": "^3.0.0",
    "file-loader": "^1.1.4",
    "friendly-errors-webpack-plugin": "^1.6.1",
    "html-webpack-plugin": "^2.30.1",
    "jest": "^22.0.4",
    "jest-serializer-vue": "^0.3.0",
    "node-notifier": "^5.1.2",
    "optimize-css-assets-webpack-plugin": "^3.2.0",
    "ora": "^1.2.0",
    "portfinder": "^1.0.13",
    "postcss-import": "^11.0.0",
    "postcss-loader": "^2.0.8",
    "postcss-url": "^7.2.1",
    "rimraf": "^2.6.0",
    "semver": "^5.3.0",
    "shelljs": "^0.7.6",
    "uglifyjs-webpack-plugin": "^1.1.1",
    "url-loader": "^0.5.8",
    "vue-jest": "^1.0.2",
    "vue-loader": "^13.3.0",
    "vue-style-loader": "^3.0.1",
    "vue-template-compiler": "^2.5.2",
    "webpack": "^3.6.0",
    "webpack-bundle-analyzer": "^2.9.0",
    "webpack-dev-server": "^2.9.1",
    "webpack-merge": "^4.1.0"
  },
  "engines": {
    "node": ">= 6.0.0",
    "npm": ">= 3.0.0"
  },
  "browserslist": ["> 1%", "last 2 versions", "not ie <= 8"]
}
```

1. ### name:

    设置软件包的名称，少于214个字符，不含空格，只能含字母，连字符`-`，下划线`__`

    ```json
    {
        "name": "nodejs_cn"
    }
    ```

2. ### author

    列出软件包的作者名称

    ```json
    {
      "author": "NodeJS中文网 <mail@nodejs.cn> (http://nodejs.cn)"
    }
    // or
    {
      "author": {
        "name": "NodeJS中文网",
        "email": "mail@nodejs.cn",
        "url": "http://nodejs.cn"
      }
    }
    ```

3. ### contributors

    除作者外的项目贡献者，数组

    ```json
    {
      "contributors": ["NodeJS中文网 <mail@nodejs.cn> (http://nodejs.cn))"]
    }
    // or
    {
      "contributors": [
        {
          "name": "NodeJS中文网",
          "email": "mail@nodejs.cn",
          "url": "http://nodejs.cn"
        }
      ]
    }
    ```

4. ### bugs

    链接到软件包的问题跟踪器，最常用的是 GitHub 的 issues 页面

    ```json
    {
      "bugs": "https://github.com/nodejscn/node-api-cn/issues"
    }
    ```

5. ### homepage

    设置软件包的主页

    ```json
    {
      "homepage": "http://nodejs.cn"
    }
    ```

6. ### version

    指定软件包的当前版本，第一个数字是主版本号，第二个数字是次版本号，第三个数字是补丁版本号。

    ```json
     "version": "1.0.0"
    ```

7. ### license

    指定软件包的许可证

    ```json
    "license": "MIT"
    ```

8. ### keywords

    与软件包功能相关的关键字数组

    ```json
    "keywords": [
      "email",
      "machine learning",
      "ai"
    ]
    ```

9. ### description

    对软件包的简短描述

    ```json
    "description": "NodeJS中文网入门教程"
    ```

10. ### repository

    指定了程序包仓库所在的位置

    ```json
    "repository": "github:nodejscn/node-api-cn",
    ```

    显式地设置版本控制系统:

    ```json
    "repository": {
      "type": "git",
      "url": "https://github.com/nodejscn/node-api-cn.git"
    }
    ```

11. ### main

    软件包的入口点

    ```json
    "main": "src/main.js"
    ```

12. ### private

    布尔值，如果设置为 `true`，则可以防止应用程序/软件包被意外发布到 `npm` 上

13. ### scripts

    定义一组可以运行的 node 脚本：

    这些脚本是命令行应用程序。 可以通过调用 `npm run XXXX` 或 `yarn XXXX` 来运行它们，其中 `XXXX` 是命令的名称。 例如：`npm run dev`。

    可以为命令使用任何的名称，脚本也可以是任何操作

    ```json
    "scripts": {
      "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
      "start": "npm run dev",
      "unit": "jest --config test/unit/jest.conf.js --coverage",
      "test": "npm run unit",
      "lint": "eslint --ext .js,.vue src test/unit",
      "build": "node build/build.js"
    }
    ```

14. ### dependencies

    作为依赖安装的 `npm` 软件包的列表

    使用 npm 或 yarn 安装软件包时,该软件包会被自动地插入此列表中

    ```json
    "dependencies": {
      "vue": "^2.5.2"
    }
    ```

15. ### devDependencies

    作为开发依赖安装的 `npm` 软件包的列表

    只需安装在开发机器上，而无需在生产环境中运行代码

    ```shell
    npm install --save-dev <PACKAGENAME>
    yarn add --dev <PACKAGENAME>
    ```

    ```json
    "devDependencies": {
      "autoprefixer": "^7.1.2",
      "babel-core": "^6.22.1"
    }
    ```

16. ### engines

    要运行的 Node.js 或其他命令的版本

    ```json
    "engines": {
      "node": ">= 6.0.0",
      "npm": ">= 3.0.0",
      "yarn": "^0.13.0"
    }
    ```

17. ### browserslist

    支持哪些浏览器（及其版本）。 Babel、Autoprefixer 和其他工具会用到它，以将所需的 polyfill 和 fallback 添加到目标浏览器

    ```json
    "browserslist": [
      "> 1%",
      "last 2 versions",
      "not ie <= 8"
    ]
    ```

    需要支持使用率超过 1％（来自 [CanIUse.com](https://caniuse.com/) 的统计信息）的所有浏览器的最新的 2 个主版本，但不含 IE8 及更低的版本

`package.json` 文件还可以承载命令特有的配置，例如 Babel、ESLint 等

---

## package-lock.json

跟踪被安装的每个软件包的确切版本，以便产品可以以相同的方式被 100％ 复制（即使软件包的维护者更新了软件包）

`package-lock.json` 会固化当前安装的每个软件包的版本，当运行 `npm install`时，`npm` 会使用这些确切的版本

`package-lock.json` 文件需要被提交到 Git 仓库，以便被其他人获取（如果项目是公开的或有合作者，或者将 Git 作为部署源）

运行 `npm update` 时，`package-lock.json` 文件中的依赖的版本会被更新

```json
{
  "requires": true, 
  "lockfileVersion": 1,
  "dependencies": {
    "ansi-regex": {
      "version": "3.0.0",
      "resolved": "https://registry.npmjs.org/ansi-regex/-/ansi-regex-3.0.0.tgz",
      "integrity": "sha1-7QMXwyIGT3lGbAKWa922Bas32Zg="
    },
    "cowsay": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/cowsay/-/cowsay-1.4.0.tgz",
      "integrity": "sha512-rdg5k5PsHFVJheO/pmE3aDg2rUDDTfPJau6yYkZYlHFktUz+UxbE+IgnUAEyyCyv4noL5ltxXD0gZzmHPCy/9g==",
      "requires": {
        "get-stdin": "^5.0.1",
        "optimist": "~0.6.1",
        "string-width": "~2.1.1",
        "strip-eof": "^1.0.0"
      }
    },
    "get-stdin": {
      "version": "5.0.1",
      "resolved": "https://registry.npmjs.org/get-stdin/-/get-stdin-5.0.1.tgz",
      "integrity": "sha1-Ei4WFZHiH/TFJTAwVpPyDmOTo5g="
    },
    "is-fullwidth-code-point": {
      "version": "2.0.0",
      "resolved": "https://registry.npmjs.org/is-fullwidth-code-point/-/is-fullwidth-code-point-2.0.0.tgz",
      "integrity": "sha1-o7MKXE8ZkYMWeqq5O+764937ZU8="
    },
    "minimist": {
      "version": "0.0.10",
      "resolved": "https://registry.npmjs.org/minimist/-/minimist-0.0.10.tgz",
      "integrity": "sha1-3j+YVD2/lggr5IrRoMfNqDYwHc8="
    },
    "optimist": {
      "version": "0.6.1",
      "resolved": "https://registry.npmjs.org/optimist/-/optimist-0.6.1.tgz",
      "integrity": "sha1-2j6nRob6IaGaERwybpDrFaAZZoY=",
      "requires": {
        "minimist": "~0.0.1",
        "wordwrap": "~0.0.2"
      }
    },
    "progress": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/progress/-/progress-2.0.3.tgz",
      "integrity": "sha512-7PiHtLll5LdnKIMw100I+8xJXR5gW2QwWYkT6iJva0bXitZKa/XMrSbdmg3r2Xnaidz9Qumd0VPaMrZlF9V9sA=="
    },
    "string-width": {
      "version": "2.1.1",
      "resolved": "https://registry.npmjs.org/string-width/-/string-width-2.1.1.tgz",
      "integrity": "sha512-nOqH59deCq9SRHlxq1Aw85Jnt4w6KvLKqWVik6oA9ZklXLNIOlqg4F2yrT1MVaTjAqvVwdfeZ7w7aCvJD7ugkw==",
      "requires": {
        "is-fullwidth-code-point": "^2.0.0",
        "strip-ansi": "^4.0.0"
      }
    },
    "strip-ansi": {
      "version": "4.0.0",
      "resolved": "https://registry.npmjs.org/strip-ansi/-/strip-ansi-4.0.0.tgz",
      "integrity": "sha1-qEeQIusaw2iocTibY1JixQXuNo8=",
      "requires": {
        "ansi-regex": "^3.0.0"
      }
    },
    "strip-eof": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/strip-eof/-/strip-eof-1.0.0.tgz",
      "integrity": "sha1-u0P/VZim6wXYm1n80SnJgzE2Br8="
    },
    "wordwrap": {
      "version": "0.0.3",
      "resolved": "https://registry.npmjs.org/wordwrap/-/wordwrap-0.0.3.tgz",
      "integrity": "sha1-o9XabNXAvAAI03I0u68b7WMFkQc="
    }
  }
}
```

 `version` 字段、

指向软件包位置的 `resolved` 字段、

以及用于校验软件包的 `integrity` 字符串

---

## npx

npx是一个非常强大的命令，从 **npm** 的 5.2 版本（发布于 2017 年 7 月）开始可用

可以运行使用 Node.js 构建并通过 npm 仓库发布的代码

1. 运行 `npx commandname` 会自动地在项目的 `node_modules` 文件夹中找到命令的正确引用，而无需知道确切的路径，也不需要在全局和用户路径中安装软件包

2. 无需先安装命令即可运行命令:

    - 不需要安装任何东西
    - 可以使用 @version 语法运行同一命令的不同版本

3. 使用 `@` 指定版本，并将其与 [`node` npm 软件包](https://www.npmjs.com/package/node) 结合使用

    有助于避免使用 `nvm` 之类的工具或其他 Node.js 版本管理工具

    ```powershell
    npx node@10 -v #v10.18.1
    npx node@12 -v #v12.14.1
    ```

4. 可以运行位于 GitHub gist 中的代码

    ```powershell
    npx https://gist.github.com/zkat/4bc19503fe9e9309e2bfaa2c58074d32
    ```

---

## Node.js 事件循环

Node.js JavaScript 代码运行在单个线程上。 每次只处理一件事

调用堆栈是一个 LIFO 队列（后进先出）

示例：

```javascript
const bar = () => console.log('bar')

const baz = () => console.log('baz')

const foo = () => {
  console.log('foo')
  bar()
  baz()
}

foo()
```

```powershell
foo
bar
baz
```

<img src="D:\证件照\call-stack-first-example.png" style="zoom:150%;" />

每次迭代中的事件循环都会查看调用堆栈中是否有东西并执行它直到调用堆栈为空

![](D:\证件照\execution-order-first-example.png)

**<u>入队函数执行</u>**

```javascript
const bar = () => console.log('bar')

const baz = () => console.log('baz')

const foo = () => {
  console.log('foo')
  setTimeout(bar, 0)
  baz()
}

foo()
```

```powershell
foo
baz
bar
```

<img src="D:\证件照\call-stack-second-example.png" style="zoom:150%;" />

程序中所有函数的执行顺序:

![](D:\证件照\execution-order-second-example.png)

**<u>消息队列</u>**

当调用 setTimeout() 时，浏览器或 Node.js 会启动定时器。 当定时器到期时（在此示例中会立即到期，因为将超时值设为 0），则回调函数会被放入“消息队列”中。

消息队列中，用户触发的事件（如单击或键盘事件、或获取响应）也会在此排队，然后代码才有机会对其作出反应。 类似 `onLoad` 这样的 DOM 事件也如此。

事件循环会赋予调用堆栈优先级，它首先处理在调用堆栈中找到的所有东西，一旦其中没有任何东西，便开始处理消息队列中的东西

**<u>作业队列</u>**

ECMAScript 2015 引入了作业队列的概念，Promise 使用了该队列，会尽快地执行异步函数的结果，而不是放在调用堆栈的末尾

在当前函数结束之前 resolve 的 Promise 会在当前函数之后被立即执行

```javascript
const bar = () => console.log('bar')

const baz = () => console.log('baz')

const foo = () => {
  console.log('foo')
  setTimeout(bar, 0)
  new Promise((resolve, reject) =>
    resolve('应该在 baz 之后、bar 之前')
  ).then(resolve => console.log(resolve))
  baz()
}

foo()
```

```powershell
foo
baz
应该在 baz 之后、bar 之前
bar
```

---

## process.nextTick()

每当事件循环进行一次完整的行程时，我们都将其称为一个滴答

将一个函数传给 `process.nextTick()` 时，则指示引擎在当前操作结束（在<u>下一个事件循环滴答开始之前</u>）时调用此函数：

```javascript
process.nextTick(() => {
  // doSomething
})
```

当要确保在下一个事件循环迭代中代码已被执行，则使用 `nextTick()`

---

## setImmediate()

异步地（但要尽可能快）执行某些代码时使用

```javascript
setImmediate(() => {
  // doSomething
})
```

作为 setImmediate() 参数传入的任何函数都是在事件循环的<u>下一个迭代中执行</u>的回调。

***`setImmediate()` 与 `setTimeout(() => {}, 0)`（传入 0 毫秒的超时）、`process.nextTick()` 有何不同？***

- 传给 `process.nextTick()` 的函数会在事件循环的**当前迭代**中（当前操作结束之后）被执行, 始终在 `setTimeout` 和 `setImmediate` 之前执行。
- 延迟 0 毫秒的 `setTimeout()` 回调与 `setImmediate()` 非常相似。 执行顺序取决于各种因素，但是它们都会在事件循环的**下一个迭代**中运行

---

## 定时器

`setTimeout` 和 `setInterval` 可通过[定时器模块](http://nodejs.cn/api/timers.html)在 Node.js 中使用。

Node.js 还提供 `setImmediate()`（相当于使用 `setTimeout(() => {}, 0)`），通常用于与 Node.js 事件循环配合使用。

---

## JavaScript 异步编程与回调

**<u>回调</u>**

你不知道用户何时单击按钮。 因此，为点击事件定义了一个事件处理程序。 该事件处理程序会接受一个函数，该函数会在该事件被触发时被调用：

```javascript
document.getElementById('button').addEventListener('click', () => {
  //被点击
})
```

这就是所谓的回调。

回调是一个简单的函数，会作为值被传给另一个函数，并且仅在事件发生时才被执行。 之所以这样做，是因为 JavaScript 具有顶级的函数，这些函数可以被分配给变量并传给其他函数（称为高阶函数）。

---

## Node.js 事件触发器

在后端，Node.js 也提供了使用 [`events` 模块](http://nodejs.cn/api/events.html)构建类似系统的选项。

具体上，此模块提供了 `EventEmitter` 类，用于处理事件。

初始化：

```javascript
const EventEmitter = require('events')
const eventEmitter = new EventEmitter()
```

该对象公开了 `on` 和 `emit` 方法。

- `emit` 用于触发事件。
- `on` 用于添加回调函数（会在事件被触发时执行）。

```java
eventEmitter.on('start', () => {
  console.log('开始')
})
eventEmitter.emit('start')
```

- `once()`: 添加单次监听器。
- `removeListener()` / `off()`: 从事件中移除事件监听器。
- `removeAllListeners()`: 移除事件的所有监听器。

---

## 搭建 HTTP 服务器

引入 [`http` 模块](http://nodejs.cn/api/http.html)。使用该模块来创建 HTTP 服务器

```javascript
const http = require('http')

const hostname = '127.0.0.1'
const port = 3000

// 使用 http.createServer() 初始化 HTTP 服务器时，
// 服务器会在获得所有 HTTP 请求头（而不是请求正文时）时调用回调
const server = http.createServer((req, res) => {
    res.statusCode = 200
    res.setHeader('Content-Type', 'text/plain')
    res.end('你好世界\n')
})

server.listen(port, () => {
    console.log(`服务器运行在 http://${hostname}:${port}/`)
})
```

**<u>发送 HTTP 请求</u>**

1. GET

    ```javascript
    const https = require('https')
    const options = {
      hostname: 'nodejs.cn',
      port: 443,
      path: '/todos',
      method: 'GET'
    }
    
    const req = https.request(options, res => {
      console.log(`状态码: ${res.statusCode}`)
    
      res.on('data', d => {
        process.stdout.write(d)
      })
    })
    
    req.on('error', error => {
      console.error(error)
    })
    
    req.end()
    ```

2. POST

    ```javascript
    const https = require('https')
    
    const data = JSON.stringify({
      todo: '做点事情'
    })
    
    const options = {
      hostname: 'nodejs.cn',
      port: 443,
      path: '/todos',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Length': data.length
      }
    }
    
    const req = https.request(options, res => {
      console.log(`状态码: ${res.statusCode}`)
    
      res.on('data', d => {
        process.stdout.write(d)
      })
    })
    
    req.on('error', error => {
      console.error(error)
    })
    
    req.write(data)
    req.end()
    ```

    PUT 和 DELETE 请求使用相同的 POST 请求格式，只需更改 `options.method` 的值即可。

---

## axios

使用 Node.js 执行 HTTP 请求的最简单的方式是使用 [Axios 库](https://github.com/axios/axios)：

```javascript
const axios = require('axios')

axios
  .post('http://nodejs.cn/todos', {
    todo: '做点事情'
  })
  .then(res => {
    console.log(`状态码: ${res.statusCode}`)
    console.log(res)
  })
  .catch(error => {
    console.error(error)
  })
```

---

## 获取 HTTP 请求数据

如果使用的是 Express，则非常简单：使用 `body-parser` Node.js 模块

```javascript
// 发送请求
const axios = require('axios')

axios.post('http://nodejs.cn/todos', {
  todo: '做点事情'
})
```

```javascript
// 对应的服务器端代码
const express = require('express')
const app = express()

app.use(
  express.urlencoded({
    extended: true
  })
)

app.use(express.json())

app.post('/todos', (req, res) => {
  console.log(req.body.todo)
})
```

---

## 文件描述符

与文件系统中的文件进行交互之前，需要先获取文件的描述符，使用 `fs` 模块提供的 `open()` 方法打开文件后返回

```javascript
const fs = require('fs')

fs.open('/Users/joe/test.txt', 'r', (err, fd) => {
  //fd 是文件描述符。
})
```

注意，将 `r` 作为 `fs.open()` 调用的第二个参数，其他常用的标志：

- `r+` 打开文件用于读写。
- `w+` 打开文件用于读写，将流定位到文件的开头。如果文件不存在则创建文件。
- `a` 打开文件用于写入，将流定位到文件的末尾。如果文件不存在则创建文件。
- `a+` 打开文件用于读写，将流定位到文件的末尾。如果文件不存在则创建文件。

也可以使用 `fs.openSync` 方法打开文件，该方法会返回文件描述符（而不是在回调中提供）：

```javascript
const fs = require('fs')

try {
  const fd = fs.openSync('/Users/joe/test.txt', 'r')
} catch (err) {
  console.error(err)
}
```

---

## 文件属性

`fs` 模块提供的 `stat()` 方法: 调用时传入文件的路径，一旦 Node.js 获得文件的详细信息，则会调用传入的回调函数，并带上两个参数：错误消息和文件属性

提取文件属性信息：

- 使用 `stats.isFile()` 和 `stats.isDirectory()` 判断文件是否目录或文件。
- 使用 `stats.isSymbolicLink()` 判断文件是否符号链接。
- 使用 `stats.size` 获取文件的大小（以字节为单位）。

```javascript
const fs = require('fs')
fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err)
    return
  }
  //可以访问 `stats` 中的文件属性
  stats.isFile() //true
  stats.isDirectory() //false
  stats.isSymbolicLink() //false
  stats.size //1024000 //= 1MB
})
```

---

## 文件路径

```javascript
const path = require('path')
```

给定一个路径，可以使用以下方法从其中提取信息：

- `dirname`: 获取文件的父文件夹。
- `basename`: 获取文件名部分。
- `extname`: 获取文件的扩展名。

```javascript
const notes = '/users/joe/notes.txt'

path.dirname(notes) // /users/joe
path.basename(notes) // notes.txt
// 通过为 basename 指定第二个参数来获取不带扩展名的文件名
path.basename(notes, path.extname(notes)) //notes
path.extname(notes) // .txt
```

`path.join()` 连接路径的两个或多个片段:

```javascript
const name = 'joe'
path.join('/', 'users', name, 'notes.txt') //'/users/joe/notes.txt'
```

 `path.resolve()` 获得相对路径的绝对路径计算:

```javascript
path.resolve('joe.txt') //'/Users/joe/joe.txt' 如果从主文件夹运行。
// 指定第二个文件夹参数，则 resolve 会使用第一个作为第二个的基础：
path.resolve('tmp', 'joe.txt') //'/Users/joe/tmp/joe.txt' 如果从主文件夹运行
// 第一个参数以斜杠开头，则表示它是绝对路径
path.resolve('/etc', 'joe.txt') //'/etc/joe.txt'
```

`path.normalize()` 是另一个有用的函数，当包含诸如 `.`、`..` 或双斜杠之类的相对说明符时，其会尝试计算实际的路径：

```javascript
path.normalize('/users/joe/..//test.txt') //'/users/test.txt'
```

---

## 读取文件

`fs.readFile()` 方法

参数：<u>文件路径</u>、<u>编码方式</u>、以及会带上文件数据（以及错误）进行调用的<u>回调函数</u>

```javascript
const fs = require('fs')

fs.readFile('/Users/joe/test.txt', 'utf8' , (err, data) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(data)
})
```

同步的版本 `fs.readFileSync()`:

```javascript
const fs = require('fs')

try {
  const data = fs.readFileSync('/Users/joe/test.txt', 'utf8')
  console.log(data)
} catch (err) {
  console.error(err)
}
```

`fs.readFile()` 和 `fs.readFileSync()` 都会在返回数据之前将文件的全部内容读取到内存中，这意味着大文件会对内存的消耗和程序执行的速度产生重大的影响。这种情况下，更好的选择是使用流来读取文件的内容

---

## 写入文件

`fs.writeFile()`

```javascript
const fs = require('fs')
const content = 'some content'
fs.writeFile('/Users/joe/test.txt', content, err => {
    if (err) {
        console.error(err)
        return
    }
    // file write success
})
```

同步的版本 `fs.writeFileSync()`

```javascript
const fs = require('fs')

const content = '一些内容'

try {
  const data = fs.writeFileSync('/Users/joe/test.txt', content)
  //文件写入成功。
} catch (err) {
  console.error(err)
}
```

默认情况下，`fs.writeFile()` 会替换文件的内容（如果文件已经存在）。通过指定标志来修改默认的行为：

```javascript
fs.writeFile('/Users/joe/test.txt', content, { flag: 'a+' }, err => {})
```

- `r+` 打开文件用于读写。
- `w+` 打开文件用于读写，将流定位到文件的开头。如果文件不存在则创建文件。
- `a` 打开文件用于写入，将流定位到文件的末尾。如果文件不存在则创建文件。
- `a+` 打开文件用于读写，将流定位到文件的末尾。如果文件不存在则创建文件。

`fs.appendFile()`（及其对应的 `fs.appendFileSync()`）: 内容追加到文件末尾

```javascript
const content = '一些内容'

fs.appendFile('file.log', content, err => {
  if (err) {
    console.error(err)
    return
  }
  //完成！
})
```

***使用流***

所有这些方法都是在将全部内容写入文件之后才会将控制权返回给程序（在异步的版本中，这意味着执行回调）。

在这种情况下，更好的选择是使用流写入文件的内容。

---

## 使用文件夹

Node.js 的 `fs` 核心模块提供了许多便捷的方法用于处理文件夹。

`fs.access()` 检查文件夹是否存在以及 Node.js 是否具有访问权限

`fs.mkdir()` 或 `fs.mkdirSync()` 可以创建新的文件夹

```javascript
const fs = require('fs')

const folderName = '/Users/joe/test'

try {
  if (!fs.existsSync(folderName)) {
    fs.mkdirSync(folderName)
  }
} catch (err) {
  console.error(err)
}
```

`fs.readdir()` 或 `fs.readdirSync()` 可以读取目录的内容

```javascript
// 读取文件夹的内容（全部的文件和子文件夹），并返回它们的相对路径
const fs = require('fs')
const path = require('path')

const folderPath = '/Users/joe'

fs.readdirSync(folderPath)

// 获取完整的路径
fs.readdirSync(folderPath).map(fileName => {
  return path.join(folderPath, fileName)
})

// 过滤结果以仅返回文件（排除文件夹）
const isFile = fileName => {
  return fs.lstatSync(fileName).isFile()
}

fs.readdirSync(folderPath).map(fileName => {
  return path.join(folderPath, fileName)
})
.filter(isFile)
```

`fs.rename()` 或 `fs.renameSync()` 可以重命名文件夹。 第一个参数是当前的路径，第二个参数是新的路径：

```javascript
const fs = require('fs')

fs.rename('/Users/joe', '/Users/roger', err => {
  if (err) {
    console.error(err)
    return
  }
  //完成
})
```

`fs.rmdir()` 或 `fs.rmdirSync()` 可以删除文件夹

最好安装 [`fs-extra`](https://www.npmjs.com/package/fs-extra) 模块，该模块非常受欢迎且维护良好。 它是 `fs` 模块的直接替代品:

安装：npm install fs-extra

使用：

```javascript
const fs = require('fs-extra')
const folder = '/Users/joe'

fs.remove(folder, err => {
  console.error(err)
})

// 可以与 promise 一起使用
fs.remove(folder)
  .then(() => {
    //完成
  })
  .catch(err => {
    console.error(err)
  })

//  async/await
async function removeFolder(folder) {
  try {
    await fs.remove(folder)
    //完成
  } catch (err) {
    console.error(err)
  }
}

const folder = '/Users/joe'
removeFolder(folder)
```

---

## 文件系统模块

```javascript
const fs = require('fs')
```

- `fs.access()`: 检查文件是否存在，以及 Node.js 是否有权限访问。
- `fs.appendFile()`: 追加数据到文件。如果文件不存在，则创建文件。
- `fs.chmod()`: 更改文件（通过传入的文件名指定）的权限。相关方法：`fs.lchmod()`、`fs.fchmod()`。
- `fs.chown()`: 更改文件（通过传入的文件名指定）的所有者和群组。相关方法：`fs.fchown()`、`fs.lchown()`。
- `fs.close()`: 关闭文件描述符。
- `fs.copyFile()`: 拷贝文件。
- `fs.createReadStream()`: 创建可读的文件流。
- `fs.createWriteStream()`: 创建可写的文件流。
- `fs.link()`: 新建指向文件的硬链接。
- `fs.mkdir()`: 新建文件夹。
- `fs.mkdtemp()`: 创建临时目录。
- `fs.open()`: 设置文件模式。
- `fs.readdir()`: 读取目录的内容。
- `fs.readFile()`: 读取文件的内容。相关方法：`fs.read()`。
- `fs.readlink()`: 读取符号链接的值。
- `fs.realpath()`: 将相对的文件路径指针（`.`、`..`）解析为完整的路径。
- `fs.rename()`: 重命名文件或文件夹。
- `fs.rmdir()`: 删除文件夹。
- `fs.stat()`: 返回文件（通过传入的文件名指定）的状态。相关方法：`fs.fstat()`、`fs.lstat()`。
- `fs.symlink()`: 新建文件的符号链接。
- `fs.truncate()`: 将传递的文件名标识的文件截断为指定的长度。相关方法：`fs.ftruncate()`。
- `fs.unlink()`: 删除文件或符号链接。
- `fs.unwatchFile()`: 停止监视文件上的更改。
- `fs.utimes()`: 更改文件（通过传入的文件名指定）的时间戳。相关方法：`fs.futimes()`。
- `fs.watchFile()`: 开始监视文件上的更改。相关方法：`fs.watch()`。
- `fs.writeFile()`: 将数据写入文件。相关方法：`fs.write()`。

 异步的 API 会与回调一起使用：

```javascript
const fs = require('fs')

fs.rename('before.json', 'after.json', err => {
  if (err) {
    return console.error(err)
  }

  //完成
})
```

同步的 API 则可以这样使用，并使用 try/catch 块来处理错误

```javascript
const fs = require('fs')
// 同步的api脚本的执行会阻塞，直到文件操作成功
try {
  fs.renameSync('before.json', 'after.json')
  //完成
} catch (err) {
  console.error(err)
}
```

---

## 路径模块

```javascript
const path = require('path')
```

### `path.sep`

（作为路径段分隔符，在 Windows 上是 `\`，在 Linux/macOS 上是 `/`）和 `path.delimiter`（作为路径定界符，在 Windows 上是 `;`，在 Linux/macOS 上是 `:`）

### `path.basename()`

返回路径的最后一部分。 第二个参数可以过滤掉文件的扩展名：

```javascript
require('path').basename('/test/something') //something
require('path').basename('/test/something.txt') //something.txt
require('path').basename('/test/something.txt', '.txt') //something
```

###  `path.dirname()`

返回路径的目录部分：

```javascript
require('path').dirname('/test/something') // /test
require('path').dirname('/test/something/file.txt') // /test/something
```

### `path.extname()`

返回路径的扩展名部分。

```javascript
require('path').extname('/test/something') // ''
require('path').extname('/test/something/file.txt') // '.txt'
```

### `path.isAbsolute()`

如果是绝对路径，则返回 true。

```javascript
require('path').isAbsolute('/test/something') // true
require('path').isAbsolute('./test/something') // false
```

### `path.join()`

连接路径的两个或多个部分：

```javascript
const name = 'joe'
require('path').join('/', 'users', name, 'notes.txt') //'/users/joe/notes.txt'
```

### `path.normalize()`

当包含类似 `.`、`..` 或双斜杠等相对的说明符时，则尝试计算实际的路径：

```javascript
require('path').normalize('/users/joe/..//test.txt') //'/users/test.txt'
```

### `path.parse()`

解析对象的路径为组成其的片段：

- `root`: 根路径。
- `dir`: 从根路径开始的文件夹路径。
- `base`: 文件名 + 扩展名
- `name`: 文件名
- `ext`: 文件扩展名

例如：

```javascript
require('path').parse('/users/test.txt')
```

结果是：

```javascript
{
  root: '/',
  dir: '/users',
  base: 'test.txt',
  ext: '.txt',
  name: 'test'
}
```

### `path.relative()`

接受 2 个路径作为参数。 基于当前工作目录，返回从第一个路径到第二个路径的相对路径。

例如：

```javascript
require('path').relative('/Users/joe', '/Users/joe/test.txt') //'test.txt'
require('path').relative('/Users/joe', '/Users/joe/something/test.txt') //'something/test.txt'
```

### `path.resolve()`

可以使用 `path.resolve()` 获得相对路径的绝对路径计算：

```javascript
path.resolve('joe.txt') //'/Users/joe/joe.txt' 如果从主文件夹运行
```

通过指定第二个参数，`resolve` 会使用第一个参数作为第二个参数的基准：

```javascript
path.resolve('tmp', 'joe.txt') //'/Users/joe/tmp/joe.txt' 如果从主文件夹运行
```

如果第一个参数以斜杠开头，则表示它是绝对路径：

```javascript
path.resolve('/etc', 'joe.txt') //'/etc/joe.txt'
```

---

## 操作系统模块

```javascript
const os = require('os')
```

### `os.arch()`

返回标识底层架构的字符串，例如 `arm`、`x64`、`arm64`。

### `os.cpus()`

返回关于系统上可用的 CPU 的信息。

例如：

```javascript
[
  {
    model: 'Intel(R) Core(TM)2 Duo CPU     P8600  @ 2.40GHz',
    speed: 2400,
    times: {
      user: 281685380,
      nice: 0,
      sys: 187986530,
      idle: 685833750,
      irq: 0
    }
  },
  {
    model: 'Intel(R) Core(TM)2 Duo CPU     P8600  @ 2.40GHz',
    speed: 2400,
    times: {
      user: 282348700,
      nice: 0,
      sys: 161800480,
      idle: 703509470,
      irq: 0
    }
  }
]
```

### `os.endianness()`

根据是使用[大端序或小端序](https://en.wikipedia.org/wiki/Endianness)编译 Node.js，返回 `BE` 或 `LE`。

### `os.freemem()`

返回代表系统中可用内存的字节数。

### `os.homedir()`

返回到当前用户的主目录的路径。

例如：

```javascript
'/Users/joe'
```

### `os.hostname()`

返回主机名。

### `os.loadavg()`

返回操作系统对平均负载的计算。

这仅在 Linux 和 macOS 上返回有意义的值。

例如：

```javascript
[3.68798828125, 4.00244140625, 11.1181640625]
```

### `os.networkInterfaces()`

返回系统上可用的网络接口的详细信息。

例如：

```javascript
{ lo0:
   [ { address: '127.0.0.1',
       netmask: '255.0.0.0',
       family: 'IPv4',
       mac: 'fe:82:00:00:00:00',
       internal: true },
     { address: '::1',
       netmask: 'ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff',
       family: 'IPv6',
       mac: 'fe:82:00:00:00:00',
       scopeid: 0,
       internal: true },
     { address: 'fe80::1',
       netmask: 'ffff:ffff:ffff:ffff::',
       family: 'IPv6',
       mac: 'fe:82:00:00:00:00',
       scopeid: 1,
       internal: true } ],
  en1:
   [ { address: 'fe82::9b:8282:d7e6:496e',
       netmask: 'ffff:ffff:ffff:ffff::',
       family: 'IPv6',
       mac: '06:00:00:02:0e:00',
       scopeid: 5,
       internal: false },
     { address: '192.168.1.38',
       netmask: '255.255.255.0',
       family: 'IPv4',
       mac: '06:00:00:02:0e:00',
       internal: false } ],
  utun0:
   [ { address: 'fe80::2513:72bc:f405:61d0',
       netmask: 'ffff:ffff:ffff:ffff::',
       family: 'IPv6',
       mac: 'fe:80:00:20:00:00',
       scopeid: 8,
       internal: false } ] }
```

### `os.platform()`

返回为 Node.js 编译的平台：

- `darwin`
- `freebsd`
- `linux`
- `openbsd`
- `win32`
- ...等

### `os.release()`

返回标识操作系统版本号的字符串。

### `os.tmpdir()`

返回指定的临时文件夹的路径。

### `os.totalmem()`

返回表示系统中可用的总内存的字节数。

### `os.type()`

标识操作系统：

- `Linux`
- macOS 上为`Darwin`
- Windows 上为 `Windows_NT`

### `os.uptime()`

返回自上次重新启动以来计算机持续运行的秒数。

### `os.userInfo()`

返回包含当前 `username`、`uid`、`gid`、`shell` 和 `homedir` 的对象。

---

## 事件模块

```javascript
const EventEmitter = require('events')
const door = new EventEmitter()
```

事件监听器返回及使用以下事件：

- 当监听器被添加时返回 `newListener`。
- 当监听器被移除时返回 `removeListener`。

### `emitter.addListener()`

`emitter.on()` 的别名。

### `emitter.emit()`

触发事件。 按照事件被注册的顺序同步地调用每个事件监听器。

```javascript
door.emit("slam") // 触发 "slam" 事件。
```

### `emitter.eventNames()`

返回字符串（表示在当前 `EventEmitter` 对象上注册的事件）数组：

```javascript
door.eventNames()
```

### `emitter.getMaxListeners()`

获取可以添加到 `EventEmitter` 对象的监听器的最大数量（默认为 10，但可以使用 `setMaxListeners()` 进行增加或减少）。

```javascript
door.getMaxListeners()
```

### `emitter.listenerCount()`

获取作为参数传入的事件监听器的计数：

```javascript
door.listenerCount('open')
```

### `emitter.listeners()`

获取作为参数传入的事件监听器的数组：

```javascript
door.listeners('open')
```

### `emitter.off()`

`emitter.removeListener()` 的别名，新增于 Node.js 10。

### `emitter.on()`

添加当事件被触发时调用的回调函数。

用法：

```javascript
door.on('open', () => {
  console.log('打开')
})
```

### `emitter.once()`

添加当事件在注册之后首次被触发时调用的回调函数。 该回调只会被调用一次，不会再被调用。

```javascript
const EventEmitter = require('events')
const ee = new EventEmitter()

ee.once('my-event', () => {
  //只调用一次回调函数。
})
```

### `emitter.prependListener()`

当使用 `on` 或 `addListener` 添加监听器时，监听器会被添加到监听器队列中的最后一个，并且最后一个被调用。 使用 `prependListener` 则可以在其他监听器之前添加并调用。

### `emitter.prependOnceListener()`

当使用 `once` 添加监听器时，监听器会被添加到监听器队列中的最后一个，并且最后一个被调用。 使用 `prependOnceListener` 则可以在其他监听器之前添加并调用。

### `emitter.removeAllListeners()`

移除 `EventEmitter` 对象的所有监听特定事件的监听器：

```javascript
door.removeAllListeners('open')
```

### `emitter.removeListener()`

移除特定的监听器。 可以通过将回调函数保存到变量中（当添加时），以便以后可以引用它：

```javascript
const doSomething = () => {}
door.on('open', doSomething)
door.removeListener('open', doSomething)
```

### `emitter.setMaxListeners()`

设置可以添加到 `EventEmitter` 对象的监听器的最大数量（默认为 10，但可以增加或减少）。

```javascript
door.setMaxListeners(50)
```

---

## http 模块

```javascript
const http = require('http')
```

该模块提供了一些属性、方法、以及类

### 属性

#### `http.METHODS`

此属性列出了所有支持的 HTTP 方法：

```javascript
> require('http').METHODS
[ 'ACL',
  'BIND',
  'CHECKOUT',
  'CONNECT',
  'COPY',
  'DELETE',
  'GET',
  'HEAD',
  'LINK',
  'LOCK',
  'M-SEARCH',
  'MERGE',
  'MKACTIVITY',
  'MKCALENDAR',
  'MKCOL',
  'MOVE',
  'NOTIFY',
  'OPTIONS',
  'PATCH',
  'POST',
  'PROPFIND',
  'PROPPATCH',
  'PURGE',
  'PUT',
  'REBIND',
  'REPORT',
  'SEARCH',
  'SUBSCRIBE',
  'TRACE',
  'UNBIND',
  'UNLINK',
  'UNLOCK',
  'UNSUBSCRIBE' ]
```

#### `http.STATUS_CODES`

此属性列出了所有的 HTTP 状态代码及其描述：

```javascript
> require('http').STATUS_CODES
{ '100': 'Continue',
  '101': 'Switching Protocols',
  '102': 'Processing',
  '200': 'OK',
  '201': 'Created',
  '202': 'Accepted',
  '203': 'Non-Authoritative Information',
  '204': 'No Content',
  '205': 'Reset Content',
  '206': 'Partial Content',
  '207': 'Multi-Status',
  '208': 'Already Reported',
  '226': 'IM Used',
  '300': 'Multiple Choices',
  '301': 'Moved Permanently',
  '302': 'Found',
  '303': 'See Other',
  '304': 'Not Modified',
  '305': 'Use Proxy',
  '307': 'Temporary Redirect',
  '308': 'Permanent Redirect',
  '400': 'Bad Request',
  '401': 'Unauthorized',
  '402': 'Payment Required',
  '403': 'Forbidden',
  '404': 'Not Found',
  '405': 'Method Not Allowed',
  '406': 'Not Acceptable',
  '407': 'Proxy Authentication Required',
  '408': 'Request Timeout',
  '409': 'Conflict',
  '410': 'Gone',
  '411': 'Length Required',
  '412': 'Precondition Failed',
  '413': 'Payload Too Large',
  '414': 'URI Too Long',
  '415': 'Unsupported Media Type',
  '416': 'Range Not Satisfiable',
  '417': 'Expectation Failed',
  '418': 'I\'m a teapot',
  '421': 'Misdirected Request',
  '422': 'Unprocessable Entity',
  '423': 'Locked',
  '424': 'Failed Dependency',
  '425': 'Unordered Collection',
  '426': 'Upgrade Required',
  '428': 'Precondition Required',
  '429': 'Too Many Requests',
  '431': 'Request Header Fields Too Large',
  '451': 'Unavailable For Legal Reasons',
  '500': 'Internal Server Error',
  '501': 'Not Implemented',
  '502': 'Bad Gateway',
  '503': 'Service Unavailable',
  '504': 'Gateway Timeout',
  '505': 'HTTP Version Not Supported',
  '506': 'Variant Also Negotiates',
  '507': 'Insufficient Storage',
  '508': 'Loop Detected',
  '509': 'Bandwidth Limit Exceeded',
  '510': 'Not Extended',
  '511': 'Network Authentication Required' }
```

#### `http.globalAgent`

指向 Agent 对象的全局实例，该实例是 `http.Agent` 类的实例。

用于管理 HTTP 客户端连接的持久性和复用，它是 Node.js HTTP 网络的关键组件。

稍后会在 `http.Agent` 类的说明中提供更多描述。

### 方法

#### `http.createServer()`

返回 `http.Server` 类的新实例。

用法：

```javascript
const server = http.createServer((req, res) => {
  //使用此回调处理每个单独的请求。
})
```

#### `http.request()`

发送 HTTP 请求到服务器，并创建 `http.ClientRequest` 类的实例。

#### `http.get()`

类似于 `http.request()`，但会自动地设置 HTTP 方法为 GET，并自动地调用 `req.end()`。

### 类

HTTP 模块提供了 5 个类：

- `http.Agent`
- `http.ClientRequest`
- `http.Server`
- `http.ServerResponse`
- `http.IncomingMessage`

#### `http.Agent`

Node.js 会创建 `http.Agent` 类的全局实例，以管理 HTTP 客户端连接的持久性和复用，这是 Node.js HTTP 网络的关键组成部分。

该对象会确保对服务器的每个请求进行排队并且单个 socket 被复用。

它还维护一个 socket 池。 出于性能原因，这是关键。

#### `http.ClientRequest`

当 `http.request()` 或 `http.get()` 被调用时，会创建 `http.ClientRequest` 对象。

当响应被接收时，则会使用响应（`http.IncomingMessage` 实例作为参数）来调用 `response` 事件。

返回的响应数据可以通过以下两种方式读取：

- 可以调用 `response.read()` 方法。
- 在 `response` 事件处理函数中，可以为 `data` 事件设置事件监听器，以便可以监听流入的数据。

#### `http.Server`

当使用 `http.createServer()` 创建新的服务器时，通常会实例化并返回此类。

拥有服务器对象后，就可以访问其方法：

- `close()` 停止服务器不再接受新的连接。
- `listen()` 启动 HTTP 服务器并监听连接。

#### `http.ServerResponse`

由 `http.Server` 创建，并作为第二个参数传给它触发的 `request` 事件。

通常在代码中用作 `res`：

```javascript
const server = http.createServer((req, res) => {
  //res 是一个 http.ServerResponse 对象。
})
```

在事件处理函数中总是会调用的方法是 `end()`，它会关闭响应，当消息完成时则服务器可以将其发送给客户端。 必须在每个响应上调用它。

以下这些方法用于与 HTTP 消息头进行交互：

- `getHeaderNames()` 获取已设置的 HTTP 消息头名称的列表。
- `getHeaders()` 获取已设置的 HTTP 消息头的副本。
- `setHeader('headername', value)` 设置 HTTP 消息头的值。
- `getHeader('headername')` 获取已设置的 HTTP 消息头。
- `removeHeader('headername')` 删除已设置的 HTTP 消息头。
- `hasHeader('headername')` 如果响应已设置该消息头，则返回 true。
- `headersSent()` 如果消息头已被发送给客户端，则返回 true。

在处理消息头之后，可以通过调用 `response.writeHead()`（该方法接受 statusCode 作为第一个参数，可选的状态消息和消息头对象）将它们发送给客户端。

若要在响应正文中发送数据给客户端，则使用 `write()`。 它会发送缓冲的数据到 HTTP 响应流。

如果消息头还未被发送，则使用 `response.writeHead()` 会先发送消息头，其中包含在请求中已被设置的状态码和消息，可以通过设置 `statusCode` 和 `statusMessage` 属性的值进行编辑：

```javascript
response.statusCode = 500
response.statusMessage = '内部服务器错误'
```

#### `http.IncomingMessage`

`http.IncomingMessage` 对象可通过以下方式创建：

- `http.Server`，当监听 `request` 事件时。
- `http.ClientRequest`，当监听 `response` 事件时。

它可以用来访问响应：

- 使用 `statusCode` 和 `statusMessage` 方法来访问状态。
- 使用 `headers` 方法或 `rawHeaders` 来访问消息头。
- 使用 `method` 方法来访问 HTTP 方法。
- 使用 `httpVersion` 方法来访问 HTTP 版本。
- 使用 `url` 方法来访问 URL。
- 使用 `socket` 方法来访问底层的 socket。

因为 `http.IncomingMessage` 实现了可读流接口，因此数据可以使用流访问。

---

## Buffer

Buffer 是内存区域，表示在 V8 JavaScript 引擎外部分配的固定大小的内存块（无法调整大小）

### 为什么需要 buffer？

Buffer 被引入用以帮助开发者处理二进制数据，在此生态系统中传统上只处理字符串而不是二进制数据。

Buffer 与流紧密相连。 当流处理器接收数据的速度快于其消化的速度时，则会将数据放入 buffer 中。

一个简单的场景是：当观看 YouTube 视频时，红线超过了观看点：即下载数据的速度比查看数据的速度快，且浏览器会对数据进行缓冲。

### 如何创建 buffer

使用 [`Buffer.from()`](http://nodejs.cn/api/buffer.html#buffer_buffer_from_buffer_alloc_and_buffer_allocunsafe)、[`Buffer.alloc()`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_alloc_size_fill_encoding) 和 [`Buffer.allocUnsafe()`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_allocunsafe_size) 方法可以创建 buffer。

```javascript
const buf = Buffer.from('Hey!')
```

- [`Buffer.from(array)`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_array)
- [`Buffer.from(arrayBuffer[, byteOffset[, length\]])`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_arraybuffer_byteoffset_length)
- [`Buffer.from(buffer)`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_buffer)
- [`Buffer.from(string[, encoding\])`](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_string_encoding)

也可以只初始化 buffer（传入大小）。 以下会创建一个 1KB 的 buffer：

```javascript
const buf = Buffer.alloc(1024)
//或
const buf = Buffer.allocUnsafe(1024)
```

虽然 `alloc` 和 `allocUnsafe` 均分配指定大小的 `Buffer`（以字节为单位），但是 `alloc` 创建的 `Buffer` 会被使用零进行初始化，而 `allocUnsafe` 创建的 `Buffer` 不会被初始化。 这意味着，尽管 `allocUnsafe` 比 `alloc` 要快得多，但是分配的内存片段可能包含可能敏感的旧数据。

当 `Buffer` 内存被读取时，如果内存中存在较旧的数据，则可以被访问或泄漏。 这就是真正使 `allocUnsafe` 不安全的原因，在使用它时必须格外小心。

### 使用 buffer

### 访问 buffer 的内容

Buffer（字节数组）可以像数组一样被访问：

```javascript
const buf = Buffer.from('Hey!')
console.log(buf[0]) //72
console.log(buf[1]) //101
console.log(buf[2]) //121
```

这些数字是 Unicode 码，用于标识 buffer 位置中的字符（H => 72、e => 101、y => 121）。

可以使用 `toString()` 方法打印 buffer 的全部内容：

```javascript
console.log(buf.toString())
```

> 注意，如果使用数字（设置其大小）初始化 buffer，则可以访问到包含随机数据的已预初始化的内存（而不是空的 buffer）！

### 获取 buffer 的长度

使用 `length` 属性：

```javascript
const buf = Buffer.from('Hey!')
console.log(buf.length)
```

### 迭代 buffer 的内容

```javascript
const buf = Buffer.from('Hey!')
for (const item of buf) {
  console.log(item) //72 101 121 33
}
```

### 更改 buffer 的内容

可以使用 `write()` 方法将整个数据字符串写入 buffer：

```javascript
const buf = Buffer.alloc(4)
buf.write('Hey!')
```

就像可以使用数组语法访问 buffer 一样，你也可以使用相同的方式设置 buffer 的内容：

```javascript
const buf = Buffer.from('Hey!')
buf[1] = 111 //o
console.log(buf.toString()) //Hoy!
```

### 复制 buffer

使用 `copy()` 方法可以复制 buffer：

```javascript
const buf = Buffer.from('Hey!')
let bufcopy = Buffer.alloc(4) //分配 4 个字节。
buf.copy(bufcopy)
```

默认情况下，会复制整个 buffer。 另外的 3 个参数可以定义开始位置、结束位置、以及新的 buffer 长度：

```javascript
const buf = Buffer.from('Hey!')
let bufcopy = Buffer.alloc(2) //分配 2 个字节。
buf.copy(bufcopy, 0, 0, 2)
bufcopy.toString() //'He'
```

### 切片 buffer

如果要创建 buffer 的局部视图，则可以创建切片。 切片不是副本：原始 buffer 仍然是真正的来源。 如果那改变了，则切片也会改变。

使用 `slice()` 方法创建它。 第一个参数是起始位置，可以指定第二个参数作为结束位置：

```javascript
const buf = Buffer.from('Hey!')
buf.slice(0).toString() //Hey!
const slice = buf.slice(0, 2)
console.log(slice.toString()) //He
buf[1] = 111 //o
console.log(slice.toString()) //Ho
```

---

## 流

### 什么是流

流是为 Node.js 应用程序提供动力的基本概念之一。

它们是一种以高效的方式处理读/写文件、网络通信、或任何类型的端到端的信息交换。

流不是 Node.js 特有的概念。 它们是几十年前在 Unix 操作系统中引入的，程序可以通过管道运算符（`|`）对流进行相互交互。

例如，在传统的方式中，当告诉程序读取文件时，这会将文件从头到尾读入内存，然后进行处理。

使用流，则可以逐个片段地读取并处理（而无需全部保存在内存中）。

Node.js 的 [`stream` 模块](http://nodejs.cn/api/stream.html) 提供了构建所有流 API 的基础。 所有的流都是 [EventEmitter](http://nodejs.cn/api/events.html#events_class_eventemitter) 的实例。

### 为什么是流

相对于使用其他的数据处理方法，流基本上提供了两个主要优点：

- **内存效率**: 无需加载大量的数据到内存中即可进行处理。
- **时间效率**: 当获得数据之后即可立即开始处理数据，这样所需的时间更少，而不必等到整个数据有效负载可用才开始。

### 流的示例

一个典型的例子是从磁盘读取文件。

使用 Node.js 的 `fs` 模块，可以读取文件，并在与 HTTP 服务器建立新连接时通过 HTTP 提供文件：

```javascript
const http = require('http')
const fs = require('fs')

const server = http.createServer(function(req, res) {
  fs.readFile(__dirname + '/data.txt', (err, data) => {
    res.end(data)
  })
})
server.listen(3000)
```

`readFile()` 读取文件的全部内容，并在完成时调用回调函数。

回调中的 `res.end(data)` 会返回文件的内容给 HTTP 客户端。

如果文件很大，则该操作会花费较多的时间。 以下是使用流编写的相同内容：

```javascript
const http = require('http')
const fs = require('fs')

const server = http.createServer((req, res) => {
  const stream = fs.createReadStream(__dirname + '/data.txt')
  stream.pipe(res)
})
server.listen(3000)
```

当要发送的数据块已获得时就立即开始将其流式传输到 HTTP 客户端，而不是等待直到文件被完全读取。

### pipe()

上面的示例使用了 `stream.pipe(res)` 这行代码：在文件流上调用 `pipe()` 方法。

该代码的作用是什么？ 它获取来源流，并将其通过管道传输到目标流。

在来源流上调用它，在该示例中，文件流通过管道传输到 HTTP 响应。

`pipe()` 方法的返回值是目标流，这是非常方便的事情，它使得可以链接多个 `pipe()` 调用，如下所示：

```javascript
src.pipe(dest1).pipe(dest2)
```

此构造相对于：

```javascript
src.pipe(dest1)
dest1.pipe(dest2)
```

### 流驱动的 Node.js API

由于它们的优点，许多 Node.js 核心模块提供了原生的流处理功能，最值得注意的有：

- `process.stdin` 返回连接到 stdin 的流。
- `process.stdout` 返回连接到 stdout 的流。
- `process.stderr` 返回连接到 stderr 的流。
- `fs.createReadStream()` 创建文件的可读流。
- `fs.createWriteStream()` 创建到文件的可写流。
- `net.connect()` 启动基于流的连接。
- `http.request()` 返回 http.ClientRequest 类的实例，该实例是可写流。
- `zlib.createGzip()` 使用 gzip（压缩算法）将数据压缩到流中。
- `zlib.createGunzip()` 解压缩 gzip 流。
- `zlib.createDeflate()` 使用 deflate（压缩算法）将数据压缩到流中。
- `zlib.createInflate()` 解压缩 deflate 流。

### 不同类型的流

流分为四类：

- `Readable`: 可以通过管道读取、但不能通过管道写入的流（可以接收数据，但不能向其发送数据）。 当推送数据到可读流中时，会对其进行缓冲，直到使用者开始读取数据为止。
- `Writable`: 可以通过管道写入、但不能通过管道读取的流（可以发送数据，但不能从中接收数据）。
- `Duplex`: 可以通过管道写入和读取的流，基本上相对于是可读流和可写流的组合。
- `Transform`: 类似于双工流、但其输出是其输入的转换的转换流。

### 如何创建可读流

从 [`stream` 模块](http://nodejs.cn/api/stream.html)获取可读流，对其进行初始化并实现 `readable._read()` 方法。

首先创建流对象：

```javascript
const Stream = require('stream')
const readableStream = new Stream.Readable()
```

然后实现 `_read`：

```javascript
readableStream._read = () => {}
```

也可以使用 `read` 选项实现 `_read`：

```javascript
const readableStream = new Stream.Readable({
  read() {}
})
```

现在，流已初始化，可以向其发送数据了：

```javascript
readableStream.push('hi!')
readableStream.push('ho!')
```

### 如何创建可写流

若要创建可写流，需要继承基本的 `Writable` 对象，并实现其 `_write()` 方法。

首先创建流对象：

```javascript
const Stream = require('stream')
const writableStream = new Stream.Writable()
```

然后实现 `_write`：

```javascript
writableStream._write = (chunk, encoding, next) => {
  console.log(chunk.toString())
  next()
}
```

现在，可以通过以下方式传输可读流：

```javascript
process.stdin.pipe(writableStream)
```

### 如何从可读流中获取数据

如何从可读流中读取数据？ 使用可写流：

```javascript
const Stream = require('stream')

const readableStream = new Stream.Readable({
  read() {}
})
const writableStream = new Stream.Writable()

writableStream._write = (chunk, encoding, next) => {
  console.log(chunk.toString())
  next()
}

readableStream.pipe(writableStream)

readableStream.push('hi!')
readableStream.push('ho!')
```

也可以使用 `readable` 事件直接地消费可读流：

```javascript
readableStream.on('readable', () => {
  console.log(readableStream.read())
})
```

### 如何发送数据到可写流

使用流的 `write()` 方法：

```javascript
writableStream.write('hey!\n')
```

### 使用信号通知已结束写入的可写流

使用 `end()` 方法：

```javascript
const Stream = require('stream')

const readableStream = new Stream.Readable({
  read() {}
})
const writableStream = new Stream.Writable()

writableStream._write = (chunk, encoding, next) => {
  console.log(chunk.toString())
  next()
}

readableStream.pipe(writableStream)

readableStream.push('hi!')
readableStream.push('ho!')

writableStream.end()
```

---

## 开发环境与生产环境

Node.js 假定其始终运行在开发环境中。 可以通过设置 `NODE_ENV=production` 环境变量来向 Node.js 发出正在生产环境中运行的信号。

通常通过在 shell 中执行以下命令来完成：

```bash
export NODE_ENV=production
```

但最好将其放在的 shell 配置文件中（例如，使用 Bash shell 的 `.bash_profile`），否则当系统重启时，该设置不会被保留。

也可以通过将环境变量放在应用程序的初始化命令之前来应用它：

```bash
NODE_ENV=production node app.js
```

此环境变量是一个约定，在外部库中也广泛使用。

设置环境为 `production` 通常可以确保：

- 日志记录保持在最低水平。
- 更高的缓存级别以优化性能。

例如，如果 `NODE_ENV` 未被设置为 `production`，则 Express 使用的模板库 Pug 会在调试模式下进行编译。 Express 页面会在开发模式下按每个请求进行编译，而在生产环境中则会将其缓存。 还有更多的示例。

可以使用条件语句在不同的环境中执行代码：

```javascript
if (process.env.NODE_ENV === "development") {
  //...
}
if (process.env.NODE_ENV === "production") {
  //...
}
if(['production', 'staging'].indexOf(process.env.NODE_ENV) >= 0) {
  //...
})
```

例如，在 Express 应用中，可以使用此工具为每个环境设置不同的错误处理程序：

```javascript
if (process.env.NODE_ENV === "development") {
  app.use(express.errorHandler({ dumpExceptions: true, showStack: true }))
})

if (process.env.NODE_ENV === "production") {
  app.use(express.errorHandler())
})
```

---

## 错误处理

Node.js 中的错误通过异常进行处理。

### 创建异常

使用 `throw` 关键字创建异常：

```javascript
throw value
```

一旦 JavaScript 执行到此行，则常规的程序流会被停止，且控制会被交给最近的异常处理程序。

通常，在客户端代码中，`value` 可以是任何 JavaScript 值（包括字符串、数字、或对象）。

在 Node.js 中，我们不抛出字符串，而仅抛出 Error 对象。

### 错误对象

错误对象是 Error 对象的实例、或者继承自 Error 类（由 [Error 核心模块](http://nodejs.cn/api/errors.html)提供）：

```javascript
throw new Error('错误信息')
```

或：

```javascript
class NotEnoughCoffeeError extends Error {
  //...
}
throw new NotEnoughCoffeeError()
```

### 处理异常

异常处理程序是 `try`/`catch` 语句。

`try` 块中包含的代码行中引发的任何异常都会在相应的 `catch` 块中处理：

```javascript
try {
  //代码行
} catch (e) {}
```

在此示例中，`e` 是异常值。

可以添加多个处理程序，它们可以捕获各种错误。

### 捕获未捕获的异常

如果在程序执行过程中引发了未捕获的异常，则程序会崩溃。

若要解决此问题，则监听 `process` 对象上的 `uncaughtException` 事件：

```javascript
process.on('uncaughtException', err => {
  console.error('有一个未捕获的错误', err)
  process.exit(1) //强制性的（根据 Node.js 文档）
})
```

无需为此导入 `process` 核心模块，因为它是自动注入的。

### Promise 的异常

使用 promise 可以链接不同的操作，并在最后处理错误：

```javascript
doSomething1()
  .then(doSomething2)
  .then(doSomething3)
  .catch(err => console.error(err))
```

你怎么知道错误发生在哪里？ 你并不知道，但是你可以处理所调用的每个函数（`doSomethingX`）中的错误，并且在错误处理程序内部抛出新的错误，这就会调用外部的 `catch` 处理程序：

```javascript
const doSomething1 = () => {
  //...
  try {
    //...
  } catch (err) {
    //... 在本地处理
    throw new Error(err.message)
  }
  //...
}
```

为了能够在本地（而不是在调用的函数中）处理错误，则可以断开链条，在每个 `then()` 函数中创建函数并处理异常：

```javascript
doSomething1()
  .then(() => {
    return doSomething2().catch(err => {
      //处理错误
      throw err //打断链条
    })
  })
  .then(() => {
    return doSomething2().catch(err => {
      //处理错误
      throw err //打断链条
    })
  })
  .catch(err => console.error(err))
```

### async/await 的错误处理

使用 async/await 时，仍然需要捕获错误，可以通过以下方式进行操作：

```javascript
async function someFunction() {
  try {
    await someOtherFunction()
  } catch (err) {
    console.error(err.message)
  }
}
```

---

## 如何记录对象

当我们记录内容到控制台时，没有那么奢侈，因为它会将对象输出到 shell（如果你手动运行 Node.js 程序）或输出到日志文件。 你会获得对象的字符串表示形式。

在达到一定嵌套级别之前一切都很好。 在经过两个级别的嵌套后，Node.js 会放弃并打印 `[Object]` 作为占位符：

```javascript
const obj = {
  name: 'joe',
  age: 35,
  person1: {
    name: 'Tony',
    age: 50,
    person2: {
      name: 'Albert',
      age: 21,
      person3: {
        name: 'Peter',
        age: 23
      }
    }
  }
}
console.log(obj)


{
  name: 'joe',
  age: 35,
  person1: {
    name: 'Tony',
    age: 50,
    person2: {
      name: 'Albert',
      age: 21,
      person3: [Object]
    }
  }
}
```

如何打印整个对象？

最好的方法（同时保留漂亮的打印效果）是使用：

```javascript
console.log(JSON.stringify(obj, null, 2))
```

其中 `2` 是用于缩进的空格数。

另一种选择是使用：

```javascript
require('util').inspect.defaultOptions.depth = null
console.log(obj)
```

但是有个问题，第 2 级之后的嵌套对象会被展平，这可能是复杂对象的问题。