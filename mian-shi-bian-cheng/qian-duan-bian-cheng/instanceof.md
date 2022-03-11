# instanceof

```javascript
function instance_of(L,R){
   let O = R.prototype;
   L = L.__proto__;
   while(L!=null){
      if(L===O) return true;
      L = L.__proto__;
   }
   return false
}
```
