# JS异步

&#x20;Promise是ES6新增的处理异步的东西，叫做“承诺”。

&#x20;Promise的实例有三种状态_**\*：pending（进行中）、resolved（成功）、rejected（失败）\***_

&#x20;Promise对象有then方法，用来做resolved的事情。

&#x20;Promise对象有catch方法，用来做rejected的事情。

&#x20;回答层次维度：

&#x20;_**\*① 用途：回调黑洞的避免（最基本的使用）\***_

&#x20;传统异步函数，必须传入回调函数。

```javascript
function addAfter2000ms(a, b, callback){  
    setTimeout(function(){   
        callback(a + b);  
    }, 2000); 
}  
addAfter2000ms(3, 4, function(result){  
    console.log(result); 
});
```

&#x20;有了Promise之后，就这么封装一下：

```javascript
function addAfter2000ms(a, b){  
    return new Promise((resolve) => {   
        setTimeout(function(){   
         resolve(a + b);   
         }, 2000);  
     }); 
 }  
 addAfter2000ms(3, 4)
 .then(result => {  
     console.log(result); 
 });
```

&#x20;连续.then().then()没有回调黑洞了，变成了火车黑洞。

&#x20;_**\*② 它的小伙伴async/await\***_

&#x20;小伙伴有async/await，它能够等异步执行完毕，再执行后面的语句。

&#x20;它能够让我们以写同步的方法写异步：

```javascript
function addAfter2000ms(a, b){  
    return new Promise((resolve) => {   
        setTimeout(function(){    
            resolve(a + b);   
        }, 2000);  
    }); 
}  
async function main(){  
    const result = await addAfter2000ms(3,4).then(data => data);  
    console.log(result); 
}  
main();
```

&#x20;让我们用写同步的形式写异步函数。

&#x20;_**\*③ 它和Generator的结合。\***_

&#x20;Generator（加星函数、产生器），产生器是产生啥的：迭代器。

```javascript
function addAfter2000ms(a, b){
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(a + b);
        }, 2000);
    });
}

function* gen(){
    yield addAfter2000ms(3, 4);
    yield 'B';
    yield 'C';
    yield 'D';
};

var i = gen();

i.next().value.then(result => {
    console.log(result);
    console.log(i.next());
    console.log(i.next());
    console.log(i.next());
    console.log(i.next());
    console.log(i.next());
    console.log(i.next());
});
```

&#x20;_**\*④ 关于方便处理异步的错误\***_

&#x20;try{}catch(){}可以捕获异步的错误了。

```javascript
async function main(){
    try{
        const result = await addAfter2000ms(3, 4).then(data => data.data);
        console.log(result);
    }catch(e){
        console.log("有错误");
        console.log(e);
    }
}
```

&#x20;原来的异步，不能用try{ }catch(){ }：

```javascript
try{
    setTimeout(function(){
        alert(m);
    }, 2000);
}catch(e){
    alert('补货到了错误');
}
```

&#x20;then 与catch捕获错误的区别？

1）then的第一个函数里抛出了异常，后面的catch能捕获到，而then的第二个函数捕获不到

2）catch只是一个语法糖而己 还是通过then 来处理的

3）then的第二个参数和catch捕获错误信息的时候会就近原则，如果是promise内部报错，reject抛出错误后，then的第二个参数和catch方法都存在的情况下，只有then的第二个参数能捕获到，如果then的第二个参数不存在，则catch方法会捕获到
