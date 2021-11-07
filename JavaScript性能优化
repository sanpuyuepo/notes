# JavaScript性能优化

## 内存管理

开发者主动申请、使用、释放内存空间的过程叫内存管理。

## 垃圾回收

- JavaScript中的内存管理是自动的
- 对象不再引用时是垃圾
- 对象不能从根上访问到时是垃圾

可达对象：可以访问到的对象就是可达对象。可达的标准就是从根出发是否能够被找到。根可以理解为全局变量对象。在一个作用域链上，只要通过根可以有路径查找到的对象都是可达对象

将内存占用量保持在一个较小的值可以让页面性能更好。优化内存占用最佳的手段是保证在执行代码时只保存必要的数据。如果数据不再必要，将其设置为null，从而释放其引用，叫做<u>**解除引用**</u>。这个方法最适合全局变量和全局对象的属性。局部变量在超出作用域后会自动解除引用。

### GC算法

GC是一种机制，垃圾回收器完成具体工作：查找垃圾并释放回收空间。

GC里的垃圾：

- 程序不再需要使用的对象
- 程序不能再访问到的对象

常见GC算法：

1. 引用计数 reference counting
2. 标记清除 mark-and-sweep
3. 标记整理
4. 分代回收

#### 引用计数

- 核心思想：设置引用数，判断当前引用数是否为0
- 引用计数器
- 引用关系改变时修改引用数
- 引用数为0时立即回收

引用计数优点：

- 发现垃圾时立即回收
- 最大程度减少程序卡顿

引用计数缺点：

- 无法回收循环引用的对象
- 时间开销大：时刻监控某个对象的引用数是否需要修改，存在大量对象时时间开销大。

#### 标记清除

1. 核心思想：分标记和清除两个阶段
2. 遍历所有对象标记活动对象
3. 遍历所有对象清除没有标记的对象
4. 回收相应空间

标记清除优点：可以回收循环引用的对象

标记清除缺点：空间碎片化，找到垃圾对象空间后直接进行回收，而有可能产生大量碎片化空间

#### 标记整理

- 标记清除的增强
- 标记阶段的操作和和标记清除一样
- 清除阶段会先执行整理，移动对象位置
- 解决标记清除的空间碎片化问题

#### 增量标记

增量标记算法就是将连续的垃圾回收拆分成多个“小步”与程序运行交替完成，提高回收效率

---

### **V8引擎**

- 主流的JavaScript执行引擎。
- 采用即时编译，速度快。
- 内存设限：32位系统不超过800M，64位不超过1.5G

#### v8垃圾回收策略

- 采用分代回收的思想

- 内存分为新生代、老生代：

    <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102110424592.png" alt="image-20211102110424592" style="zoom:67%;" />

    1. 小空间（32M|16M）存储新生代对象：存活时间较短的对象
    2. 大空间（1.4G|700M）存储老生代对象：存活时间较长的对象：全局对象或闭包等

- 针对不同生代采用不同算法

#### v8常用的GC算法

1. 分代回收
2. 空间复制
3. 标记清除
4. 标记整理
5. 增量标记

#### **新生代对象回收**

- 采用复制算法+标记整理
-  新生代内存等分为两个空间：使用空间From，空闲空间To
- 活动对象存储于From空间
- 标记整理后将<u>**活动对象复制到To空间**</u>
    1. 复制过程可能出现晋升：将新生代对象移动至老生代
    2. 一轮GC过后还存活的新生代需要晋升
    3. To空间的使用率超过25%时需要晋升
- From和To交换空间完成释放，
- 空间复制的时候产生 From 和 To 二个等大小的空间，始终有一部分是空间不使用的，即使很小也是一种浪费，属于用空间换时间

#### 老生代对象回收

- 标记清除+标记整理+增量标记
- 首先使用标记清除完成垃圾空间的回收
- 采用标记整理进行空间优化
- 采用增量标记进行效率优化

#### 新生代老生代垃圾回收细节对比

1. 新生代空间换时间
2. 老生代不适合复制算法：空间浪费

---

### Performance工具介绍

Performance（性能）工具监控内存使用。浏览器自带。

<img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102112715806.png" alt="image-20211102112715806" style="zoom:67%;" />

点击录制，结束后得到：

![image-20211102113151706](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102113151706.png)

---

#### 内存问题的体现：

- 内存问题的外在表现
    1. 页面出现延迟加载或经常性暂停
    2. 页面持续性出现糟糕的性能：内存膨胀
    3. 页面的性能随时间延长越来越差：内存泄漏

**界定内存问题的标准**

- 内存泄漏：内存使用持续升高
- 内存膨胀：在多数设备上都存在糟糕的性能问题
- 频繁垃圾回收：通过内存变化图分析

#### 内存监控的几种方式

