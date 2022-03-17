# async/await：使用同步的方式去写异步代码

​下面代码展示的是使用 fetch 来实现这样的需求，fetch 被定义在 window 对象中，可以用它来发起对远程资源的请求，该方法返回的是一个 Promise 对象，这和我们上篇文章中讲的 XFetch 很像，只不过 fetch 是浏览器原生支持的，并有没利用 XMLHttpRequest 来封装。

```javascript
fetch('https://www.geekbang.org')
      .then((response) => {
          console.log(response)
          return fetch('https://www.geekbang.org/test')
      }).then((response) => {
          console.log(response)
      }).catch((error) => {
          console.log(error)
      })
```

从这段 Promise 代码可以看出来，使用 promise.then 也是相当复杂，虽然整个请求流程已经线性化了，但是代码里面包含了大量的 then 函数，使得代码依然不是太容易阅读。<mark style="color:blue;">**基于这个原因，ES7 引入了 async/await，这是 JavaScript 异步编程的一个重大改进，提供了在不阻塞主线程的情况下使用同步代码实现异步访问资源的能力，并且使得代码逻辑更加清晰**</mark>。

```javascript
async function foo(){
  try{
    let response1 = await fetch('https://www.geekbang.org')
    console.log('response1')
    console.log(response1)
    let response2 = await fetch('https://www.geekbang.org/test')
    console.log('response2')
    console.log(response2)
  }catch(err) {
       console.error(err)
  }
}
foo()
```

通过上面代码，你会发现整个异步处理的逻辑都是使用同步代码的方式来实现的，而且还支持 try catch 来捕获异常，这就是完全在写同步代码，所以是非常符合人的线性思维的。但是很多人都习惯了异步回调的编程思维，对于这种采用同步代码实现异步逻辑的方式，还需要一个转换的过程，因为这中间隐藏了一些容易让人迷惑的细节。

本文我们首先介绍生成器（Generator）是如何工作的，接着讲解 Generator 的底层实现机制——协程（Coroutine）；又因为 async/await 使用了 Generator 和 Promise 两种技术，所以紧接着我们就通过 Generator 和 Promise 来分析 async/await 到底是如何以同步的方式来编写异步代码的。

## 生成器&#x20;

<mark style="color:red;">**生成器函数是一个带星号函数，而且是可以暂停执行和恢复执行的。**</mark>

```javascript
function* genDemo() {
    console.log("开始执行第一段")
    yield 'generator 2'

    console.log("开始执行第二段")
    yield 'generator 2'

    console.log("开始执行第三段")
    yield 'generator 2'

    console.log("执行结束")
    return 'generator 2'
}

console.log('main 0')
let gen = genDemo()
console.log(gen.next().value)
console.log('main 1')
console.log(gen.next().value)
console.log('main 2')
console.log(gen.next().value)
console.log('main 3')
console.log(gen.next().value)
console.log('main 4')
```

执行上面这段代码，观察输出结果，你会发现函数 genDemo 并不是一次执行完的，全局代码和 genDemo 函数交替执行。其实这就是生成器函数的特性，可以暂停执行，也可以恢复执行。下面我们就来看看生成器函数的具体使用方式：

1. 在生成器函数内部执行一段代码，如果遇到 yield 关键字，那么 JavaScript 引擎将返回关键字后面的内容给外部，并暂停该函数的执行。
2. 外部函数可以通过 next 方法恢复函数的执行。

## 协程

<mark style="color:red;">**协程是一种比线程更加轻量级的存在**</mark>。你可以把协程看成是跑在线程上的任务，一个线程上可以存在多个协程，<mark style="color:red;">**但是在线程上同时只能执行一个协程**</mark>，比如当前执行的是 A 协程，要启动 B 协程，那么 A 协程就需要将主线程的控制权交给 B 协程，这就体现在 A 协程暂停执行，B 协程恢复执行；同样，也可以从 B 协程中启动 A 协程。<mark style="color:red;">**通常，如果从 A 协程启动 B 协程，我们就把 A 协程称为 B 协程的父协程**</mark>。

