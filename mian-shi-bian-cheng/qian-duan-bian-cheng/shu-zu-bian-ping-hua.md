# 数组扁平化

```javascript
function flat(arr){
  return arr.reduce((x,v)=>
     Array.isArray(v)?
       x.concat(flat(v)):
       x.concat(v)
  ,[])
}
arr = [1,2,3,4,5,[1,3,4,5],[1,3]]
console.log(flat(arr));
```