- ***浏览器任务管理器***

    shift + esc打开浏览器任务管理器，主要用于判断是否有问题，但不能定位出现问题的代码位置

    <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102121530036.png" alt="image-20211102121530036" style="zoom:67%;" />

    ---

- **Timeline 时序图记录**

    ![image-20211102164446811](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102164446811.png)

    ---

- **堆快照查找分离DOM**

    什么是分离DOM：脱离DOM树，但是JS仍对该DOM有引用的DOM节点。界面上不可见，但是仍然占据内存。

    ![image-20211102165824052](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102165824052.png)

    在快照中输入过滤字段：deta, 代表 Detached，意为分离的

    ![image-20211102165946730](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211102165946730.png)

    获取数据的即为分离DOM。

    ---

- **判断是否存在频繁垃圾回收**
    1. Timeline中频繁的上升下降
    2. 任务管理器中数据频繁的增加、减少

---

### V8引擎执行流程

V8架构演进历史：

![image-20211103093722242](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103093722242.png)

在加入Ignition解释器之前，v8是直接将源码编译为机器码的架构，内存消耗很大。

Ignition解释器之后引入字节码架构。

<img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103094141737.png" alt="image-20211103094141737" style="zoom:67%;" />

#### 执行过程

1. 解析器Parser将源码解析为抽象语法树AST（Abstract Syntax Tree）

    - **词法分析**：Scanner  将字符流即一行行代码，转换为tokens，token为字符或字符串。

        <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103095422943.png" alt="image-20211103095422943" style="zoom:67%;" />

        案例：

        ```js
        var a = 1;
        ```

        上面语句通过Scanner词法分析器分析后得到的tokens为：

        <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103100251449.png" alt="image-20211103100251449" style="zoom:50%;" />

        

    - **语法分析**：Parser 根据语法规则将tokens组成一个有嵌套层级的抽象语法树，过程中如果源码不符合语法规范，将终止解析并抛出语法错误。

        <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103095628552.png" alt="image-20211103095628552" style="zoom:67%;" />

        `var a = 1;`的抽象语法树：可以通过https://www.astexplorer.net/查看源码转换为AST。

        <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103100549312.png" alt="image-20211103100549312" style="zoom:67%;" />

    - **延迟解析**

        解析过程中，对于不是立即执行的函数，只进行预解析（Pre Parser），只有当函数调用的时候才对函数进行全量解析。

        **预解析**：

        - 跳过未被使用的代码
        - 不生成AST，创建无变量引用和声明的scopes
        - 依据规范抛出特点错误
        - 解析速度更快 

        **全量解析**：

        - 解析被使用的代码
        - 生产AST
        - 构建具体的scopes信息，变量引用、声明等
        - 抛出所有语法错误

        <img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103103248629.png" alt="image-20211103103248629" style="zoom:67%;" />

        

2. 解释器Ignition将AST翻译为字节码，边解释边执行。在此过程中，解释器会记录特定代码的运行次数，当运行次数超过某个阈值，该段代码被标记为热代（hot code）并且将运行信息反馈给优化编译器TurboFan。

    

3. TurboFan优化编译器根据反馈信息优化并编译字节码，最终生成优化后的机器码。这样该段hot code再次执行时解释器就直接使用优化后的机器码执行，不用再次解释，从而提高代码运行效率。TurboFan主要通过以下两个算法进行优化编译：

    - 内联算法
    - 逃逸分析

4. 这种在运行时编译代码的技术成为JIT，即时编译。

---

## 堆栈处理

### 堆栈准备

- JS执行环境，如V8引擎
- 执行环境栈，ECStack, execution context stack
- 执行上下文
- VO(G)，全局变量对象

![image-20211103110106573](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103110106573.png)

---

### 引用类型堆栈处理

<img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103113120364.png" alt="image-20211103113120364" style="zoom:80%;" />

<img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103113509098.png" alt="image-20211103113509098" style="zoom:80%;" />

<img src="JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103114222177.png" alt="image-20211103114222177" style="zoom:80%;" />

---

---

### 函数堆栈处理

![image-20211103121843479](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103121843479.png)

---

### 闭包堆栈处理

![image-20211103123346823](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211103123346823.png)

---

### 闭包与GC堆栈

![image-20211104140802426](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104140802426.png)

![image-20211104140322778](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104140322778.png)

![image-20211104140410414](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104140410414.png)

---

## 循环添加事件实现

