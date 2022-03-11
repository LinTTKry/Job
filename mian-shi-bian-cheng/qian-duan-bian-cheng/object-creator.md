# Object-Creator

```javascript
function objCreator(proto){
   function F(){};
   F.prototype = proto;
   return new F()
}
```
