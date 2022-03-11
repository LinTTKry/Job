# promise并发控制



#### promise并发控制

实现一个批量请求函数 multiRequest(urls, maxNum)，要求如下：

* 要求最大并发数 maxNum
* 每当有一个请求返回，就留下一个空位，可以增加新的请求
* 所有请求完成后，结果按照 urls 里面的顺序依次打出

```javascript
function multiRequest(urls = [], maxNum){
	// 请求总数量
	const len = urls.length
	// 根据请求数量创建一个数组来保存请求的结果
	const result = new Array(len).fill(false)
	// 当前完成的数量
	let count = 0

	return new Promise((resolve, reject) => {
		// 请求maxNum个
		while (count < maxNum) {
			next()
		}

		function next(){
			let current = count++
			// 处理边界条件
			if (current >= len) {
				// 请求全部完成就将promise置为成功状态, 然后将result作为promise值返回
				!result.includes(false) && resolve(result)
				return
			}
			const url = urls[current]
			console.log(`开始 ${current}`, new Date().toLocaleString())
			fetch(url)
				.then((res) => {
					// 保存请求结果
					result[current] = res
					console.log(`完成 ${current}`, new Date().toLocaleString())
					// 请求没有全部完成, 就递归
					if (current < len) {
						next()
					}
				})
				.catch((err) => {
					console.log(`结束 ${current}`, new Date().toLocaleString())
					result[current] = err
					// 请求没有全部完成, 就递归
					if (current < len) {
						next()
					}
				})
		}
	})
}
```

```javascript
  class Scheduler{
    constructor(count){
      this.count = count;
      this.runningNumber = 0;
      this.pendtask= [];
    }
    add (task){
      return new Promise((resolve,reject)=>{
        if(this.runningNumber<this.count){
          this.runningNumber++;
          task().then(res=>{
            resolve(res);
            this.runningNumber--;
            this.schedule();
          })
        }else{
          this.pendtask.push({
            task,
            resolve,
            reject
          })
        }
      })
    }
    schedule(){
      if(this.pendtask.length===0)return;
      const obj = this.pendtask.shift();
      obj.task().then(res=>{
        obj.resolve(res);
        this.schedule()
      })
    }
  }
  // Usage
  const timeout = (time) => new Promise(resolve => {
    setTimeout(resolve, time)
  })
  const scheduler = new Scheduler(2)
  const addTask = (time, order) => {
    scheduler.add(() => timeout(time))
      .then(() => console.log(order))
  }

addTask(1000, '1')
addTask(500, '2')
addTask(300, '3')
addTask(400, '4')
```
