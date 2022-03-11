# loader，plugin，chunk

**loader**用于加载某些资源文件，因为webpack本身只能打包CommonJS规范的js文件，对于其他资源，例如css，图片等，是没有办法加载的，这就需要对应的loader将资源转换 plugin用于扩展webpack的功能，直接作用于webpack，loader只专注于转换文件，而plugin不仅局限于资源加载

Loader只能处理单一文件的输入输出，而**Plugin**则可以对整个打包过程获得更多的灵活性，譬如 ExtractTextPlugin，它可以将所有文件中的css剥离到一个独立的文件中，这样样式就不会随着组件加载而加载了。

Webpack提供一个功能可以拆分模块，每一个模块称为**chunk**，这个功能叫做Code Splitting。你可以在你的代码库中定义分割点，调用require.ensure，实现按需加载&#x20;



### 常见的loader:

1\.    Babel-loader: 转换es6\es7等JS新语法特性。

2\.    Style-loader: 把css插入到的style标签里。

3\.    Css-loader: 支持css文件的加载和解析，转化成commonjs对象插入到Js中

4\.    Less-loader: 将less转为css

5\.    File-loader: 进行图片、字体等打包

6\.    url-loader: 和 file-loader 类似，但是能在文件很小的情况下以 base64 的方式把文件内容注入到代码中去.

7\.    Eslint-loader: 通过eslint检查js代码

8\.    Raw-loader: 可以将文件作为字符串导入到页面

9\.    Thread-loader: 多进程打包优化构建速度,webpack内置，原理：每次webpack解析一个模块，thread-loader会将它及放在它的之后的loader分配给node的一个单独的worker线程中，以达到多线程构建

### 常见的plugin:

1\.    Html-webpack-plugin：创建html文件去承载输出的bundle

2\.    Mini-css-extract-plugin: 将注入到js中的css单独打包成css文件，以link的形式插入到html中

3\.    Optimize-css-assets-webpack-plugin: 与cssnano一起用来压缩css

4\.    Uglifyjs-webpack-plugin: 压缩js，webpack4内置了该插件

5\.    Split-chunks-plugin:用于代码分割，结合动态import实现懒加载，webpack4内置

6\.    ModuleConcatenationPlugin: 减少模块包裹代码，将模块按照引用顺序放在一个函数作用域里，webpack4内置

7\.    Hard-source-webpack-plugin：利用缓存，提升二次构建速度

8\.    DLLPlugin/DllReferencePlugin：预编译资源模块，进行分包，通常用来分离基础库

9\.    Copy-webpacl-plugin: 将文件或者文件夹拷贝到构建的输出目录

10\.   Clean-webpack-plugin: 清理构建目录

11\.   Webpack-bundle-analyzer分析体积

12\.   htmlWebpackExternalsPlugin: 将资源以cdn的方式引入

### Loader机制：

&#x20;1\) loaders 就像首尾相接的管道那样，**从右到左**地被依次运行,从下岛上的运行；

2） loader 也是分为同步和异步两种的，比如 style-loader 是同步的（看源码就知道，直接 return）；而 less-loader 却是异步的，less-loader 本质上只是调用了 less 本身的 render 方法，由于 less.render 是异步的，less-loader 肯定也得异步，所以需要通过回调函数来获取其解析之后的 css 代码

3）node-modules 的逐级查找