正如<mark style="color:red;">**一个进程可以拥有多个线程一样，一个线程也可以拥有多个协程**</mark>。最重要的是，协程不是被操作系统内核所管理，而完全是<mark style="color:red;">**由程序所控制（也就是在用户态执行）**</mark>。这样带来的好处就是性能得到了很大的提升，不会像线程切换那样消耗资源。

为了让你更好地理解协程是怎么执行的，我结合上面那段代码的执行过程，画出了下面的“协程执行流程图”:

​![](<../../.gitbook/assets/image (83).png>)

从图中可以看出来协程的四点规则：

* 通过调用生成器函数 genDemo 来创建一个协程 gen，创建之后，gen 协程并没有立即执行。
* 要让 gen 协程执行，需要通过调用 gen.next。
* 当协程正在执行的时候，可以通过 yield 关键字来暂停 gen 协程的执行，并返回主要信息给父协程。
* 如果协程在执行期间，遇到了 return 关键字，那么 JavaScript 引擎会结束当前协程，并将 return 后面的内容返回给父协程。

父协程有自己的调用栈，gen 协程时也有自己的调用栈，当 gen 协程通过 yield 把控制权交给父协程时，V8 是如何切换到父协程的调用栈？当父协程通过 gen.next 恢复 gen 协程时，又是<mark style="color:red;">**如何切换 gen 协程的调用栈？**</mark>

要搞清楚上面的问题，你需要关注以下两点内容。

* 第一点：gen 协程和父协程是在主线程上交互执行的，并不是并发执行的，<mark style="color:red;">**它们之前的切换是通过 yield 和 gen.next 来配合完成的**</mark>。
* 第二点：<mark style="color:red;">**当在 gen 协程中调用了 yield 方法时，JavaScript 引擎会保存 gen 协程当前的调用栈信息，并恢复父协程的调用栈信息。**</mark>同样，当在父协程中执行 gen.next 时，JavaScript 引擎会保存父协程的调用栈信息，并恢复 gen 协程的调用栈信息。

为了直观理解父协程和 gen 协程是如何切换调用栈的：

![](<../../.gitbook/assets/image (63).png>)

其实在 JavaScript 中，生成器就是协程的一种实现方式，这样相信你也就理解什么是生成器了。那么接下来，我们使用生成器和 Promise 来改造开头的那段 Promise 代码。

```javascript
//foo函数
function* foo() {
    let response1 = yield fetch('https://www.geekbang.org')
    console.log('response1')
    console.log(response1)
    let response2 = yield fetch('https://www.geekbang.org/test')
    console.log('response2')
    console.log(response2)
}

//执行foo函数的代码
let gen = foo()
function getGenPromise(gen) {
    return gen.next().value
}
getGenPromise(gen).then((response) => {
    console.log('response1')
    console.log(response)
    return getGenPromise(gen)
}).then((response) => {
    console.log('response2')
    console.log(response)
})
```

从图中可以看到，foo 函数是一个生成器函数，在 foo 函数里面实现了用同步代码形式来实现异步操作；但是在 foo 函数外部，我们还需要写一段执行 foo 函数的代码，如上述代码的后半部分所示，那下面我们就来分析下这段代码是如何工作的。

* 首先执行的是let gen = foo()，创建了 gen 协程。
* 然后在父协程中通过执行 gen.next 把主线程的控制权交给 gen 协程。
* gen 协程获取到主线程的控制权后，就调用 fetch 函数创建了一个 Promise 对象 response1，然后通过 yield 暂停 gen 协程的执行，并将 response1 返回给父协程。
* 父协程恢复执行后，调用 response1.then 方法等待请求结果。
* 等通过 fetch 发起的请求完成之后，会调用 then 中的回调函数，then 中的回调函数拿到结果之后，通过调用 gen.next 放弃主线程的控制权，将控制权交 gen 协程继续执行下个请求。

