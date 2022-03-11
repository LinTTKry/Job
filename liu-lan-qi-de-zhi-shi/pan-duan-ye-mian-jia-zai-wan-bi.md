# 判断页面加载完毕

Document.onload：是在结构和样式加载完成才执行的JS。&#x20;

Window.onload:不仅仅要在结构和样式加载完，还要执行完所有的样式，图片这些资源文件，完全加载完才会触发Window.onload事件。

DOMContentLoaded: 当页面的DOM树解析好并且需要等待JS执行完才触发 DOMContentLoaded事件不直接等待CSS文件、图片的加载完成

onreadytstatechange: 当对象状态变更时触发这个事件，一旦document的readyState属性发生变化就会触发&#x20;
