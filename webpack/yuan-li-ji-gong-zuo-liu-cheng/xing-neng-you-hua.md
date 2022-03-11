# 性能优化

### 打包速度：指标speed-measure-webpack-plugin

* 预编译模块：dllPlugin,dllRefrerencePlugin
* 并行压缩：parallel-webpack-plugin
* 多线程打包：thread-loader
* 缓存：babel-loader?cacheDirectory = true, TerserPlugin(cache= true), HardSourceWebpackPlugin
* 缩小目标： exclude在loader声明, resolve声明别名

### 打包体积：指标webpack-bundle-analyzer

* Tree shaking: ESModule和CSS: purgecss-webpack-plugin, mini-css-extract-plugin
* code split