以上就是协程和 Promise 相互配合执行的一个大致流程。不过通常，我们把执行生成器的代码封装成一个函数，并把这个执行生成器代码的函数称为执行器（可参考著名的 co 框架），如下面这种方式：

```javascript
function* foo() {
    let response1 = yield fetch('https://www.geekbang.org')
    console.log('response1')
    console.log(response1)
    let response2 = yield fetch('https://www.geekbang.org/test')
    console.log('response2')
    console.log(response2)
}
co(foo());
```

通过使用生成器配合执行器，就能实现使用同步的方式写出异步代码了，这样也大大加强了代码的可读性。

## async/await

其实 async/await 技术背后的秘密就是 Promise 和生成器应用，往低层说就是微任务和协程应用。要搞清楚 async 和 await 的工作原理，我们就得对 async 和 await 分开分析。

### 1. async

async 是一个通过<mark style="color:red;">**异步执行并隐式返回 Promise 作为结果**</mark>的函数。

```javascript
async function foo() {
    return 2
}
console.log(foo())  // Promise {<resolved>: 2}
```

### 2. await

```javascript
async function foo() {
    console.log(1)
    let a = await 100
    console.log(a)
    console.log(2)
}
console.log(0)
foo()
console.log(3)
```

![](<../../.gitbook/assets/image (215).png>)

结合上图，我们来一起分析下 async/await 的执行流程。

首先，执行console.log(0)这个语句，打印出来 0。

紧接着就是执行 foo 函数，由于 foo 函数是被 async 标记过的，所以当进入该函数的时候，JavaScript 引擎会保存当前的调用栈等信息，然后执行 foo 函数中的console.log(1)语句，并打印出 1。

<mark style="color:red;">**接下来就执行到 foo 函数中的await 100这个语句了，这里是我们分析的重点，因为在执行await 100这个语句时，JavaScript 引擎在背后为我们默默做了太多的事情，那么下面我们就把这个语句拆开，来看看 JavaScript 到底都做了哪些事情。**</mark>

<mark style="color:red;">**当执行到await 100时，会默认创建一个 Promise 对象，代码如下所示：**</mark>

```javascript
let promise_ = new Promise((resolve,reject){
  resolve(100)
})
```

<mark style="color:red;">**在这个 promise\_ 对象创建的过程中，我们可以看到在 executor 函数中调用了 resolve 函数，JavaScript 引擎会将该任务提交给微任务队列。**</mark>

<mark style="color:red;">**然后 JavaScript 引擎会暂停当前协程的执行，将主线程的控制权转交给父协程执行，同时会将 promise\_ 对象返回给父协程。**</mark>

<mark style="color:red;">**主线程的控制权已经交给父协程了，这时候父协程要做的一件事是调用 promise\_.then 来监控 promise 状态的改变。**</mark>

<mark style="color:red;">**接下来继续执行父协程的流程，这里我们执行console.log(3)，并打印出来 3。随后父协程将执行结束，在结束之前，会进入微任务的检查点，然后执行微任务队列，微任务队列中有resolve(100)的任务等待执行，执行到这里的时候，会触发 promise\_.then 中的回调函数，如下所示：**</mark>

```javascript
promise_.then((value)=>{
   //回调函数被激活后
  //将主线程控制权交给foo协程，并将vaule值传给协程
})
```

<mark style="color:red;">**该回调函数被激活以后，会将主线程的控制权交给 foo 函数的协程，并同时将 value 值传给该协程。**</mark>

<mark style="color:red;">**foo 协程激活之后，会把刚才的 value 值赋给了变量 a，然后 foo 协程继续执行后续语句，执行完成之后，将控制权归还给父协程。**</mark>

