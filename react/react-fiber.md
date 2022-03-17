---
description: https://yuchengkai.cn/react/
---

# React Fiber

组件更新归结到底还是 DOM 的更新。对于 React 来说，这部分的内容会分为两个阶段：调和阶段，基本上也就是大家熟知的虚拟 DOM 的 diff 算法提交阶段，也就是将上一个阶段中 diff 出来的内容体现到 DOM 上

## 什么叫reconciler？

计算树的哪些部分已经更改

diff算法是**调和的具体实现。**

## 什么叫render?

使用这些reconciler计算的差异来实际更新呈现的应用程序；

## 为什么需要调度?

JS 和渲染引擎是一个互斥关系。如果 JS 在执行代码，那么渲染引擎工作就会被停止。假如我们有一个很复杂的复合组件需要重新渲染，那么调用栈可能会很长, 调用栈过长，再加上如果中间进行了复杂的操作，就可能导致长时间阻塞渲染引擎带来不好的用户体验。

## Fiber出现的原因？

在react15中，在组件需要更新时，会递归的遍历整棵树来判断需要更新的地方，这种递归的方式弊端在无法中断，必须更新完所有的组件才会停止，如果我们需要更新一些庞大的组件时，更新的过程可能就会阻塞主线程，从而造成用户的交互，动画的更新等等都不能及时响应。

React 的组件更新过程简而言之就是在持续调用函数的一个过程，这样的一个过程会形成一个虚拟的调用栈。假如我们控制这个调用栈的执行，把整个更新任务拆解开来，尽可能地将更新任务放到浏览器空闲的时候去执行，那么就能解决以上的问题

## **React通过什么来解决上述问题即怎么样将**整个更新任务拆分为一个个小的任务，并且能控制这些任务的执行？

这些功能主要是通过两个核心的技术来实现的：

* 新的数据结构 fiber
* 调度器

### 1：fiber----拆分任务的最小颗粒度，可以认为是一个工作单元，执行更新任务的整个流程（不包括渲染）就是在反复寻找工作单元并运行它们

fiber 内部其实存储了很多上下文信息，我们可以把它认为是改进版的虚拟 DOM，它同样也对应了组件实例及 DOM 元素。同时 fiber 也会组成 fiber tree，但是它的结构不再是一个树形，而是一个链表的结构。

