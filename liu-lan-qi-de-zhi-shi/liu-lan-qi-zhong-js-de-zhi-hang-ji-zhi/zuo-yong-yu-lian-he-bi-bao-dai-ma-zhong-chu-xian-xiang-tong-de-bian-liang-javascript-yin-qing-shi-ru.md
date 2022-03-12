# 作用域链和闭包 ：代码中出现相同的变量，JavaScript引擎是如何选择的？

## ​作用域链

在<mark style="color:red;">**每个执行上下文的变量环境中，都包含了一个外部引用，用来指向外部的执行上下文，我们把这个外部引用称为 outer**</mark>。

```javascript
function bar() {
    console.log(myName)
}
function foo() {
    var myName = "极客邦"
    bar()
}
var myName = "极客时间"
foo()
```

上面那段代码<mark style="color:blue;">**在查找 myName 变量时，如果在当前的变量环境中没有查找到，那么 JavaScript 引擎会继续在 outer 所指向的执行上下文中查找**</mark>。为了直观理解，你可以看下面这张图：![](<../../.gitbook/assets/image (85).png>)

bar 函数和 foo 函数的 outer 都是指向全局上下文的，这也就意味着如果在 bar 函数或者 foo 函数中使用了外部变量，那么 JavaScript 引擎会去全局执行上下文中查找。我们把这个查找的链条就称为作用域链。（<mark style="color:red;">**为什么bar的outer指向的是全局？因为词法作用域**</mark>）

## 词法作用域

词法作用域就是指作用域是<mark style="color:red;">**由代码中函数声明的位置**</mark>来决定的，所以<mark style="color:red;">**词法作用域是静态的作用域**</mark>，通过它就能够预测代码在执行过程中如何查找标识符。

![](<../../.gitbook/assets/image (82).png>)

<mark style="color:blue;">**因为 JavaScript 作用域链是由词法作用域决定的，所以整个词法作用域链的顺序是：foo 函数作用域—>bar 函数作用域—>main 函数作用域—> 全局作用。**</mark>

## 块级作用域中的变量查找

```javascript
function bar() {
    var myName = "极客世界"
    let test1 = 100
    if (1) {
        let myName = "Chrome浏览器"
        console.log(test)
    }
}
function foo() {
    var myName = "极客邦"
    let test = 2
    {
        let test = 3
        bar()
    }
}
var myName = "极客时间"
let myAge = 10
let test = 1
foo()
```

​![](<../../.gitbook/assets/image (67).png>)

现在是执行到 bar 函数的 if 语块之内，需要打印出来变量 test，那么就需要查找到 test 变量的值，其查找过程我已经在上图中使用序号 1、2、3、4、5 标记出来了。

<mark style="color:blue;">**首先是在 bar 函数的**</mark><mark style="color:red;">**执行上下文中**</mark><mark style="color:blue;">**查找，但因为 bar 函数的执行上下文中没有定义 test 变量，所以根据**</mark><mark style="color:red;">**词法作用域**</mark><mark style="color:blue;">**的规则，下一步就在 bar 函数的外部作用域中查找，也就是全局作用域。**</mark>

## 闭包

```javascript
function foo() {
    var myName = "极客时间"
    let test1 = 1
    const test2 = 2
    var innerBar = {
        getName:function(){
            console.log(test1)
            return myName
        },
        setName:function(newName){
            myName = newName
        }
    }
    return innerBar
}
var bar = foo()
bar.setName("极客邦")
bar.getName()
console.log(bar.getName())
```

我们看看当执行到 foo 函数内部的return innerBar这行代码时调用栈的情况：

![](<../../.gitbook/assets/image (78).png>)

innerBar 是一个对象，包含了 getName 和 setName 的两个方法（通常我们把对象内部的函数称为方法）。你可以看到，<mark style="color:blue;">**这两个方法都是在 foo 函数内部定义的，并且这两个方法内部都使用了 myName 和 test1 两个变量。**</mark>

<mark style="color:blue;">**根据词法作用域的规则，内部函数 getName 和 setName 总是可以访问它们的外部函数 foo 中的变量**</mark><mark style="color:blue;">，</mark><mark style="color:blue;">**所以当 innerBar 对象返回给全局变量 bar 时，虽然 foo 函数已经执行结束，但是 getName 和 setName 函数依然可以使用 foo 函数中的变量 myName 和 test1**</mark>。所以当 foo 函数执行完成之后，其整个调用栈的状态如下图所示：

![](<../../.gitbook/assets/image (66).png>)

从上图可以看出，foo 函数执行完成之后，其执行上下文从栈顶弹出了，但是由于返回的 setName 和 getName 方法中使用了 foo 函数内部的变量 myName 和 test1，所以这两个变量依然保存在内存中。这像极了 setName 和 getName 方法背的一个专属背包，无论在哪里调用了 setName 和 getName 方法，它们都会背着这个 foo 函数的专属背包。

<mark style="color:blue;">**之所以是专属背包，是因为除了 setName 和 getName 函数之外，其他任何地方都是无法访问该背包的，我们就可以把这个背包称为 foo 函数的闭包。**</mark>

<mark style="color:red;">**在 JavaScript 中，根据词法作用域的规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为闭包。比如外部函数是 foo，那么这些变量的集合就称为 foo 函数的闭包。**</mark>

当执行到 bar.setName 方法中的myName = "极客邦"这句代码时，JavaScript 引擎会沿着“当前执行上下文–>foo 函数闭包–> 全局执行上下文”的顺序来查找 myName 变量，调用栈状态图：

![](<../../.gitbook/assets/image (81).png>)

setName 的执行上下文中没有 myName 变量，foo 函数的闭包中包含了变量 myName，所以调用 setName 时，会修改 foo 闭包中的 myName 变量的值。同样的流程，当调用 bar.getName 的时候，所访问的变量 myName 也是位于 foo 函数闭包中的。

![](<../../.gitbook/assets/image (87).png>)

当调用 bar.getName 的时候，右边 <mark style="color:blue;">**Scope 项**</mark>就体现出了作用域链的情况：Local 就是当前的 getName 函数的作用域，<mark style="color:red;">**Closure(foo) 是指 foo 函数的闭包**</mark>，最下面的 Global 就是指全局作用域，从“<mark style="color:red;">**Local–>Closure(foo)–>Global**</mark>”就是一个完整的作用域链。

## 闭包是怎么回收的

<mark style="color:blue;">**通常，如果引用闭包的函数是一个全局变量，那么闭包会一直存在直到页面关闭；但如果这个闭包以后不再使用的话，就会造成内存泄漏。**</mark>

如果引用闭包的函数是个局部变量，等函数销毁后，在下次 JavaScript 引擎执行垃圾回收时，判断闭包这块内容如果已经不再被使用了，那么 JavaScript 引擎的垃圾回收器就会回收这块内存。

尽量注意一个原则：如果该闭包会一直使用，那么它可以作为全局变量而存在；但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一个局部变量。

## 思考时间

```javascript
var bar = {
    myName:"time.geekbang.com",
    printName: function () {
        console.log(myName)
    }    
}
function foo() {
    let myName = "极客时间"
    return bar.printName// 当前词法->当前变量->outer全局词法-outer全局变量找到bar
}
let myName = "极客邦"
let _printName = foo()
_printName() //极客邦// 压方法的执行上下入调用栈 // 找当前词法环境（没有）->查找当前变量环境（没有） -> 查找outer全局词法环境（找到了）
bar.printName()//极客邦//当前词法环境（没有）->查找当前变量环境（没有） -> 查找outer全局词法环境（找到了）
```

