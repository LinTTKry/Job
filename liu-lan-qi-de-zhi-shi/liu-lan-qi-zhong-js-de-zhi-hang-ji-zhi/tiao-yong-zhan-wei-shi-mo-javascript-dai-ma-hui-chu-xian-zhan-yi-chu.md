# 调用栈：为什么JavaScript代码会出现栈溢出？

​哪些情况下代码才算是“一段”代码，才会在执行之前就进行编译并创建执行上下文。一般说来，有这么三种情况：

1. 当 JavaScript 执行<mark style="color:blue;">**全局代码**</mark>的时候，会编译全局代码并创建全局执行上下文，<mark style="color:red;">**而且在整个页面的生存周期内，全局执行上下文只有一份**</mark>。
2. 当调用一个<mark style="color:blue;">**函数的时候，函数体内的代码会被编译，并创建函数执行上下文**</mark>，一般情况下，<mark style="color:red;">**函数执行结束之后，创建的函数执行上下文会被销毁**</mark>。
3. 当使用 eval 函数的时候，eval 的代码也会被编译，并创建执行上下文。

## 什么是函数调用

```javascript
var a = 2
function add(){
  var b = 10
  return  a+b
}
add()
```

![](<../../.gitbook/assets/image (58).png>)

<mark style="color:blue;">**执行上下文准备好之后，便开始执行全局代码，当执行到 add 这儿时，JavaScript 判断这是一个函数调用，那么将执行以下操作**</mark>：

* 首先，从全局执行上下文中，取出 add 函数代码。
* 其次，<mark style="color:blue;">**对 add 函数的这段代码进行编译，并创建该函数的执行上下文和可执行代码**</mark>。
* 最后，执行代码，输出结果。

![](<../../.gitbook/assets/image (69).png>)

当执行到 add 函数的时候，我们就有了两个执行上下文了——<mark style="color:blue;">**全局执行上下文和 add 函数的执行上下文**</mark>。也就是说在执行 JavaScript 时，可能会存在多个执行上下文，那么 <mark style="color:blue;">**JavaScript 引擎是通过一种叫栈（后进先出）的数据结构来管理的管理这些执行上下文的**</mark>。

## 什么是 JavaScript 的调用栈

<mark style="color:red;">**JavaScript 引擎会将执行上下文压入栈中，通常把这种用来管理执行上下文的栈称为执行上下文栈，又称调用栈**</mark>

```javascript
var a = 2
function add(b,c){
  return b+c
}
function addAll(b,c){
  var d = 10
  result = add(b,c)
  return  a+result+d
}
addAll(3,6)
```

* 创建全局上下文，并将其压入栈底，执行a=2

![](<../../.gitbook/assets/image (75).png>)![](<../../.gitbook/assets/image (76).png>)

* 调用 addAll 函数，同样会为其创建执行上下文，并执行d=10

![](<../../.gitbook/assets/image (88) (1).png>)

* 执行到 add 函数调用语句时，同样会为其创建执行上下文，并将其压入调用栈![](<../../.gitbook/assets/image (84).png>)
* 当 add 函数返回时，该函数的执行上下文就会从栈顶弹出，并将 result 的值设置为 add 函数的返回值，也就是 9。如下图所示：

![](<../../.gitbook/assets/image (66) (1).png>)

* 紧接着 addAll 执行最后一个相加操作后并返回，addAll 的执行上下文也会从栈顶部弹出，此时调用栈中就只剩下全局上下文了。最终如下图所示：

![](<../../.gitbook/assets/image (71).png>)



## 如何利用浏览器查看调用栈的信息

你可以打开“开发者工具”，点击“<mark style="color:blue;">**Source**</mark>”标签，选择<mark style="color:blue;">**js代码**</mark>的页面，然后在第 3 行加上<mark style="color:blue;">**断点**</mark>，并<mark style="color:blue;">**刷新页面**</mark>。你可以看到执行到 add 函数时，执行流程就暂停了，这时可以通过右边“<mark style="color:blue;">**call stack**</mark>”来查看当前的调用栈的情况，如下图：

![](<../../.gitbook/assets/image (81) (1).png>)

从图中可以看出，右边的“call stack”下面显示出来了函数的调用关系：栈的最底部是 a<mark style="color:blue;">**nonymous，也就是全局的函数入口**</mark>；中间是 addAll 函数；顶部是 add 函数。这就清晰地反映了函数的调用关系，所以在分析复杂结构代码，或者检查 Bug 时，调用栈都是非常有用的。除了通过断点来查看调用栈，你还可以使用 console.trace() 来输出当前的函数调用关系，比如在示例代码中的 add 函数里面加上了console.trace()，你就可以看到控制台输出的结果：

![](<../../.gitbook/assets/image (67) (1).png>)

## 栈溢出

调用栈是有大小的，<mark style="color:blue;">**当入栈的执行上下文超过一定数目，JavaScript 引擎就会报错，我们把这种错误叫做栈溢出**</mark>

## 总结

* 每调用一个函数，JavaScript 引擎会为其创建执行上下文，并把该执行上下文压入调用栈，然后 JavaScript 引擎开始执行函数代码。
* 如果在一个函数 A 中调用了另外一个函数 B，那么 JavaScript 引擎会为 B 函数创建执行上下文，并将 B 函数的执行上下文压入栈顶。
* 当前函数执行完毕后，JavaScript 引擎会将该函数的执行上下文弹出栈。
* 当分配的调用栈空间被占满时，会引发“堆栈溢出”问题。