```js
const buttons = document.querySelectorAll('button');

// for (var i = 0; i < buttons.length; i++) {
//     buttons[i].onclick = function() {
//         console.log(`current index is: ${i}`);
//     }
// }

// 基于闭包：方法1
for (var i = 0; i < buttons.length; i++) {
    (function(x) {
        buttons[x].onclick = function() {
            console.log(`current index is: ${x}`);
        }
    })(i);
}

// 基于闭包：方法2
for (var i = 0; i < buttons.length; i++) {
    buttons[i].onclick = (function(x) {
       return function() {
        console.log(`current index is: ${x}`);
       }
    })(i);
}
/** 
* 基于闭包的方法，需要开辟新的内存空间，内存消耗大，见下图分析：
*/

// 使用 let 关键字
for (let i = 0; i < buttons.length; i++) {
    buttons[i].onclick = function() {
        console.log(`current index is: ${i}`);
    }
}

// 添加自定义属性 不需要开辟新的内存空间
for (var i = 0; i < buttons.length; i++) {
    buttons[i].myIndex = i;
    buttons[i].onclick = function() {
        console.log(`current index is: ${this.myIndex}`);
    }
}

// 事件委托机制
document.body.onclick = function(event) {
    let target = event.target;
    let targetDom = target.tagName;
    if (targetDom === "BUTTON") {
        let index = target.getAttribute('index'); // index 是 HTML 中 button 的属性
        console.log(`The current index is: ${index}`);
    }
}

```

**上述代码堆栈分析**：

![image-20211104145414430](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104145414430.png)

![image-20211104145924354](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104145924354.png)

![image-20211104150322468](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104150322468.png)

![image-20211104151616478](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104151616478.png)

---

## JSBench的使用

JSBench，一个在线测试JS代码执行效率的网站。https://jsbench.me/

---

## 变量局部化

变量尽量不要声明在全局对象上，这样可以提高代码执行效率（减少变量查找访问层级）

![image-20211104153557950](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104153557950.png)

---

## 缓存数据

缓存数据：对多次需要使用的数据提前缓存，后续使用。

![image-20211104160433163](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211104160433163.png)

减少不必要的声明和语句，减少JS引擎的词法及语法分析。

---

## 防抖&节流

在一些高频率事件触发的场景下不希望对应的事件处理函数多次执行，如：滚动事件、输入的模糊匹配、轮播图切换、点击操作等

浏览器默认都有自己的监听事件间隔，如Chrome为 4~6 ms，如果检测到多次事件的监听执行，会造成不必要的资源浪费。

### 防抖 debounce

在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

```js
function debounce(callback, delay, immediate) {
    // 参数判断
    if (!(typeof callback === 'function')) {
        throw new TypeError('callback must be a function');
    }
    if (typeof delay === 'undefined') wait = 300;
    if (typeof delay === 'boolean') {
        immediate = delay;
        delay = 300;
    }
    if (!(typeof immediate === 'boolean')) {
        immediate = false;
    }

    // 防抖
    let timer = null;
    return function(...args) {
        let self = this;
        let init = immediate && !timer; // 是否立即执行且只执行第一次
        clearTimeout(timer);
        timer = setTimeout(() => {
            timer = null;
            !immediate ? callback.apply(self, ...args) : null;
        }, delay);

        // immediate = true 表示立即执行
        // 如果想要实现只在第一次触发执行，添加 timer 为 null作为判断，只要timer为null，意味着没有第二次第三次...点击
        init ? callback.apply(self, ...args) : null;
    }
}
```



### 节流 throttle

规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。

```js
function throttle(handler, delay) {
    // 参数判断
    if (!(typeof handler === 'function')) {
        throw new TypeError('handler must be a function');
    }
    if (typeof delay === 'undefined') delay = 300;

    let previous = 0; // 上次执行的时间
    let timer = null; 
    return function (...args) {
        let self = this;

        let now = new Date();
        let interval = now - previous; // 两次触发的时间间隔, 大于 delay 说明可以执行，小于delay 说明高频触发

        if (interval >= delay) {
            // 浏览器自动触发事件和节流函数触发事件时间点恰好相同时需走这一步，需要清除定时器
            clearTimeout(timer);
            timer = null;
            handler.apply(self, ...args);
            previous = new Date();
        } else if (!timer){
            // 开启定时器，让handler在 delay - interval 之后去执行,
            timer = setTimeout(() => {
                // 有一个定时器后不需要再开启新的定时器
                clearTimeout(timer);
                timer = null;
                handler.apply(self, ...args);
                previous = new Date();
            }, delay - interval);
        }
    }
}
```

---

## 减少条件判断层级

![image-20211105145952579](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211105145952579.png)

---

## 减少循环体活动

以for循环举例：

![image-20211105151226949](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211105151226949.png)

循环体内重复用到的数据，这些数据在循环体内是不变的，应将其定义在循环体外。

还有对于循环的选择，如：

![image-20211105151801656](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211105151801656.png)

---

## 字面量与构造式

![image-20211105152750345](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211105152750345.png)

如果是基本数据类型，效率差别更大：

![image-20211105153357846](JavaScript%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.assets/image-20211105153357846.png)

