# Promise：使用Promise，告别回调函数

## ​异步编程的问题：代码逻辑不连续

JavaScript 的异步编程模型，相对于页面来说，主线程就是它整个的世界，所以在执行一项耗时的任务时，比如下载网络文件任务、获取摄像头等设备信息任务，这些任务都会放到页面主线程之外的进程或者线程中去执行，这样就避免了耗时任务“霸占”页面主线程的情况，可以结合下图来看看这个处理过程：

![](<../../.gitbook/assets/image (151).png>)

上图展示的是一个标准的异步编程模型，<mark style="color:red;">**页面主线程发起了一个耗时的任务，并将任务交给另外一个进程去处理，这时页面主线程会继续执行消息队列中的任务。等该进程处理完这个任务后，会将该任务添加到渲染进程的消息队列中，并排队等待循环系统的处理。排队结束之后，循环系统会取出消息队列中的任务进行处理，并触发相关的回调操作。**</mark>

这就是页面编程的一大特点：<mark style="color:blue;">**异步回调**</mark>。

Web 页面的单线程架构决定了异步回调，而异步回调影响到了我们的编码方式，到底是如何影响的呢？假设有一个下载的需求，使用 XMLHttpRequest 来实现，具体的实现方式你可以参考下面这段代码：

```javascript
//执行状态
function onResolve(response){console.log(response) }
function onReject(error){console.log(error) }

let xhr = new XMLHttpRequest()
xhr.ontimeout = function(e) { onReject(e)}
xhr.onerror = function(e) { onReject(e) }
xhr.onreadystatechange = function () { onResolve(xhr.response) }

//设置请求类型，请求URL，是否同步信息
let URL = 'https://time.geekbang.com'
xhr.open('Get', URL, true);

//设置参数
xhr.timeout = 3000 //设置xhr请求的超时时间
xhr.responseType = "text" //设置响应返回的数据格式
xhr.setRequestHeader("X_TEST","time.geekbang")

//发出请求
xhr.send();
```

我们执行上面这段代码，可以正常输出结果的。但是，这短短的一段代码里面竟然出现了五次回调，这么多的回调会导致代码的逻辑不连贯、不线性，非常不符合人的直觉，这就是异步回调影响到我们的编码方式。那有什么方法可以解决这个问题吗？

当然有，我们可以封装这堆凌乱的代码，降低处理异步回调的次数。

## 封装异步代码，让处理流程变得线性

由于我们重点关注的是输入内容（请求信息）和输出内容（回复信息），至于中间的异步请求过程，我们不想在代码里面体现太多，因为这会干扰核心的代码逻辑。整体思路如下图所示：

![](<../../.gitbook/assets/image (88).png>)

图中你可以看到，我们将 XMLHttpRequest 请求过程的代码封装起来了，重点关注输入数据和输出结果。那我们就按照这个思路来改造代码。首先，我们把输入的 HTTP 请求信息全部保存到一个 request 的结构中，包括请求地址、请求头、请求方式、引用地址、同步请求还是异步请求、安全设置等信息。request 结构如下所示：

```javascript
//makeRequest用来构造request对象
function makeRequest(request_url) {
    let request = {
        method: 'Get',
        url: request_url,
        headers: '',
        body: '',
        credentials: false,
        sync: true,
        responseType: 'text',
        referrer: ''
    }
    return request
}
```

然后就可以封装请求过程了，这里我们将所有的请求细节封装进 XFetch 函数，XFetch 代码如下所示：

```javascript
//[in] request，请求信息，请求头，延时值，返回类型等
//[out] resolve, 执行成功，回调该函数
//[out] reject  执行失败，回调该函数
function XFetch(request, resolve, reject) {
    let xhr = new XMLHttpRequest()
    xhr.ontimeout = function (e) { reject(e) }
    xhr.onerror = function (e) { reject(e) }
    xhr.onreadystatechange = function () {
        if (xhr.status = 200)
            resolve(xhr.response)
    }
    xhr.open(request.method, URL, request.sync);
    xhr.timeout = request.timeout;
    xhr.responseType = request.responseType;
    //补充其他请求信息
    //...
    xhr.send();
}
```

这个 XFetch 函数需要一个 request 作为输入，然后还需要两个回调函数 resolve 和 reject，当请求成功时回调 resolve 函数，当请求出现问题时回调 reject 函数。

