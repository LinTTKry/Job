# ES6与CommonJs的不同

### 为什么不在浏览器也是用CommonJS

CommonJS的 `require` 语法是同步的，当我们使用`require` 加载一个模块的时候，必须要等这个模块加载完后，才会执行后面的代码。如果知道这个事实，那我们的问题也就很容易回答了。**NodeJS** 是服务端，使用 `require` 语法加载模块，一般是一个文件，只需要从本地硬盘中读取文件，它的速度是比较快的。但是在浏览器端就不一样了，文件一般存放在服务器或者CDN上，如果使用同步的方式加载一个模块还需要由网络来决定快慢，可能时间会很长，这样浏览器很容易进入“假死状态”。

#### 两个重大的差异

相信大家或多或少听说过一些它们之间的差异，毕竟我们日常开发中都少不了跟它们打交道。其实，它们最大的两个差异就是：

* CommonJS模块输出的是一个值的拷贝，ES6 模块输出的是值的引用；
* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口

CommonJS输出的是值的拷贝，换句话说就是，一旦输出了某个值，如果模块内部后续的变化，影响不了外部对这个值的使用。具体例子：

```
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
复制代码
```

然后我们在其它文件中使用这个模块：

```
var mod = require('./lib');
console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

S6模块运行机制完全不一样，JS 引擎对脚本静态分析的时候，遇到模块加载命令`import`，就会生成一个只读引用。等到脚本真正执行的时候，再根据这个只读引用，到被加载的那个模块里去取值。

```
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
复制代码
```

上面代码说明，ES6 模块`import`的变量`counter`是可变的，完全反应其所在模块`lib.js`内部的变化。

&#x20;还有的不同就是`this`关键词，在ES6模块顶层，`this`指向`undefined`；而CommonJS模块的顶层的`this`指向当前模块
