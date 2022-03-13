# 块级作用域：var缺陷以及为什么要引入let和const？

## ​作用域

作用域是指在程序中定义变量的区域，该位置决定了变量的生命周期。通俗地理解，作用域就是变量与函数的可访问范围，即作用域控制着变量和函数的可见性和生命周期。

在 ES6 之前，ES 的作用域只有两种：

* 全局作用域和函数作用域。全局作用域中的对象在代码中的任何地方都能访问，其生命周期伴随着页面的生命周期。
* 函数作用域就是在函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问。函数执行结束之后，函数内部定义的变量会被销毁。

## 变量提升所带来的问题：

* 变量容易在不被察觉的情况下被覆盖掉

```javascript
var myname = "极客时间"
function showName(){
  console.log(myname);// undefined
  if(0){
   var myname = "极客邦"//声明提前但是执行（赋值）没有
  }
  console.log(myname);//undefined
}
showName()
```

* 本应销毁的变量没有被销毁

```javascript
function foo(){
  for (var i = 0; i < 7; i++) {
  }
  console.log(i); //7
}
foo()
```

在 for 循环结束之后，i 就已经被销毁了，但是在 JavaScript 代码中，i 的值并未被销毁，所以最后打印出来的是 7。这同样也是由变量提升而导致的，在创建执行上下文阶段，变量 i 就已经被提升了，所以当 for 循环结束之后，变量 i 并没有被销毁。

## ES6 是如何解决变量提升带来的缺陷

使用 let 关键字声明的变量是可以被改变的，而使用 const 声明的变量其值是不可以被改变的.

```javascript
function varTest() {
  var x = 1;
  if (true) {
    var x = 2;  // 同样的变量!
    console.log(x);  // 2
  }
  console.log(x);  // 2
}
```

从执行上下文的变量环境中可以看出，最终<mark style="color:blue;">**只生成了一个变量 x**</mark>，函数体内所有对 x 的赋值操作都会直接改变变量环境中的 x 值。

```javascript
function letTest() {
  let x = 1;
  if (true) {
    let x = 2;  // 不同的变量
    console.log(x);  // 2
  }
  console.log(x);  // 1
}
```

这是因为 let 关键字是支持块级作用域的，所以在编译阶段，JavaScript 引擎并不会把 if 块中通过 let 声明的变量存放到变量环境中，这也就意味着<mark style="color:blue;">**在 if 块通过 let 声明的关键字，并不会提升到全函数可见.**</mark>

## JavaScript 是如何支持块级作用域的

```javascript
function foo(){
    var a = 1
    let b = 2
    {
      let b = 3
      var c = 4
      let d = 5
      console.log(a)
      console.log(b)
    }
    console.log(b) 
    console.log(c)
    console.log(d)
}   
foo()
```

### ​第一步是编译并创建执行上下文

![](<../../../.gitbook/assets/image (7).png>)

通过上图，我们可以得出以下结论：

* 函数内部通过 <mark style="color:blue;">**var**</mark> 声明的变量，在编译阶段全都被存放到<mark style="color:blue;">**变量环境**</mark>里面了。
* 通过 <mark style="color:blue;">**let 声明的变量**</mark>，在编译阶段会被存放到<mark style="color:blue;">**词法环境**</mark>（Lexical Environment）中。
* 在<mark style="color:blue;">**函数的作用域块内部，通过 let 声明的变量并没有被存放到词法环境**</mark>中。

### 第二步继续执行代码

![](<../../../.gitbook/assets/image (77).png>)

当进入函数的作用域块时，作用域块中通过 let 声明的变量，会被存放在词法环境的一个单独的区域中，这个区域中的变量并不影响作用域块外面的变量，比如<mark style="color:blue;">**在作用域外面声明了变量 b，在该作用域块内部也声明了变量 b，当执行到作用域内部时，它们都是独立的存在。**</mark>

<mark style="color:red;">**在**</mark><mark style="color:red;"><mark style="color:orange;">**词**<mark style="color:orange;"></mark><mark style="color:red;">**法环境内部，维护了一个小型栈结构**</mark>，栈底是函数最外层的变量，进入一个作用域块后，就会把该作用域块内部的变量压到栈顶；当作用域执行完成之后，该作用域的信息就会从栈顶弹出，这就是词法环境的结构。

当执行到作用域块中的console.log(a)这行代码时，就需要在词法环境和变量环境中查找变量 a 的值了，具体查找方式是：<mark style="color:red;">**沿着词法环境的栈顶向下查询，如果在词法环境中的某个块中查找到了，就直接返回给 JavaScript 引擎，如果没有查找到，那么继续在变量环境中查找。**</mark>

<mark style="color:red;">****</mark>![](<../../../.gitbook/assets/image (73).png>)<mark style="color:red;">****</mark>

当作用域块执行结束之后，其内部定义的变量就会从词法环境的栈顶弹出，最终执行上下文如下图所示：

![](<../../../.gitbook/assets/image (63).png>)

<mark style="color:red;">**块级作用域就是通过词法环境的栈结构来实现的，而变量提升是通过变量环境来实现，通过这两者的结合，JavaScript 引擎也就同时支持了变量提升和块级作用域了。**</mark>

## 思考题

```javascript
let myname= '极客时间'
{
  console.log(myname) 
  let myname= '极客邦'
}
```

【最终打印结果】：VM6277:3 Uncaught ReferenceError: Cannot access 'myname' before initialization 【分析原因】：在块作用域内，<mark style="color:red;">**let声明的变量被提升，但变量只是创建被提升，初始化并没有被提升，在初始化之前使用变量，就会形成一个暂时性死区**</mark>。

【拓展】&#x20;

<mark style="color:red;">**var的创建和初始化被提升，赋值不会被提升。**</mark>

&#x20;<mark style="color:red;">**let的创建被提升，初始化和赋值不会被提升。**</mark>&#x20;

<mark style="color:red;">**function的创建、初始化和赋值均会被提升。**</mark> (简单点，let/ const不存在变量提升)
