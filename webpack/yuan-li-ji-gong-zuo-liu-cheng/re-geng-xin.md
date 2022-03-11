# 热更新

1.当修改了一个或多个文件；&#x20;

2\. 文件系统接收更改并通知webpack；

3\. webpack 重新编译构建一个或多个模块，并通知HMR 服务器进行更新；

4\. HMR Server 使用webSocket 通知HMR runtime 需要更新，HMR 运行时 通过HTTP 请求更新jsonp；&#x20;

5\. HMR 运行时替换更新中的模块，如果确定这些模块无法更新，则触发整 个页面刷新。
