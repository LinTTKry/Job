# promise封装ajax

```javascript
function ajax(url,data,method){
   return new Promise((resolve,reject)=>{
      const xhr = XMLHttpRequest();
      xhr.open(url);
      if(method === 'post') xhr.send(data);
      xhr.onreadystatechange= function(){
         if(xhr.state == 200 & xhr.readyState == 4){
             var json = JSON.parse(xhr.responseText);
             resolve(json)
         }else if(xhr.readyState == 4 & xhr.status!=200){
             reject('error')
         }
      }
   })
}
```
