# 实现千分位

```javascript
function parseToMoney(num){
    num = parseFloat(num.toFixed(3)).toString();
    let integer, decimal;
    [integer,decimal] = num.split('.');
    integer = integer.replace(/(\d)(?=(\d{3})+$)/g,'$1,')
    return integer+'.'+decimal
}
console.log(parseToMoney(1112222.56))
```

