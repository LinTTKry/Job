# 算法总结0308



1. 斐波那契数列和爬楼梯的优化方式尾调用（递归）的方式，其优点是不用保存上一层调用栈的变量；关键语句

```javascript
if(n==0)return ac1;
else{
    return f(n,ac2,(ac1+ac2)%1000000007);
}
```

&#x20;&#x20;

&#x20;   3\. K个一组翻转列表，最末尾的是pre,关键语句

```javascript
head.next = reverseKGroup(pre,K)
```

&#x20;   4.环形链表慢指针永远比快指针慢一步，包括初始化。

&#x20;   5.删除链表倒数第k个节点：设置头，初始化快慢指针为pre,快要先走k+1;后面走的时候要保证fast&\&fast.next都要有

```javascript
let slow = pre, fast = pre;
while(n--){
    fast = fast.next;
}
while(fast && fast.next)
```

&#x20;    6.第k个最大元素里面最小堆要交换len-1-k次，返回的是num\[0]

```javascript
for(let i = len-1; i >=len-1-k;i--){
    [num[0],num[i]]=[num[i],num[0]];
    heapify(arr,0,i)
}
return num[0]
```

&#x20;   7.具有给定数值的最小字符 :大于n-1-26取z,往前放字符，转ASCII符  &#x20;

```javascript
while(k){
   if(k>n-1-26){
      res= 'z'+res;
   } 
   else res = String.fromCharCode(k-n+1)+res
}
```

&#x20;    8\. LRU缓存机制this.map = new map()，不然没有has方法；

&#x20;    9.平方根注意

```javascript
while(l<=r){
   if(middle*middle = x) return middle;
   if(<)left = middle+1;
   if(>)right = middle -1;
}
return r
```

&#x20;    10.字符的最短距离

```javascript
//左边
let pre = -Infinity;
res[i] = i-pre
//右边
let pre = Infinity;
res[i] = min[pre-i,res[i]]
```

&#x20;   11.有效的数独注意判断空格'.'并且注意key

```javascript
if(row[num+i]||col[num+j]||boxes[num+box])
```

&#x20;     12.左旋字符串可以分开两边考虑，大于放l，小于的放r,

&#x20;     13.翻转字符串可以用左右指针，同时向中间靠近

&#x20;     14.前缀树的contain返回的是h，因为在search的时候得到标志位

```javascript
Trie.prototype.contain(word){
   let h = this.h;
   for(w of word){
      if(!h[w]) return false;
      h  = h[w]
   }
   return h
}
Trie.prototype.search(word){
   return (h=this.contain(word))&&h.end===1
}
```

&#x20;     15.二进制加法

{% hint style="info" %}
1：单纯的数字：while(b){sum = a^b,carry = (a\&b）<<1  a = sum, b = carry } return a;

2：两个二进制的字符串：

1）前面加0至相同的长度；

2）每一位parseInt后相加+carry,从末尾开始

3\)如果sum >=2; carry=1; res\[i]=sum-2;

4\)如果sum<2; ca=0; res\[i]= sum;

5\)循环完后，还要查询carry是否为1，为1则res.unshift(1);

3：两个大数的相加

1）转成字符串

2）前面加0至相同的长度；

3）sum=每一位parseInt后相加+carry,**从末尾开始**

4\)sum = sum/10; carry = sum %10;

5\)循环完后，还要查询carry是否>1，为1则res = 1+res;
{% endhint %}

&#x20;     16.质数总和：主要是for(let j = i\*i;j\<n;j+=i) f\[j]=0;
