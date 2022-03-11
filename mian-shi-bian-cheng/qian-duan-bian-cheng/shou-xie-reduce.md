# 手写reduce

```
function reduce(arr,fn,initialvalue){
  initialvalue = initialvalue === undefined?0:initialvalue;
  let res = initialvalue;
  for(let num of arr){
     res = fn(res,num)
  }
  return res
}
function fn(res,num){
  return res+num
}
```