有了这些后，我们就可以来实现业务代码了，具体的实现方式如下所示：

```javascript
XFetch(makeRequest('https://time.geekbang.org'),
    function resolve(data) {
        console.log(data)
    }, function reject(e) {
        console.log(e)
    })
```

## 新的问题：回调地狱

```javascript
XFetch(makeRequest('https://time.geekbang.org/?category'),
      function resolve(response) {
          console.log(response)
          XFetch(makeRequest('https://time.geekbang.org/column'),
              function resolve(response) {
                  console.log(response)
                  XFetch(makeRequest('https://time.geekbang.org')
                      function resolve(response) {
                          console.log(response)
                      }, function reject(e) {
                          console.log(e)
                      })
              }, function reject(e) {
                  console.log(e)
              })
      }, function reject(e) {
          console.log(e)
      })
```

这段代码之所以看上去很乱，归结其原因有两点：

* 第一是嵌套调用，下面的任务依赖上个任务的请求结果，并在上个任务的回调函数内部执行新的业务逻辑，这样当嵌套层次多了之后，代码的可读性就变得非常差了。
* 第二是<mark style="color:blue;">**任务的不确定性，执行每个任务都有两种可能的结果（成功或者失败），所以体现在代码中就需要对每个任务的执行结果做两次判断，这种对每个任务都要进行一次额外的错误处理的方式，明显增加了代码的混乱程度。**</mark>

原因分析出来后，那么问题的解决思路就很清晰了：

* <mark style="color:blue;">**消灭嵌套调用；**</mark>
* <mark style="color:red;">**合并多个任务的错误处理**</mark>

## Promise：消灭嵌套调用和多次错误处理

首先，我们使用 Promise 来重构 XFetch 的代码，示例代码如下所示：

```javascript
function XFetch(request) {
  function executor(resolve, reject) {
      let xhr = new XMLHttpRequest()
      xhr.open('GET', request.url, true)
      xhr.ontimeout = function (e) { reject(e) }
      xhr.onerror = function (e) { reject(e) }
      xhr.onreadystatechange = function () {
          if (this.readyState === 4) {
              if (this.status === 200) {
                  resolve(this.responseText, this)
              } else {
                  let error = {
                      code: this.status,
                      response: this.response
                  }
                  reject(error, this)
              }
          }
      }
      xhr.send()
  }
  return new Promise(executor)
}
```

接下来，我们再利用 XFetch 来构造请求流程，代码如下：

```javascript
var x1 = XFetch(makeRequest('https://time.geekbang.org/?category'))
var x2 = x1.then(value => {
    console.log(value)
    return XFetch(makeRequest('https://www.geekbang.org/column'))
})
var x3 = x2.then(value => {
    console.log(value)
    return XFetch(makeRequest('https://time.geekbang.org'))
})
x3.catch(error => {
    console.log(error)
}
```

* 首先我们引入了 Promise，在调用 XFetch 时，会返回一个 Promise 对象。
* 构建 Promise 对象时，需要传入一个 executor 函数，XFetch 的主要业务流程都在 executor 函数中执行。如果运行在 excutor 函数中的业务执行成功了，会调用 resolve 函数；
* 如果执行失败了，则调用 reject 函数。
* 在 excutor 函数中调用 resolve 函数时，会触发 promise.then 设置的回调函数；而调用 reject 函数时，会触发 promise.catch 设置的回调函数。

Promise 主要通过下面两步解决嵌套回调问题的。

首先，<mark style="color:red;">**Promise 实现了回调函数的延时绑定**</mark>。回调函数的延时绑定在代码上体现就是先创建 Promise 对象 x1，通过 Promise 的构造函数 executor 来执行业务逻辑；创建好 Promise 对象 x1 之后，再使用 x1.then 来设置回调函数:

```javascript
//创建Promise对象x1，并在executor函数中执行业务逻辑
function executor(resolve, reject){
    resolve(100)
}
let x1 = new Promise(executor)


//x1延迟绑定回调函数onResolve
function onResolve(value){
    console.log(value)
}
x1.then(onResolve)
```

其次，需要将回调函数 onResolve 的返回值穿透到最外层。因为我们会根据 onResolve 函数的传入值来决定创建什么类型的 Promise 任务，创建好的 Promise 对象需要返回到最外层，这样就可以摆脱嵌套循环了。

![](<../../.gitbook/assets/image (59).png>)

