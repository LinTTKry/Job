---
description: >-
  中间件（middleware）就是处理HTTP请求的函数，用来完成各种特定的任务，比如检查用户是否登录、分析数据、以及其他在需要最终将数据发送给用户之前完成的任务。它最大的特点就是，一个中间件处理完，再传递给下一个中间件
---

# 中间件

## 定义

中间件就是一堆方法，可以接收客户端发来的请求，可以对请求作出响应，也可以将请求交给下一个中间件处理；

![](<../../.gitbook/assets/image (33).png>)

## 组成

中间方法加请求处理函数；

```
app.get('路径','请求处理函数')
```

## 使用规则

可以针对同一个请求设置多个中间件，对同一个请求进行多次处理；

默认情况下，请求从上到下依次匹配中间件，一旦匹配成功，终止匹配；

可以调用next方法将请求的控制权交给下一个中间件，直到遇到结束请求的中间件。

```javascript
// 引入 express 框架
const express = require('express')

// 创建网站服务器
const app = express();

app.get('/request', (req, res, next) => {
    req.name = 'zhangsan';
    next(); // 向下执行
})

app.get('/request', (req, res)=> {
    res.send(req.name);
})

// 监听端口
app.listen(3000);
console.log('网站服务器启动成功');
```

## 分类

* 应用级中间件
* 路由级中间件
* 错误处理中间件
* 内置中间件
* 第三方中间件

## 应用级中间件

应用级中间键绑定到`app对象`使用`app.use`和`app.METHOD()-需要处理http请求的方法，例如GET、PUT、POST`

```javascript
var app = express()

// 没有挂载路径的中间件，应用中的每个请求都会执行该中间件
app.use((req,res,next) => {
    console.log('Time',Dtae.now());
    next(); // 传递request对象给下一个中间件
})

// 挂载至/user/:id的中间件，任何执行/user/:id的请求都会执行它
app.use('/use/:id',(req,res,next) => {
    console.log('Request Type',req.method);
    next();
})

// 路由和句柄函数（中间件系统），处理指向/user/:id的GET请求
app.get('/user/:id',(req,res,next)=>{
    console.log('USER');
})
```

## 路由级中间件

路由级中间件和应用级中间件类似，只不过是它绑定对象为`express.Router()`

```
var router = express.Router()
```

路由级使用`router.use()`或`router.VERB()`加载

```javascript
var app = express()
var router = express.Router()
// 没有挂载路径的中间件，通过该路由的每个请求都会执行该中间件
router.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
})

// 一个中间件，显示任何指向/user/:id的HTTP请求的信息
router.use('/user/:id',(req,res,next)=>{
    console.log('Request URL',req.originalUrl)
    next()
},(req,res,next)=>{
    console.log('Request Type',req.method)
    next()
})

// 一个中间件栈，处理指向/user/:id的GET请求
router.get('/user/:id',(req,res,next)=>{
    if(req.params.id == 0) next('router')
    else next()
},(req,res,next)=>{
    res.render('regular')
})

// 处理/user/:id，渲染一个特殊页面
router.get('user/:id',(req,res,next)=>{
    console.log(req.params.id)
    res.render('special')
})

// 将路由挂载至应用
app.use('/',router)
```

## 内置中间件

通过网络发送静态文件对 Web 应用来说是一个常见的需求场景。这些资源通常包括图片资源、CSS 文件以及静态 HTML 文件。但是一个简单的文件发送行为其实代码量很大，因为需要检查大量的边界情况以及性能问题的考量。而 Express 内置的 _express.static_ 模块能最大程度简化工作。

假设现在需要对 _public_ 文件夹提供文件服务，只需通过静态文件中间件我们就能极大压缩代码量：

```javascript
var express = require("express");
var path = require("path");
var http = require("http");
var app = express();
var publicPath = path.resolve(__dirname, "public"); 
app.use(express.static(publicPath)); 
app.use(function(request, response) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Looks like you didn't find a static file.");
});
http.createServer(app).listen(3000);
```

## 第三方中间件 <a href="#di-san-fang-zhong-jian-jian" id="di-san-fang-zhong-jian-jian"></a>

通过使用第三方中间件从而为Express应用增加更多的功能\
安装所需功能的node模块，并在应用中加载，可以在应用级中加载，也可以在路由级中加载

```javascript
$ npm install cookie-parser
```

```javascript
var express = require('express')
var app = express()
var cookieParser = require('cookie-parser')

// 加载用于解析cookie的中间件
app.use(cookieParser())
// 声明使用解析post请求的中间件
app.use(express.urlencoded({extended: true})) // 请求体参数是: name=tom&pwd=123
app.use(express.json()) // 请求体参数是json结构: {name: tom, pwd: 123}
```

## 错误处理中间件 <a href="#cuo-wu-chu-li-zhong-jian-jian" id="cuo-wu-chu-li-zhong-jian-jian"></a>

> 错误处理中间件有四个参数,定义错误处理中间件必须使用这四个参数。即使不需要next对象，也必须在参数中声明它，否者中间件会识别为一个常规中间件，不能处理错误

举个栗子：

```javascript
app.use((err,req,res,next)=>{
    console.error(err.stack)
    res.status(500).send('Something broke')
})
```

中间件返回的响应是随意的，可以响应一个 HTML 错误页面、一句简单的话、一个 JSON 字符串，或者其他任何您想要的东西。

所以你可能想要像处理常规中间件那样，定义多个错误处理中间件\
,比如您想为使用 XHR 的请求定义一个，还想为没有使用的定义一个，那么：

```
app.use(logErrors)
app.use(clientErrorHandler)
app.use(errorHandler)
```

`logErrors` 将请求和错误信息写入标准错误输出、日志或者类似服务

```
function logErrors(err,req,res,next){
    console.error(err.stack)
    next(err)
}
```

`clientErrorHandler` 定义如下(这里将错误直接传给了next)

```
function clientErrorHandler(err,req,res,next){
    if(req.xhr){
        res.status(500).send({error:'Something blew up!'})
    }else{
        next(err)
    }
}
```

`errorHandler` 捕获所有错误

```
function errorHandler(err,req,res,next){
    res.status(500)
    res.render('error',{error:err})
}

```

## app 中间件和 router 中间件的区别

Express 是一个非常轻量级的框架，源代码很短。你问的这个问题文档里是看不明白的，只能阅读源码才能搞清楚。

简单的来说，Express 框架主要是在 Node.js 自带的 http/https 模块的基础上做了一层轻量级包装构造出来的。

当你调用 express() 方法时，就创建了一个 Application。这一点相信你已经了解。但是你可能不清楚的是，事实上这个 Application 也只不过是对 Router 的一个轻量级封装而已。

每个 Application 内部都创建了一个 Router，大部分对 Application 的操作实际上都被重定向到了这个内部的 Router 上而已。而 Application 所做的，只不过是在这个 Router 的基础上添加了一些额外的便捷 API 而已。

举个例子，当你调用 app.get(...) 或者 app.use(...) 的时候，实际上真正调用的是 app.\_router.get(...) 或者 app.\_router.use(...)。

所以，Application 和 Router 的区别便很清楚了，Application 是 Router 的一个“包装”，而 Router 才是“核心”
