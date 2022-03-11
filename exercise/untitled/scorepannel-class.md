---
description: 属性有分数，级别，分数元素，级别元素，方法有加分，升级
---

# ScorePannel Class

```typescript
// 定义表示记分牌的类
class ScorePannel{
    score = 0;
    level = 1;
    scoreEle: HTMLElement;
    levelEle: HTMLElement;
    maxLevel: number;
    upScore: number;
    constructor(maxLevel:number=20, upScore: number= 2){
        this.scoreEle = document.getElementById('score')!;
        this.levelEle = document.getElementById('level')!;
        this.maxLevel = maxLevel;
        this.upScore = upScore;
    }
    // 加了score还需要反映到页面上
    addScore(){
        this.scoreEle.innerHTML = ++this.score+'';
        if(this.score % this.upScore ==0){
            this.levelUp();
        }
    }
    levelUp(){
        if(this.level<this.maxLevel){
            this.levelEle.innerHTML = ++this.level+''
        }
    }
}
export default ScorePannel
```
