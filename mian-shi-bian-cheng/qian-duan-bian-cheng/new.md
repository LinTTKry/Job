# new

1. 新建对象；
2. 将对象的原型指向构造函数的原理，实现原型链继承
3. 将构造函数的this绑到对象上，实现构造函数继承

```javascript
function createObj(con,...args){
   let obj = Object.create();
   Object.setPrototypeOf(obj,con);
   let res = con.apply(obj,args)
   return res = res instanceof ?res:obj
}
```
