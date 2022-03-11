---
description: https://developer.aliyun.com/article/61047
---

# webpack工作流程

Webpack的基本构建流程如下：

校验配置文件 ：读取命令行传入或者webpack.config.js文件，初始化本次构建的配置参数

生成Compiler对象：执行配置文件中的插件实例化语句new MyWebpackPlugin()，为webpack事件流挂上自定义hooks

进入entryOption阶段：webpack开始读取配置的Entries，递归遍历所有的入口文件

run/watch：如果运行在watch模式则执行watch方法，否则执行run方法

compilation：创建Compilation对象回调compilation相关钩子，依次进入每一个入口文件(entry)，使用loader对文件进行编译。通过compilation我可以可以读取到module的resource（资源路径）、loaders（使用的loader）等信息。再将编译好的文件内容使用acorn解析生成AST静态语法树。然后递归、重复的执行这个过程， 所有模块和和依赖分析完成后，执行 compilation 的 seal 方法对每个 chunk 进行整理、优化、封装**webpack\_require**来模拟模块化操作.

emit：所有文件的编译及转化都已经完成，包含了最终输出的资源，我们可以在传入事件回调的compilation.assets上拿到所需数据，其中包括即将输出的资源、代码块Chunk等等信息。

———————————————— 版权声明：本文为CSDN博主「frontend\_frank」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。 原文链接：[https://blog.csdn.net/frontend\_frank/article/details/106205260](https://blog.csdn.net/frontend\_frank/article/details/106205260)
