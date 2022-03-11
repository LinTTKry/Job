# 柯里化

柯里化是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术。

```javascript
function curry(fn,...args){
   if(args.length>=fn.length){
       return fn(...args)
   }else{
       return (...args2)=>curry(fn,...args,...args2)
   }
}
function add(a, b) {
    return a + b;
}
add(1, 2) // 3
var addCurry = curry(add);
addCurry(1)(2) // 3
```

```javascript
function add(...args){
    const fn = (...args2){
         return add (...args,...args2)
    }
    fn.sumof = ()=>{
         console.log(args.reduce((x,v)=>x+v,0))
    }
    return fn;
}
add(1, 2, 3).sumOf()
add(1)(2)(3).sumOf()  
add(1, 2)(3).sumOf()   
add(4, 5)(1)(2, 3).sumOf()   
add(1, 1)(3)(6).sumOf() 
```
