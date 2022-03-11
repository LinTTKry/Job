---
description: 属性只有本身指的元素，方法包括获取位置以及改变位置
---

# Food Class

```typescript
// 定义Food类
class Food{
    // 定义一个属性表示食物所对应的元素
    element: HTMLElement;
    constructor(){
        //！表示不会为空
        this.element = document.getElementById("food")!;
    }
    //获取食物x轴坐标
    get X(){
        return this.element.offsetLeft;
    }
    //获取食物x轴坐标
    get Y(){
        return this.element.offsetTop;
    }
    change(){
        // 生成一个随机位置
        // 食物的位置最小是0，最大是290（290+10 = 300）
        // 蛇移动一次就是一格，大小为10排序，所以要求食物的坐标必须是10的倍数
        let top = Math.round(Math.round(Math.random()*29))*10
        let left = Math.round(Math.round(Math.random()*29))*10
        this.element.style.left = left+'px';
        this.element.style.top = top+'px';
    }
}
export default Food;
```
