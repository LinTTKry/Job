# 如何写一个plugin

Compiler在开始打包时就进行实例化，实例对象里面装着与打包相关的环境和参数，包括options、plugins和loaders等。

compilation对象，它继承于compiler，所以能拿到一切compiler的内容。Compilation表示有关模块资源，已编译资源，Compilation在每次文件变化重新打包时都进行一次实例化

apply方法：当安装这个插件的时候，这个apply方法就会被webpack compiler调用

plugin书写流程：

&#x20;    1：创建一个js命名函数options；

&#x20;    2：在函数原型中绑定一个apply方法，参数为complier；

&#x20;    3：指定一个挂载到webpack自身的事件钩子；

&#x20;    4：处理webpack内部实例的特定数据

&#x20;    5：功能完成后调用webpack提供的回调

```
function HelloWorldPlugin(options) {
  // Setup the plugin instance with options...
}
//
HelloWorldPlugin.prototype.apply = function(compiler) {
  compiler.plugin('done', function() {
    console.log('Hello World!');
  });
};

module.exports = HelloWorldPlugin;
```
