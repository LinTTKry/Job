# deepclone

```javascript
function deepclone (parent){
  if(parent === null) return null
  if(typeof parent !== 'object') return parent;
  getType=(parent,type)=>{
     const typeStr = Object.prototype.toString.call(parent);
     let flag;
     switch(type){
        case 'Date':
           flag = typeStr === '[object Date]';
           break;
        case 'RegExp':
           flag = typeStr === '[object RegExp]';
           break;
        case 'Array':
           flag = typeStr === '[object Array]';
           break;     
     }
     return flag;
  }
  getdec=(parent)=>{
     let flag = '';
     if(parent.global) flag+='g';
     if(parent.ignoreCase) flag+='i';
     if(parent.multiline) flag +='m';
     return flag;
  }
  let children = [], parents = [];
  clone= (parent)=>{
      let child;
      if(typeof parent !== 'object') return parent
      if(getType(parent,'Array')){
            child = [];
      }else if(getType(parent,'Date')){
           child = new Date(parent)
      }else if(getType(parent,'RegExp')){
           child = new RegExp(parent.source,getdec(parent));
           if(parent.lastIndex)child.lastIndex = parent.lastIndex
      }else{
          let proto = Object.getPrototypeOf(parent);
          child = Object.create(proto)
      }
      let index = parents.indexOf(parent);
      if(index !== -1) return children[index];
      parents.push(parent);
      children.push(parent);
      for(let i = 0 ; i<parent.length; i++){
          child[i] = clone(parent[i])
      }
      return child
  }
  return clone(parent)
}
```
