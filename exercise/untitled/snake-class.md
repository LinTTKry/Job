---
description: >-
  注意：蛇不行撞到自己，蛇不行掉头，不行出边界，蛇在吃到食物的时候要加身体，身体的移动为后一节移动到前一节的位置；属性有蛇本身元素，身体元素，头元素，方法包括获取头位置，设置头位置，加身体，移动身体
---

# Snake Class

```typescript
class Snake{
    head:HTMLElement;
    bodies:HTMLCollection;
    element:HTMLElement;
    constructor(){
        this.element = document.getElementById("snake")!
        this.head = document.querySelector("#snake>div") as HTMLElement;
        this.bodies = this.element.getElementsByTagName('div');
    }
    get X(){
        return this.head.offsetLeft;
    }
    get Y(){
        return this.head.offsetTop;
    }
    set X(value:number){
       if(this.X === value) return
       if(value<0|| value>290){
         throw new Error('蛇死了')
       }
       // 发生水平方向的调头
       if(this.bodies[1] && (this.bodies[1]as HTMLElement).offsetLeft === value){
            if(value > this.X){
                value = this.X - 10;
            }else{
                value = this.X + 10;
            }
       }
       this.moveBody();
       this.head.style.left = value+'px'
       this.checkHeadBody();       
    }
    set Y(value:number){
        if(this.Y === value) return
        if(value<0|| value>290){
            throw new Error('蛇死了')
        }
        // 发生垂直方向的调头
       if(this.bodies[1] && (this.bodies[1]as HTMLElement).offsetTop === value){
            if(value > this.Y){
                value = this.Y - 10;
            }else{
                value = this.Y + 10;
            }
        }
        this.moveBody();
        this.head.style.top = value+'px'
        this.checkHeadBody();
    }
    addBody(){
        this.element.insertAdjacentHTML('beforeend',"<div></div>")
    }
    moveBody(){
        // 将后边的身体设置为前边身体的位置（从后往前）
        // 蛇头的位置改过了， i！=0
        for(let i = this.bodies.length-1; i>0;i--){
            let X= (this.bodies[i-1] as HTMLElement).offsetLeft;
            let Y= (this.bodies[i-1] as HTMLElement).offsetTop;

            (this.bodies[i] as HTMLElement).style.left = X + 'px';
            (this.bodies[i] as HTMLElement).style.top = Y + 'px';
        }
    }
    checkHeadBody(){
        // 获取所有身体，检查蛇头是否与身体坐标重复
        for(let i =1; i < this.bodies.length; i++){
            let bd = this.bodies[i] as HTMLElement;
            if(this.X === bd.offsetLeft && this.Y=== bd.offsetTop){
                throw new Error('撞到自己了')
            }
        }
    }
}
export default Snake;
```
