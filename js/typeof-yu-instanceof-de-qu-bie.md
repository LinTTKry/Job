# typeof 与 instanceof的区别

1. typeof 用来判断基本类型，对于引用类型，返回的类型有基本类型，object（数组，{}，正则）,  function
2. instanceof 用来判断引用类型，本意是判断一个变量是否是某个对对象的实例或者说某个构造函数的原型链是否另一个对象的原型链上；
3.  在跨框架（cross-frame）页面中的数组时，会失败。原因就是在不同框架（iframe）中创建的数组不会相互共享其prototype属性

    ```
    <script>
    window.onload=function(){
    var iframe_arr=new window.frames[0].Array;
    alert(iframe_arr instanceof Array); // false
    alert(iframe_arr.constructor == Array); // false
    }
    </script>
    ```

    1 2 3 4 5 6 7 而跨原型链调用toString()方法：Object.prototype.toString()，可以解决上面的跨框架问题。Object.prototype.toString.call(parent)
