---
description: react却利用其特殊的diff算法做到了O(n^3)到O(n)的飞跃性的提升
---

# React的diff算法

## 虚拟DOM：本质是用来模拟DOM的JS 对象。 一般含有标签名（tag）、属性（props）和子元素对象（children） 三个属性。

* 虚拟dom使用js对象描述的树形结构；然后用这个树构建一个真实的DOM树，插到文档中；
* 当状态变更时，重新构造一个新的对象树，然后用比较 Virtual Dom 与之前 Virtual Dom 的异同，记录两棵树的差异
* 把所记录的差异应用到所构建的DOM树上，视图就更新了；

好处：1）实现了对DOM的集中化操作，在数据改变时先对虚拟DOM进行修改，在反映到真实的DOM中，用最小的代价来更新DOM，提高效率。2）抽象了原本的渲染过程，实现了跨平台的能力，而不仅仅局限于浏览器的DOM；

## Diff算法：

三大策略：

1）web中DOM跨层级的移动操作比较小，可以忽略不计；

2 )  拥有相同类的两个组件将会生成相似的属性结构；

3）对于同一层级的一组子节点，它们可以通过唯一id进行区分；

基于三大策略，react分别对tree diff, component diff, element diff进行优化；

#### Tree diff: 对树进行分层比较，在同一父节点下的所有子节点，当发现节点已经不存在，则该节点以及子节点会被删除，这样对树只进行一次遍历，就可以完成 DOM树的比较。如果出现跨层级的，就要删除后重建，官方不建议。建议：在开发时，尽量保持树的结构，可以通过css隐藏或显示节点，不要真的移除或添加节点。

#### Component Diff: 同一类型的组件按原策略继续比较vitual DOM tree, 不是同一类型，判断为dirty component，移除；对于同一类型的组件，有可能其virtual dom没有任何变化，如果能够确切知道这点就可以节点大量的diff算法。官方允许用户通过shouldComponentUpdate()来判断该组件是否需要进行diff。

#### Element Diff:同一层级的为一个集合；

#### 1)首先先对新集合进行遍历，通过唯一的key来判断新老集合

* 如果新集合中的节点存在于旧的集合中，则需要移动，移动的规则如下：(mountindex\<nextIndex)
* 新集合中的节点不存在于旧集合中，则需要插入操作
* 旧集合当中含有新节点不存在的节点，则需要删除操作

情节一： **新旧集合中存在相同节点但位置不同时，如何移动节点**

![](<../.gitbook/assets/image (16).png>)

| index | 节点 | oldIndex | maxIndex | 操作                                                           |
| ----- | -- | -------- | -------- | ------------------------------------------------------------ |
| 0     | B  | 1        | 0        | <p>oldIndex(1)>maxIndex(0),</p><p>maxIndex=oldIndex</p>      |
| 1     | A  | 0        | 1        | <p>oldIndex(0)&#x3C;maxIndex(1),</p><p>节点A移动至index(1)的位置</p> |
| 2     | D  | 3        | 1        | <p>oldIndex(3)>maxIndex(1),</p><p>maxIndex=oldIndex</p>      |
| 3     | C  | 2        | 3        | <p>oldIndex(2)&#x3C;maxIndex(3),</p><p>节点C移动至index(2)的位置</p> |

* index： 新集合的遍历下标。
* oldIndex：当前节点在老集合中的下标。
* maxIndex：在新集合访问过的节点中，其在老集合的最大下标值。

操作一栏中只比较oldIndex和maxIndex：

* 当oldIndex>maxIndex时，将oldIndex的值赋值给maxIndex
* 当oldIndex=maxIndex时，不操作
* 当oldIndex\<maxIndex时，将当前节点移动到index的位置

情形二：新集合中有新加入的节点，旧集合中有删除的节点



![](../.gitbook/assets/5518628-eb7ef5477ea1a678.webp)

| index | 节点 | oldIndex | maxIndex | 操作                                                           |
| ----- | -- | -------- | -------- | ------------------------------------------------------------ |
| 0     | B  | 1        | 0        | oldIndex(1)>maxIndex(0)，maxIndex=oldIndex                    |
| 1     | E  | -        | 1        | <p>oldIndex不存在，添加</p><p>节点E至index(1)的位置</p>                  |
| 2     | C  | 2        | 1        | 不操作                                                          |
| 3     | A  | 0        | 3        | <p>oldIndex(0)&#x3C;maxIndex(3),</p><p>节点A移动至index(3)的位置</p> |

> 注：最后还需要对旧集合进行循环遍历，找出新集合中没有的节点，此时发现存在这样的节点D，因此删除节点D，到此 diff 操作全部完成。

同样操作一栏中只比较oldIndex和maxIndex，但是oldIndex可能有不存在的情况

总结：

先遍历新集合，

1）新旧集合共有节点，但顺序不一致：若老index<新的lastIndex，就进行移动，否则不移动新集合的节点；

2）如果新集合有，而旧集合没有，则插入；

最后遍历旧集合，若发现存在于旧集合而不存在新集合的点，则进行删除操作；

**diff的不足与待优化的地方**

****![](https://upload-images.jianshu.io/upload\_images/5518628-aea2bb7e8e843db6.png?imageMogr2/auto-orient/strip|imageView2/2/w/636/format/webp)

看图的 D，此时D不移动，但它的index是最大的，导致更新lastIndex=3，从而使得其他元素A,B,C的index\<lastIndex，导致A,B,C都要去移动。

**理想情况是只移动D，不移动A,B,C。因此，在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，会影响React的渲染性能。**