[![](https://camo.githubusercontent.com/64a54d5f2779f4ffdf8117790a91f643e3aa106ce978acd9d80c424edf25dadd/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f372f32312f313663313465613231326535383536363f773d33343526683d33393226663d706e6726733d3532313235)](https://camo.githubusercontent.com/64a54d5f2779f4ffdf8117790a91f643e3aa106ce978acd9d80c424edf25dadd/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f372f32312f313663313465613231326535383536363f773d33343526683d33393226663d706e6726733d3532313235)

以下是 fiber 中的一些重要属性：

```
{
  ...
  // 浏览器环境下指 DOM 节点
  stateNode: any,
    
  // 形成列表结构
  return: Fiber | null,
  child: Fiber | null,
  sibling: Fiber | null,

  // 更新相关
  pendingProps: any,  // 新的 props
  memoizedProps: any,  // 旧的 props
  // 存储 setState 中的第一个参数
  updateQueue: UpdateQueue<any> | null, 
  memoizedState: any, // 旧的 state
    
  // 调度相关
  expirationTime: ExpirationTime,  // 任务过期时间
    
  // 大部分情况下每个 fiber 都有一个替身 fiber
  // 在更新过程中，所有的操作都会在替身上完成，当渲染完成后，
  // 替身会代替本身
  alternate: Fiber | null,

  // 先简单认为是更新 DOM 相关的内容
  effectTag: SideEffectTag, // 指这个节点需要进行的 DOM 操作
  // 以下三个属性也会形成一个链表
  nextEffect: Fiber | null, // 下一个需要进行 DOM 操作的节点
  firstEffect: Fiber | null, // 第一个需要进行 DOM 操作的节点
  lastEffect: Fiber | null, // 最后一个需要进行 DOM 操作的节点，同时也可用于恢复任务
  ....
}
```

总的来说，我们可以认为 fiber 就是一个工作单元的数据结构表现，当然它同样也是调用栈中的一个重要组成部分。Fiber 和 fiber 不是同一个概念。前者代表新的调和器，后者代表 fiber node，也可以认为是改进后的虚拟 DOM。

### 2：React 实现调度主要靠两块内容：

1. 计算任务的 expriationTime
2. 实现 requestIdleCallback 的 polyfill 版本

就是通过定时器的方式，来获取每一帧的结束时间。得到每一帧的结束时间以后我们就能判断当下距离结束时间的一个差值。如果还未到结束时间，那么也就意味着我可以继续执行更新任务；如果已经过了结束时间，那么就意味着当前帧已经没有时间给我执行任务了，必须把执行权交还给浏览器，也就是打断任务的执行。另外当开始执行更新任务（也就是寻找工作单元并执行的过程）时，如果有新的更新任务进来，那么调度器就会按照两者的优先级大小来进行决策。如果新的任务优先级小，那么当然继续当下的任务；如果新的任务优先级大，那么会打断任务并开始新的任务

![](https://github.com/LinTTKry/Job/blob/interview/react/broken-reference)

#### 调度的具体实现：

* 首先每个任务都会有各自的优先级，通过当前时间加上优先级所对应的常量我们可以计算出 `expriationTime`，**高优先级的任务会打断低优先级任务**
* 在调度之前，判断当前任务**是否过期**，过期的话无须调度，直接调用 `port.postMessage(undefined)`，这样就能在渲染后马上执行过期任务了
* 如果任务没有过期，就通过 `requestAnimationFrame` 启动定时器，在重绘前调用回调方法
* 在回调方法中我们首先需要**计算每一帧的时间以及下一帧的时间**，然后执行 `port.postMessage(undefined)`
* `channel.port1.onmessage` 会在渲染后被调用，在这个过程中我们首先需要去判断**当前时间是否小于下一帧时间**。如果小于的话就代表我们尚有空余时间去执行任务；如果大于的话就代表当前帧已经没有空闲时间了，这时候我们需要去判断是否有任务过期，**过期的话不管三七二十一还是得去执行这个任务**。如果没有过期的话，那就只能把这个任务丢到下一帧看能不能执行了

#### 1.expriationTime = 当前时间+常量

常量指的是根据不同优先级得出的一个数值，React 内部目前总共有五种优先级，分别为：

```
var ImmediatePriority = 1;
var UserBlockingPriority = 2;
var NormalPriority = 3;
var LowPriority = 4;
var IdlePriority = 5;
```

#### 2.requestIdleCallback

说完了 `expriationTime`，接下来的主题就是实现 `requestIdleCallback` 了，我们首先来了解下该函数的作用

[![](https://camo.githubusercontent.com/22d3cabbe3c09d04abc360dc106f929e54db2be737682c0594beba1973bcc33b/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134352e706e67)](https://camo.githubusercontent.com/22d3cabbe3c09d04abc360dc106f929e54db2be737682c0594beba1973bcc33b/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134352e706e67)

该函数的回调方法会在浏览器的空闲时期依次调用， 可以让我们在事件循环中执行一些任务，并且不会对像动画和用户交互这样延迟敏感的事件产生影响。

在上图中我们也可以发现，该回调方法是在渲染以后才执行的。

因为`requestIdleCallback`不是哪个浏览器都兼容，为了照顾大多数浏览器实现 `requestIdleCallback` 函数的核心只有一点，**如何多次在浏览器空闲时且是渲染后才调用回调方法？**

说到多次执行，那么肯定得使用定时器了。在多种定时器中，唯有 `requestAnimationFrame` 具备一定的精确度，因此 `requestAnimationFrame` 就是当下实现 `requestIdleCallback` 的一个步骤。`requestAnimationFrame` 的回调方法会在每次重绘前执行，另外它还存在一个瑕疵：页面处于后台时该回调函数不会执行，因此我们需要对于这种情况做个补救措施

```
rAFID = requestAnimationFrame(function(timestamp) {
	// cancel the setTimeout
	localClearTimeout(rAFTimeoutID);
	callback(timestamp);
});
rAFTimeoutID = setTimeout(function() {
	// 定时 100 毫秒是算是一个最佳实践
	localCancelAnimationFrame(rAFID);
	callback(getCurrentTime());
}, 100);
```

当 `requestAnimationFrame` 不执行时，会有 `setTimeout` 去补救，两个定时器内部可以互相取消对方。

使用 `requestAnimationFrame` 只完成了多次执行这一步操作，接下来我们需要实现如何知道当前浏览器是否空闲呢？

[![](https://camo.githubusercontent.com/cf9d774e415f15129578fa5fa5fb47e8e90b5fc9f1cdbbaf71a8b3501f26febe/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134372e706e67)](https://camo.githubusercontent.com/cf9d774e415f15129578fa5fa5fb47e8e90b5fc9f1cdbbaf71a8b3501f26febe/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134372e706e67)

大家都知道在一帧当中，浏览器可能会响应用户的交互事件、执行 JS、进行渲染的一系列计算绘制。假设当前我们的浏览器支持 1 秒 60 帧，那么也就是说一帧的时间为 16.6 毫秒。如果以上这些操作超过了 16.6 毫秒，那么就会导致渲染没有完成并出现掉帧的情况，继而影响用户体验；如果以上这些操作没有耗时 16.6 毫秒的话，那么我们就认为当下存在空闲时间让我们可以去执行任务。

因此接下去我们需要计算出当前帧是否还有剩余时间让我们使用。

```
let frameDeadline = 0
let previousFrameTime = 33
let activeFrameTime = 33
let nextFrameTime = performance.now() - frameDeadline + activeFrameTime
if (
	nextFrameTime < activeFrameTime &&
	previousFrameTime < activeFrameTime
) {
	if (nextFrameTime < 8) {
		nextFrameTime = 8;
	}
	activeFrameTime =
		nextFrameTime < previousFrameTime ? previousFrameTime : nextFrameTime;
} else {
	previousFrameTime = nextFrameTime;
}
```

以上这部分代码核心就是得出每一帧所耗时间及下一帧的时间。简单来说就是假设当前时间为 5000，浏览器支持 60 帧，那么 1 帧近似 16 毫秒，那么就会计算出下一帧时间为 5016。

得出下一帧时间以后，我们只需对比当前时间是否小于下一帧时间即可，这样就能清楚地知道是否还有空闲时间去执行任务。

那么最后一步操作就是如何在渲染以后才去执行任务。这里就需要用到事件循环的知识了

[![](https://camo.githubusercontent.com/e8f613c213f9e920b663cf6d55c02d8ec4bce87fb1fe1ac28fe4eb5ca7061226/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134382e706e67)](https://camo.githubusercontent.com/e8f613c213f9e920b663cf6d55c02d8ec4bce87fb1fe1ac28fe4eb5ca7061226/68747470733a2f2f79636b2d313235343236333432322e636f732e61702d7368616e676861692e6d7971636c6f75642e636f6d2f626c6f672f323031392d30362d30342d3135353134382e706e67)

想必大家都知道微任务宏任务的区别，这里就不再赘述这部分的内容了。从上图中我们可以发现，在渲染以后只有宏任务是最先会被执行的，因此宏任务就是我们实现这一步的操作了。

但是生成一个宏任务有很多种方式并且各自也有优先级，那么为了最快地执行任务，我们肯定得选择优先级高的方式。在这里我们选择了 `MessageChannel` 来完成这个任务，不选择 `setImmediate` 的原因是因为兼容性太差。

到这里为止，`requestAnimationFrame` + 计算帧时间及下一帧时间 + `MessageChannel` 就是我们实现 `requestIdleCallback` 的三个关键点了。
