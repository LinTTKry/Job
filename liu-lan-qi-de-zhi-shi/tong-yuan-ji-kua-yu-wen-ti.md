# 浏览器同源及跨域问题

## **什么是同源？**

端口，域名，协议需要一致；

## **为什么需要同源？**

同源限制了同一个源的文档或者脚本与另一个源的资源进行交互，这是用与隔离潜在恶意文件的重要安全机制。

## **跨域方式？**

### **1：**jsonp

#### &#x20;     原理：利用了\<script>标签的无同源限制的特性进行操作；

&#x20;              step1: 创建一个script标签；

&#x20;              step2：设置script的src属性，包括url以及一个callback函数；

&#x20;              step3: 定义callback函数，功能为处理接收到的服务器数据

&#x20;              step4: 服务器收到请求后，返回的js代码就是调用callback,将需要返回的数据作为参数传给callback

&#x20;              step5: 浏览器遇到script标签会访问其src属性，将js代码下载并执行；

#### &#x20;    优点：

&#x20;        实现简单，兼容性比较好；

#### &#x20;    缺点：

&#x20;        只支持get请求，因为script标签只能get;

&#x20;        有安全问题，容易受到xss攻击；

&#x20;        需要服务端配合jsonp进行一定程度的修改

#### &#x20;    实现

```javascript
function JSOP ({
    url,
    parmas,
    callbackKey,
    callback
}){
    params = params || {};
    params[callbackKey] = 'jsonpCallback';
    window.jsonpCallback = callback;
    const paramKeys = Object.keys(params);
    const paramStr = paramsKeys.map(key=>
       `${key}=${params[key]}`
    ).join('&');
    const script = document.createElement('script');
    script.setAttribute('src', `${url}?${paramStr}`);
    document.body.appendChild(script)
}
JSONP({
    url:'***',
    param:{
       key:'test'
    },
    callbackKey:'a',
    callback(res){
      console.log(res)
    }
})
```

### 2: cors

* 简单请求：

1. 请求头：head, post, head；
2. 头信息不超过以下内容：Accept, Accept-lauguage,

content-type: application/x-www-urlencoded, multipart/form-data,text/plain

&#x20;    3\. 响应头: Access-control-allow-origin

* 复杂请求：不仅包含通信内容的请求，也包含预请求；

&#x20;    预请求：以options的形式发送，请求头需要包含access-control-request-method/headers, 响应头： access-control-allow-orgin/method/headers

****

