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