现在我们知道了 Promise 通过回调函数延迟绑定和回调函数返回值穿透的技术，解决了循环嵌套。之所以<mark style="color:blue;">**可以使用最后一个对象来捕获所有异常，是**</mark><mark style="color:red;">**因为 Promise 对象的错误具有“冒泡”性质，会一直向后传递，**</mark><mark style="color:blue;">**直到被 onReject 函数处理或 catch 语句捕获为止。具备了这样“冒泡”的特性后，就不需要在每个 Promise 对象中单独捕获异常了。**</mark>

## Promise 与微任务

```javascript
function executor(resolve, reject) {
    resolve(100)
}
let demo = new Promise(executor)

function onResolve(value){
    console.log(value)
}
demo.then(onResolve)
```

通过模拟源码查看背后的执行顺序：

```javascript
function Bromise(executor) {
    var onResolve_ = null
    var onReject_ = null
     //模拟实现resolve和then，暂不支持rejcet
    this.then = function (onResolve, onReject) {
        onResolve_ = onResolve
    };
    function resolve(value) {
          //setTimeout(()=>{
            onResolve_(value)
           // },0)
    }
    executor(resolve, null);
}
```

观察上面这段代码，我们实现了自己的构造函数、resolve、then 方法。接下来我们使用 Bromise 来实现我们的业务代码，实现后的代码如下所示：

```javascript
function executor(resolve, reject) {
    resolve(100)
}
//将Promise改成我们自己的Bromsie
let demo = new Bromise(executor)

function onResolve(value){
    console.log(value)
}
demo.then(onResolve)
```

执行这段代码，我们发现执行出错，输出的内容是：

```javascript
Uncaught TypeError: onResolve_ is not a function
    at resolve (<anonymous>:10:13)
    at executor (<anonymous>:17:5)
    at new Bromise (<anonymous>:13:5)
    at <anonymous>:19:12
```

之所以出现这个错误，是由于 Bromise 的延迟绑定导致的，<mark style="color:red;">**在调用到 onResolve\_ 函数的时候，Bromise.then 还没有执行，所以执行上述代码的时候，当然会报“onResolve\_ is not a function**</mark>**“**的错误了。

正是因为此，我们要改造 Bromise 中的 resolve 方法，让 resolve 延迟调用 onResolve\_。

<mark style="color:red;">**要让 resolve 中的 onResolve\_ 函数延后执行，可以在 resolve 函数里面加上一个定时器，让其延时执行 onResolve\_ 函数**</mark>，你可以参考下面改造后的代码：

```javascript
function resolve(value) {
          setTimeout(()=>{
              onResolve_(value)
            },0)
    }
```

<mark style="color:red;"><mark style="background-color:orange;">**上面采用了定时器来推迟 onResolve 的执行，不过使用定时器的效率并不是太高，好在我们有微任务，所以 Promise 又把这个定时器改造成了微任务了，这样既可以让 onResolve\_ 延时被调用，又提升了代码的执行效率。这就是 Promise 中使用微任务的原由了**<mark style="background-color:orange;"></mark><mark style="color:red;">。</mark>

## <mark style="color:red;">问题1：Promise 中为什么要引入微任务？</mark>

由于promise采用.then延时绑定回调机制，而new Promise时又需要直接执行promise中的方法，即发生了先执行方法后添加回调的过程，此时需等待then方法绑定两个回调。后才能继续执行方法回调，便可将回调添加到当前js调用栈中执行结束后的任务队列中，由于宏任务较多容易堵塞，则采用了微任务。

## <mark style="color:red;">问题2：Promise 中是如何实现回调函数返回值穿透的？</mark>

首先Promise的执行结果保存在promise的data变量中，然后是.then方法返回值为使用resolved或rejected回调方法新建的一个promise对象，即例如成功则返回new Promise（resolved），将前一个promise的data值赋给新建的promise

## <mark style="color:red;">问题3：Promise 出错后，是怎么通过“冒泡”传递给最后那个捕获？</mark>

promise内部有resolved\_和rejected\_变量保存成功和失败的回调，进入.then（resolved，rejected）时会判断rejected参数是否为函数，若是函数，错误时使用rejected处理错误；若不是，则错误时直接throw错误，一直传递到最后的捕获，若最后没有被捕获，则会报错。可通过监听unhandledrejection事件捕获未处理的promise错误。
