# 手写发布订阅

```javascript
class EventEmiter {
  constructor(){
    this.events = {}
  }
  on(eventName,callback){
    if(this.events[eventName]) this.events[eventName].push(callback)
    else this.events[eventName] = [callback]
  }
  off(eventName, callback){
    if(this.events[eventName]){
      this.events[eventName].filter(f=>f!=callback)
    }
  }
  emitter(eventName,...rest){
    if(this.events[eventName]){
      this.events[eventName].forEach(x=>x.apply(this,rest))
    }
  }
  once(eventName,callback){
    const fn = function (){
      callback();
      this.off(eventName,fn)
    }
    this.on(eventName,fn)
  }
}
const event = new EventEmiter();
const handle = (...pyload) => console.log(pyload);
event.on("click", handle);
event.emitter("click", 100, 200, 300, 100);
event.off("click", handle);
event.once("dbclick", function() {
console.log("click");
});
event.emitter("dbclick", 100);
```
