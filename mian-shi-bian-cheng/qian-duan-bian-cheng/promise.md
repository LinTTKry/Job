# promise

```javascript
function MyPromise(constructor){
    const self = this;
    self.status='pending';
    self.data = undefined;
    self.callbacks = [];
    function resolve(value){
      if(self.status !== 'pending') return;
      self.status = 'fulfilled';
      self.data = value;
      setTimeout(()=>{
         self.callbacks(x=>x.onResolved(value))
      })
    }
    function reject(reason){
      if(self.status !== 'pending') return;
        self.status = 'rejected';
        self.data = reason;
        setTimeout(()=>{
           self.callbacks(x=>x.onRejected(reason))
        })
    }
    try{
      constructor(resolve,reject)
    }catch(e){
       reject(e)
    }
}
MyPromise.prototype.then = function(onResolved, onRejected){
    const self= this;
    onResolved= onResolved instanceof 'Function' ? onResolved:value=>value;
    onRejected= onRejected instanceof 'Function' ? onRejected:reason=>{throw reason};
    return new  MyPromise((resolve,reject)=>{
         function handle = (callback)=>{
            try{
               const x = callback(self.data);
               if(x instanceof MyPromise){
                  x.then(resolve,reject)
               }else{
                  resolve(x)
               }
            }catch(e){
               reject(e)
            }
         }
         if(self.status === 'fulfilled'){
            setTimeout(()=>{
               handle(onResolved)
            })
         }
         if(self.status === 'rejected'){
            setTimeout(()=>{
               handle(onRejected)
            })
         }
         if(self.status === 'pending'){
            self.callbacks.push({
              onResolved{handle(onResolved(value))},
              onRejected{handle(onRejected(reason))}
            }
         }  
    })

}
MyPromise.prototype.catch = function (reject){
   return this.then(null,reject)
}

Mypromise.resolve = funtion(value){
   return new MyPromise((resolve,reject)=>{
       if(value instanceof MyPromise){
          value.then(resolve,reject)
       }else{
          resolve(value)
       }
   })
}
MyPromise.reject= funtion(reason){
   return new MyPromise((resolve,reject)=>{
      reject(reason)
   })
}
MyPromise.all = function (promises){
    return new MyPromise((resolve,reject)=>{
        const len = promises.length;
        let resolveNum = 0;
        let values = [len];
        for(let i = 0; i<len;i++){
            MyPromise.resolve(promises[i]).then(value=>{
               values[i] = value;
               resolveNum ++;
               if(resolveNum === len) resolve(values);
            },reason=>{
               reject(reason)
            })
        }          
    })
}
MyPromise.race = function(promises){
   return new MyPromise((resolve,reject)=>{
        for(let i = 0; i< promises.length; i++){
           MyPromise.resolve(promises[i]).then(value=>{
             resolve(value)
           },reason=>{reject(reason)})     
        }  
   })
}
```
