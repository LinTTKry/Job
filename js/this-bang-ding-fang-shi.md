# this绑定方式

默认：在非严格模式下是window，在严格模式下是undefined；

隐式：在函数作为对象的方法调用时，this指向该对象；

显式：利用Bind,call,apply三者来绑定函数的this；

其他：用new创建一个新对象时，this指向新对象
