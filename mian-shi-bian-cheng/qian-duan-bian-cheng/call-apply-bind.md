---
description: >-
  call和apply是绑定this并立即调用，其中第一个参数都为this,其余参数call是以列表形式传递即2,3,4，apply是将其他参数放在一个数组中，一并传递；bind不同于call与apply虽然也是绑定this,但bind却不是立即调用，而是返回绑定this后的函数
---

# call/apply/bind

```javascript
Function.prototype.mycall = function (context=window,...args) {
  if(this === Function.prototype){
    return undefined;//防止Function.prototype.myapply直接调用
  }
  const fn = Symbol();
  context[fn] = this;
  const res = context[fn](...args);
  delete context[fn];
  return res
}
function add (a,b){
    return this.c+a+b;
}
var obj = {c:1}
console.log(add.mycall(obj,1,2))
```

```javascript
Function.prototype.myapply = function (context = window, args){
  if(this === Function.prototype){
    return undefined;//防止Function.prototype.myapply直接调用
  }
   const fn = Symbol();
   context[fn] = this;
   let res = context[fn](...args);
   delete context[fn];
   return res
}
function add (a,b){
  return this.c+a+b;
}
var obj = {c:1}
console.log(add.myapply(obj,[1,2]))
```

```javascript
Function.prototype.mybind = function (obj) {
  // var args = Array.prototype.slice.call(arguments);
  var args = Array.prototype.slice.call(arguments, 1);
  var that = this;
  var fn = function () {};
  var f = function (){
    var arr =args.concat(Array.prototype.slice.call(arguments))
    if(this instanceof fn)  that.apply(this,arr)
    else  that.apply(obj,arr)
  }
  fn.prototype = that.prototype;
  f.prototype = new fn();
  return f;
}
function person(a,b,c){
  console.log(this.name);
  console.log(a,b,c)
}
person.prototype.collection = '收';
var egg = {name:'蛋'};
var bibi = person.mybind(egg,'点赞','投币')
var b = new bibi('充电')
console.log(b.collection)
```
