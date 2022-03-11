# 箭头函数与普通函数的区别

* 函数体内的this 对象，就是定义时所在的对象，而不是使用时所在的对 象。
* 不可以使用arguments 对象，该对象在函数体内不存在。如果要用，可 以用rest 参数代替。&#x20;
* 不可以使用yield 命令，因此箭头函数不能用作Generator 函数
* 不可以使用new 命令，因为： 1.没有自己的this，无法调用call，apply。 2.没有prototype 属性，而new 命令在执行时需要将构造函数 的prototype 赋值给新的对象的**proto**
