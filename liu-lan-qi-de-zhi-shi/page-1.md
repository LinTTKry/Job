# 栈空间和堆空间：数据是如何存储的？

## ​JavaScript 是什么类型的语言

在声明变量之前需要先定义变量类型。我们把这种在<mark style="color:blue;">**使用之前就需要确认其变量数据类型的称为静态语言**</mark>。相反地，我们把在运行过程中需要检查数据类型的语言称为动态语言。

C 编译器会把 int 型的变量悄悄转换为 bool 型的变量，我们通常把这种偷偷转换的操作称为隐式类型转换。而<mark style="color:blue;">**支持隐式类型转换的语言称为弱类型语言**</mark>，不支持隐式类型转换的语言称为强类型语言。

![](<../.gitbook/assets/image (88).png>)

<mark style="color:red;">**JavaScript 是一种弱类型的、动态的语言。**</mark>

<mark style="color:red;">****</mark>![](<../.gitbook/assets/image (68).png>)<mark style="color:red;">****</mark>

第一点，使用 typeof 检测 Null 类型时，返回的是 Object。这是当初 JavaScript 语言的一个 Bug，一直保留至今，之所以一直没修改过来，主要是为了兼容老的代码。

第二点，Object 类型比较特殊，它是由上述 7 种类型组成的一个包含了 key-value 对的数据类型。

第三点，我们把前面的 7 种数据类型称为原始类型，把最后一个对象类型称为引用类型，之所以把它们区分为两种不同的类型，是因为它们在内存中存放的位置不一样。

## 内存空间

![](<../.gitbook/assets/image (77).png>)

在 JavaScript 的执行过程中， 主要有三种类型内存空间，分别是代码空间、栈空间和堆空间。

### 栈空间和堆空间

```javascript
function foo(){
    var a = "极客时间"
    var b = a
    var c = {name:"极客时间"}
    var d = c
}
foo()
```

![](<../.gitbook/assets/image (78).png>)

JavaScript 引擎并不是直接将该对象存放到变量环境中，而是将它分配到堆空间里面，<mark style="color:blue;">**分配后该对象会有一个在“堆”中的地址，然后再将该数据的地址写进 c 的变量值。**</mark><mark style="color:red;">**原始类型的数据值都是直接保存在“栈”中的，引用类型的值是存放在“堆”中的。**</mark>

为什么一定要分“堆”和“栈”两个存储空间呢？所有数据直接存放在“栈”中不就可以了吗？

答案是不可以的。这是因为 <mark style="color:blue;">JavaScript 引擎需要用栈来维护程序执行期间上下文的状态</mark>，<mark style="color:blue;">如果栈空间大了话，所有的数据都存放在栈空间里面，那么会影响到上下文切换的效率，</mark>进而又影响到整个程序的执行效率。

![](<../.gitbook/assets/image (63).png>)

通常情况下，栈空间都不会设置太大，主要用来存放一些原始类型的小数据。而引用类型的数据占用的空间都比较大，所以这一类数据会被存放到堆中，堆空间很大，能存放很多大的数据，不过缺点是分配内存和回收内存都会占用一定的时间。

在 JavaScript 中，原始类型的赋值会完整复制变量值，而引用类型的赋值是复制引用地址。

<mark style="color:red;">****</mark>

