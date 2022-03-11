# 用map实现reduce

```javascript
array.reduce(function(total, currentValue, currentIndex, arr){
}, initialValue);

//total 必需。初始值, 或者计算结束后的返回值。
//currentValue  必需。当前元素
//currentIndex  可选。当前元素的索引
//arr   可选。当前元素所属的数组对象。
//initialValue可选。传递给函数的初始值
```



```javascript
array.map(function(currentValue,index,arr), thisValue)；
//currentValue  必须。当前元素的值
//index 可选。当前元素的索引值
//arr   可选。当前元素属于的数组对象
//thisValue可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。如果省略了 thisValue，或者传入 null、undefined，那么回调函数的 this 为全局对象。
```



```javascript
Array.prototype.myMap = function(fn,thisValue){
    var res = [];
    thisValue = thisValue||[];
    this.reduce(function(pre,cur,index,arr){
            return res.push(fn.call(thisValue,cur,index,arr));
  },[]);
  return res;
}
var arr = [2,3,1,5];
arr.myMap(function(item,index,arr){
    console.log(item,index,arr);
})
```
