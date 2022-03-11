# 防抖/节流

防抖：执行操作的最后一次

节流：在一定时间内，操作只执行一次

```javascript
//防抖
function debounce(fn,delay){
    let timer = null;
    return (...args)=>{
        clearTimeout(timer);
        timer = setTimeout(()=>{
           fn.apply(this,args)
        },delay)
    }
}
```

```javascript
//节流
function throttle(fn,delay){
     let timer = null;
     return (...args)=>{
        if(timer) return;
        timer = setTimeout(()=>{
          fn.apply(this.args);
          timer = null
        },delay)    
     }
}
```
