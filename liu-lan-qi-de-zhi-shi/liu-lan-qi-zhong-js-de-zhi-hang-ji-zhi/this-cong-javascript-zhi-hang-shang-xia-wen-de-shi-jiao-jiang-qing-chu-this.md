# this：从JavaScript执行上下文的视角讲清楚this

## ​JavaScript 中的 this 是什么

​![](<../../.gitbook/assets/image (160).png>)

<mark style="color:red;">**this 是和执行上下文绑定的**</mark>**。**<mark style="color:blue;">**执行上下文主要分为三种——全局执行上下文、函数执行上下文和 eval 执行上下文，所以对应的 this 也只有这三种——全局执行上下文中的 this、函数中的 this 和 eval 中的 this。**</mark>

## 全局执行上下文中的 this

全局执行上下文中的 this 是指向 window 对象的。<mark style="color:red;">**这也是 this 和作用域链的唯一交点，作用域链的最底端包含了 window 对象**</mark>，全局执行上下文中的 this 也是指向 window 对象。

## 函数执行上下文中的 this

在默认情况下调用一个函数，其执行上下文中的 this 也是指向 window 对象的。三种方式来设置函数执行上下文中的 this 值。

1. 通过函数的 call/ bind/ apply方法设置
2. 通过对象调用方法设置
3. 通过构造函数中设置

```javascript
function CreateObj(){
  this.name = "极客时间"
}
var myObj = new CreateObj()
```

当执行 new CreateObj() 的时候，JavaScript 引擎做了如下四件事：

* 首先创建了一个空对象 tempObj；
* 接着调用 CreateObj.call 方法，并将 tempObj 作为 call 方法的参数，这样当 CreateObj 的执行上下文创建时，它的 this 就指向了 tempObj 对象；
* 然后执行 CreateObj 函数，此时的 CreateObj 函数执行上下文中的 this 指向了 tempObj 对象；
* 最后返回 tempObj 对象。

如：

```javascript
var tempObj = {}
CreateObj.call(tempObj)
return tempObj
```

## this 的设计缺陷以及应对方案

* 嵌套函数中的 this 不会从外层函数中继承

```javascript
var myObj = {
  name : "极客时间", 
  showThis: function(){
    console.log(this)
    function bar(){console.log(this)}
    bar()
  }
}
myObj.showThis()
```

函数 bar 中的 this 指向的是全局 window 对象，而函数 showThis 中的 this 指向的是 myObj 对象。你可以通过一个小技巧来解决这个问题，比如在 showThis 函数中声明一个变量 self 用来保存 this，然后在 bar 函数中使用 self，<mark style="color:red;">**这个方法的的本质是把 this 体系转换为了作用域的体系**</mark>,代码如下所示：

```javascript
var myObj = {
  name : "极客时间", 
  showThis: function(){
    console.log(this)
    var self = this
    function bar(){
      self.name = "极客邦"
    }
    bar()
  }
}
myObj.showThis()
console.log(myObj.name)
console.log(window.name)
```

其实，你也可以使用 ES6 中的箭头函数来解决这个问题:

```javascript
var myObj = {
  name : "极客时间", 
  showThis: function(){
    console.log(this)
    var bar = ()=>{
      this.name = "极客邦"
      console.log(this)
    }
    bar()
  }
}
myObj.showThis()
console.log(myObj.name)
console.log(window.name)
```

<mark style="color:red;">**这是因为 ES6 中的箭头函数并不会创建其自身的执行上下文，所以箭头函数中的 this 取决于它的外部函数。**</mark>

* 普通函数中的 this 默认指向全局对象 window

在<mark style="color:blue;">**默认情况下调用一个函数，其执行上下文中的 this 是默认指向全局对象 window 的**</mark>**。**不过这个设计也是一种缺陷，因为在实际工作中，我们并不希望函数执行上下文中的 this 默认指向全局对象，<mark style="color:blue;">**因为这样会打破数据的边界，造成一些误操作**</mark>**。**如果要让函数执行上下文中的 this 指向某个对象，最好的方式是通过 call 方法来显示调用。这个问题可以通过设置 JavaScript 的“严格模式”来解决。<mark style="color:blue;">**在严格模式下，默认执行一个函数，其函数的执行上下文中的 this 值是 undefined，这就解决上面的问题了。**</mark>
