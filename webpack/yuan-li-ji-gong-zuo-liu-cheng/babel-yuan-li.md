# babel原理

&#x20;详细参考：[https://my.oschina.net/u/4088983/blog/4545928](https://my.oschina.net/u/4088983/blog/4545928)

使用 Babel parser 解析器对输入的源代码字符串进行解析并生成初始 AST 遍历 AST 树并应用各 transformers（plugin） 生成变换后的 AST 树 利用 babel-generator 将 AST 树输出为转码后的代码字符串 分为三个阶段：

解析：将代码字符串解析成抽象语法树&#x20;

变换：对抽象语法树进行变换操作&#x20;

再建：根据变换后的抽象语法树再生成代码字符串