以上就是 await/async 的执行流程。正是因为 async 和 await 在背后为我们做了大量的工作，所以我们才能用同步的方式写出异步代码来。

## 思考问题

```javascript
async function foo() {
    console.log('foo')
}
async function bar() {
    console.log('bar start')
    await foo()
    console.log('bar end')
}
console.log('script start')
setTimeout(function () {
    console.log('setTimeout')
}, 0)
bar();
new Promise(function (resolve) {
    console.log('promise executor')
    resolve();
}).then(function () {
    console.log('promise then')
})
console.log('script end')
```

1.首先执行console.log('script start');打印出script start&#x20;

2.接着遇到定时器，创建一个新任务，放在延迟队列中&#x20;

3.紧接着执行bar函数，由于bar函数被async标记的，所以进入该函数时，JS引擎会保存当前调用栈等信息，然后执行bar函数中的console.log('bar start');语句，打印bar start。

4.接下来执行到bar函数中的await foo();语句，执行foo函数，也由于foo函数被async标记的，所以进入该函数时，JS引擎会保存当前调用栈等信息，然后执行foo函数中的console.log('foo');语句，打印foo。

5.执行到await foo()时，会默认创建一个Promise对象&#x20;

6.在创建Promise对象过程中，调用了resolve()函数，且JS引擎将该任务交给微任务队列&#x20;

7.然后JS引擎会暂停当前协程的执行，将主线程的控制权交给父协程，同时将创建的Promise对象返回给父协程&#x20;

8.主线程的控制权交给父协程后，父协程就调用该Promise对象的then()方法监控该Promise对象的状态改变&#x20;

9.接下来继续父协程的流程，执行new Promise()，打印输出promise executor，其中调用了 resolve 函数，JS引擎将该任务添加到微任务队列队尾&#x20;

10.继续执行父协程上的流程，执行console.log('script end');，打印出来script end&#x20;

11.随后父协程将执行结束，在结束前，会进入微任务检查点，然后执行微任务队列，微任务队列中有两个微任务等待执行，先执行第一个微任务，触发第一个promise.then()中的回调函数，将主线程的控制权交给bar函数的协程，bar函数的协程激活后，继续执行后续语句，执行 console.log('bar end');，打印输出bar end&#x20;

12.bar函数协程执行完成后，执行微任务队列中的第二个微任务，触发第二个promise.then()中的回调函数，该回调函数被激活后，执行console.log('promise then');，打印输出promise then&#x20;

13.执行完之后，将控制权归还给主线程，当前任务执行完毕，取出延迟队列中的任务，执行console.log('setTimeout');，打印输出setTimeout。

## 总结1：generator 函数是如何暂停执行程序的？

答案是通过协程来控制程序执行。 generator 函数是一个生成器，执行它会返回一个迭代器，这个迭代器同时也是一个协程。一个线程中可以有多个协程，但是同时只能有一个协程在执行。线程的执行是在内核态，是由操作系统来控制；协程的执行是在用户态，是完全由程序来进行控制，通过调用生成器的next()方法可以让该协程执行，通过yield关键字可以让该协程暂停，交出主线程控制权，通过return 关键字可以让该协程结束。协程切换是在用户态执行，而线程切换时需要从用户态切换到内核态，在内核态进行调度，协程相对于线程来说更加轻量、高效。

## 总结2：async function实现原理？

async function 是通过 promise + generator 来实现的。generator 是通过协程来控制程序调度的。 ​在协程中执行异步任务时，<mark style="color:red;">**先用promise封装该异步任务，如果异步任务完成，会将其结果放入微任务队列中，然后通过yield 让出主线程执行权，继续执行主线程js，主线程js执行完毕后，会去扫描微任务队列，如果有任务则取出任务进行执行，这时通过调用迭代器的next(result)方法，并传入任务执行结果result，将主线程执行权转交给该协程继续执行，并且将result赋值给yield 表达式左边的变量，从而以同步的方式实现了异步编程。**</mark>
