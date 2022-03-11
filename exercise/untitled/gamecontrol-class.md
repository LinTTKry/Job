---
description: >-
  在gamecontrol
  中对所有“基本”类进行操作，注意游戏是在按下键的时候就开始，然后在按不同键调整方向，属性有蛇，食物，分数板块，方向，是否存活，方法，包括初始化（跑起来【持续】）
---

# GameControl Class

```typescript
import Food from './Food';
import ScorePannel from './ScorePannel'
import Snake from './Snake'
class GameControl{
    snake:Snake;
    food:Food;
    scorePannel: ScorePannel;
    direction:string = '';
    isLive = true;
    constructor(){
        this.snake = new Snake();
        this.food = new Food();
        this.scorePannel = new ScorePannel();        
        this.init();
    }
    init(){
        // 若不加bind(this), 回调函数中的this指向document,
        // this.direction= event.key不会正常执行
        document.addEventListener("keydown",this.keydownhandler.bind(this))       
        this.run();
     }
    keydownhandler(event:KeyboardEvent){               
        this.direction = event.key
    }
    run(){
        let X  = this.snake.X;
        let Y  = this.snake.Y;
        switch(this.direction){
            case "ArrowUp":
            case "Up":
                Y -= 10;
                break;
            case "ArrowDown":
            case "Down":
                Y += 10;
                break;
            case "ArrowLeft":
            case "Left":
                X -= 10;
                break;
            case "ArrowRight":
            case "Right":
                X += 10;
                break;           
        }
        this.checkEat(X,Y);
        try{
            this.snake.X = X
            this.snake.Y = Y
        }catch(e){
            alert(e.message+' GAME OVER')
            this.isLive = false
        }        
        this.isLive && setTimeout(this.run.bind(this),300-(this.scorePannel.level-1)*30)
    }
    checkEat(X:number, Y: number){
        if( X === this.food.X && Y === this.food.Y){
            this.food.change();
            this.scorePannel.addScore();
            this.snake.addBody();
        }
    }
}
export default GameControl;
```
