# 渲染流程：html、js、css如何变成页面

### 1：定义

## ​![](<../.gitbook/assets/image (61) (1).png>)

**HTML** 的内容是由**标记（标签）和文本**组成；

**CSS** 又称为层叠样式表，是由**选择器和属性**组成；

**JavaScript**（简称为 JS），**使用它可以使网页的内容“动”起来**，比如上图中，可以通过 JavaScript 来修改 CSS 样式值，从而达到修改文本颜色的目的。

### ​2：渲染流水线

![](<../.gitbook/assets/9259f8732ddad472e5e08a633ad46de8 (1).webp>)

流水线可分为如下几个子阶段：构建 DOM 树、样式计算、布局阶段、分层、绘制、分块、光栅化和合成。

开始每个子阶段都有其输入的内容；

然后每个子阶段有其处理过程；

最终每个子阶段会生成输出内容。

#### A. 构建DOM树

这是因为浏览器无法直接理解和使用 HTML，所以需要将 HTML 转换为浏览器能够理解的结构——DOM 树

![​](../.gitbook/assets/9259f8732ddad472e5e08a633ad46de8.webp)

为了更加直观地理解 DOM 树，你可以打开 Chrome 的“开发者工具”，选择“Console”标签来打开控制台，然后在控制台里面输入“document”后回车，这样你就能看到一个完整的 DOM 树结构，如下图所示：

![](<../.gitbook/assets/image (60).png>)

DOM 可视化图中的 document 就是 DOM 结构，你可以看到，DOM 和 HTML 内容几乎是一样的，但是和 HTML 不同的是，DOM 是保存在内存中树状结构，可以通过 JavaScript 来查询或修改其内容。那下面就来看看如何通过 JavaScript 来修改 DOM 的内容，在控制台中输入：

```javascript
document.getElementsByTagName("p")[0].innerText = "black"
```

这行代码的作用是把第一个标签的内容修改为 black，具体执行结果你可以参考下图：

![](<../.gitbook/assets/image (65).png>)

通过 JavaScript 修改 DOM从图中可以看出，在执行了一段修改第一个标签的 JavaScript 代码后，DOM 的第一个 p 节点的内容成功被修改，同时页面中的内容也被修改了。好了，现在我们已经生成 DOM 树了，但是 DOM 节点的样式我们依然不知道，要让 DOM 节点拥有正确的样式，这就需要样式计算了。

#### B.样式计算（Recalculate Style）

* [ ] 把 CSS 转换为浏览器能够理解的结构

![](<../.gitbook/assets/image (63).png>)

和 HTML 文件一样，浏览器也是无法直接理解这些纯文本的 CSS 样式，所以当渲染引擎接收到 CSS 文本时，会执行一个转换操作，将 CSS 文本转换为浏览器可以理解的结构——styleSheets。为了加深理解，你可以在 Chrome 控制台中查看其结构，只需要在控制台中输入 document.styleSheets，然后就看到如下图所示的结构：

![](<../.gitbook/assets/image (58).png>)

从图中可以看出，这个样式表包含了很多种样式，已经把那三种来源的样式都包含进去了。当然样式表的具体结构不是我们今天讨论的重点，你只需要知道渲染引擎会把获取到的 CSS 文本全部转换为 styleSheets 结构中的数据，并且该结构同时具备了查询和修改功能，这会为后面的样式操作提供基础。

* [ ] 转换样式表中的属性值，使其标准化

那么接下来就要对其进行属性值的标准化操作, 要理解什么是属性值标准化，你可以看下面这样一段 CSS 文本：

```javascript
body { font-size: 2em } 
p {color:blue;}
span {display: none} 
div {font-weight: bold}
div p {color:green;} 
div {color:red; }
```

可以看到上面的 CSS 文本中有很多属性值，如 2em、blue、bold，这些类型数值不容易被渲染引擎理解，所以需要将所有值转换为渲染引擎容易理解的、标准化的计算值，这个过程就是属性值标准化。

![](<../.gitbook/assets/image (6).png>)

* [ ] 计算出 DOM 树中每个节点的具体样式

这就涉及到 CSS 的继承规则和层叠规则了。首先是 CSS 继承。CSS 继承就是每个 DOM 节点都包含有父节点的样式。这么说可能有点抽象，我们可以结合具体例子，看下面这样一张样式表是如何应用到 DOM 节点上的。body { font-size: 20px }

```javascript
p {color:blue;}
span  {display: none}
div {font-weight: bold;color:red}
div  p {color:green;}
```

这张样式表最终应用到 DOM 节点的效果如下图所示：

![](<../.gitbook/assets/image (62).png>)

从图中可以看出，所有子节点都继承了父节点样式。比如 body 节点的 font-size 属性是 20，那 body 节点下面的所有节点的 font-size 都等于 20。为了加深你对 CSS 继承的理解，你可以打开 Chrome 的“开发者工具”，选择第一个“element”标签，再选择“style”子标签，你会看到如下界面：

![](<../.gitbook/assets/image (64).png>)

样式计算阶段的目的是为了计算出 DOM 节点中每个元素的具体样式，在计算过程中需要遵守 CSS 的继承和层叠两个规则。

#### C: 布局阶段
