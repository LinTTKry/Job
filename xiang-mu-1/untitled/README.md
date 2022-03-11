# express框架

#### &#x20;**Express框架建立在node.js内置的http模块上；**

```java
var http = require("http");
 
var app = http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello world!");
});
 
app.listen(3000, "localhost")
```

#### **关键是http模块的createServer方法，表示生成一个HTTP服务器实例**

#### Express框架的核心是对http模块的再包装

#### Express框架等于在http模块之上，加了一个中间层，而use方法则相当于调用中间件
