# 变量提升：JavaScript代码是按顺序执行的吗？

## ​变量提升

指在 JavaScript 代码执行过程中，JavaScript 引擎把<mark style="color:red;">**变量的声明部分和函数的声明部分**</mark>提升到代码开头的“行为”。变量被提升后，会给变量设置默认值，这个默认值就是我们熟悉的 undefined。

![](<../../.gitbook/assets/image (70) (1) (1).png>)![](<../../.gitbook/assets/image (67) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (7) (1).png>)

## JS代码执行流程

变量提升”意味着变量和函数的声明会在物理层面移动到代码的最前面，正如我们所模拟的那样。但，这并不准确。<mark style="color:red;">**实际上变量和函数声明在代码里的位置是不会改变的，而且是在编译阶段被 JavaScript 引擎放入内存中**</mark>。一段 JavaScript 代码在执行之前需要被 JavaScript 引擎编译，编译完成之后，才会进入执行阶段。大致流程：

![](<../../.gitbook/assets/image (81) (1) (1).png>)

### 1. 编译阶段

![](<../../.gitbook/assets/image (62) (1) (1).png>)

输入一段代码，经过编译后，会生成两部分内容：<mark style="color:red;">**执行上下文（Execution context）和可执行代码**</mark>。

执行上下文是 JavaScript 执行一段代码时的运行环境，比如调用一个函数，就会进入这个函数的执行上下文，确定该函数在执行期间用到的诸如 <mark style="color:red;">**this、变量、对象以及函数**</mark>等。总之，<mark style="color:red;">**在执行上下文中存在一个变量环境的对象（Viriable Environment）**</mark>，该对象中保存了变量提升的内容，比如上面代码中的变量 myname 和函数 showName，都保存在该对象中。

简单地把变量环境对象看成是如下结构：

```javascript
VariableEnvironment:
     myname -> undefined, 
     showName ->function : {console.log(myname)
```

### 2. 执行阶段

* 当执行到 showName 函数时，JavaScript 引擎便开始在变量环境对象中查找该函数，由于变量环境对象中存在该函数的引用，所以 JavaScript 引擎便开始执行该函数，并输出“函数 showName 被执行”结果。
* 接下来打印“myname”信息，JavaScript 引擎继续在变量环境对象中查找该对象，由于变量环境存在 myname 变量，并且其值为 undefined，所以这时候就输出 undefined。
* 接下来执行第 3 行，把“极客时间”赋给 myname 变量，赋值后变量环境中的 myname 属性值改变为“极客时间”，变量环境如下所示：

```javascript
VariableEnvironment:
     myname -> "极客时间", 
     showName ->function : {console.log(myname)
```

### 3. 代码中出现相同的变量或者函数怎么办？

```javascript
function showName() {
    console.log('极客邦');
}
showName();
function showName() {
    console.log('极客时间');
}
showName(); 
```

* 首先<mark style="color:red;">**是编译阶段**</mark>。遇到了第一个 showName 函数，会将该函数体存放到变量环境中。接下来是第二个 showName 函数，继续存放至变量环境中，但是变量环境中已经存在一个 showName 函数了，此时，<mark style="color:red;">**第二个 showName 函数会将第一个 showName 函数覆盖掉**</mark>。这样变量环境中就只存在第二个 showName 函数了。
* 接下来是执行阶段。先执行第一个 showName 函数，但由于是从变量环境中查找 showName 函数，而变量环境中只保存了第二个 showName 函数，所以最终调用的是第二个函数，打印的内容是“极客时间”。第二次执行 showName 函数<mark style="color:red;">**也是**</mark>走同样的流程，所以输出的结果也是“<mark style="color:red;">**极客时间**</mark>”。

### 巩固练习

```javascript
showName()
var showName = function() {
    console.log(2)
}
function showName() {
    console.log(1)
}
```

输出1

编译阶段:&#x20;

```javascript
var showName 
function showName(){console.log(1)}
```

执行阶段:&#x20;

```javascript
showName()//输出1 showName=function(){console.log(2)} 
```

//如果后面再有showName执行的话，就输出2因为这时候函数引用已经变了
