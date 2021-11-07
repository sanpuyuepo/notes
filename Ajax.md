# 原生Ajax

## Ajax简介

Ajax：Asynchronous JavaScript And XML，异步的 JS 和 XML。

通过 Ajax 在浏览器中向服务端**发送异步请求**，最大的优势是：**无刷新获取数据**。

---

## XML简介

XML 可扩展标记语言，传输和存储数据，与HTML类似，但是XML都是自定义标签，来表示数据，如：

```json
{
    "name": "sanpuyuepo",
    "age": 18
}
```

XML 表示为：

```xml
<student>
    <name>sanpuyuepo</name>
    <age>18</age>
</student>
```

这是最初的数据交换格式，现在用的是 json。

---

## Ajax 特点

### 优点

1. 与服务端通信时，无需刷新页面；
2. 允许根据用户事件更新部分页面内容。

### 缺点

1. 没有浏览历史，不能回退
2. 存在跨域问题
3. SEO 不友好



## Nodemon

nodemon is a tool that helps develop node.js based applications by automatically restarting the node application when file changes in the directory are detected.

全局安装：`npm install -g nodemon`

本地安装：`npm install --save-dev nodemon`

使用方法：直接运行 `nodemon [your node app] `

本地安装有个坑，不能直接执行nodemon命令，可以在package.json配置文件的脚本部分配置运行命令，如`"start": "nodemon originalAjax/server"`，。

原因：本地安装时，模块安装在项目的node_modules文件夹中，

全局安装时，会安装在node的安装目录下，根据由内向外原则，所以没问题

---

## IE缓存问题

IE浏览器会对Ajax请求结果做缓存，下次去发送相同请求获取相同的数据时，IE浏览器会去取缓存而不是请求最新的数据

解决方法：在初始化xhr对象的时候，在URL后加一个时间戳参数 `t`：

```js
xhr.open('GET', 'http://localhost:8000/server?t=' + Date.now());
```

---

## Ajax 请求超时





---

## 同源策略

Same-Origin-Policy，是浏览器的一种安全策略

同源策略：协议、域名、端口三者必须完全一致。

违背同源策略就是跨域

Ajax 默认遵守同源策略

## JSONP

JSON with Padding，非官方的跨域解决方案，只支持 GET 请求

HTML中某些标签如：`img` `link`  `script` 等天生具有跨域能力；

JSONP就是利用 `script` 标签的跨域能力来发送请求的。

**JSONP的使用**

1. 动态创建一个`script`标签

2. 设置script的src，设置回调函数用于接受响应数据，这个回调函数需要事先定义，这是因为**服务端要在返回的数据外层包裹一个客户端已经定义好的函数**：

    ```html
    <!-- 全局作用域下预先定义回调函数 -->
    <script>
        function callback(data) {
            const element = document.getElementById('text-area');
            element.innerHTML = data.name;
        }
    </script>
    <!-- 返回的数据会立马被浏览器当作javascript语句去执行, 所以返回的数据格式需要是js代码。
         所以在返回数据的时候，将数据当做函数的参数，这个函数就是回调函数 
         当数据返回到客户端的时候，执行这个回调函数对数据进行处理-->
    <!-- 
    script标签的请求必须在写在定义回调函数之后
    将回调函数名作为参数传递，在后端返回数据时使用
    -->
    <script src="https://photo.sina.cn/aj/index?page=1&cate=recommend&callback=callback"></script>
    ```
    

开发中可能会遇到多个 JSONP 请求的回调函数名是相同的，这时候就需要自己封装一个 JSONP

```js
function jsonp({ url, params, callback }) {
  return new Promise((resolve, reject) => {
    let script = document.createElement('script')
    window[callback] = function(data) {
      resolve(data)
      document.body.removeChild(script)
    }
    params = { ...params, callback } // wd=b&callback=show
    let arrs = []
    for (let key in params) {
      arrs.push(`${key}=${params[key]}`)
    }
    script.src = `${url}?${arrs.join('&')}`
    document.body.appendChild(script)
  })
}
jsonp({
  url: 'http://localhost:3000/say',
  params: { wd: 'Iloveyou' },
  callback: 'show'
}).then(data => {
  console.log(data)
})
```

---

## CORS

Cross-Origin-Resource-Sharing，跨域资源共享

是官方的跨域解决方案，不需要在客户端做任何特殊操作，完全在服务器中进行处理。

cors 新增了一组http首部字段，如 `Access-Control-Allow-Origin` 等，允许服务器声明哪些源站通过浏览器有权访问哪些资源

**原理**： 通过设置一个响应头，告诉浏览器该请求允许跨域，浏览器收到该响应以后会对响应放行

**使用**：主要是服务端的设置



