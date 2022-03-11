# hashMap的底层原理

### 哈希表是一种将键 **映射到** 值的数据结构

至少有两种方式可以实现 hashmap：

1. **数组**：通过哈希函数将键映射为数组的索引。（查找）最差情况： O(n)，平均： O(1)。
2. **二叉搜索树**: 使用自平衡二叉搜索树查找值（另外的文章会详细介绍）。 （查找）最差情况： _O(log n)_，平均：_O(log n)_。

### **通过数组实现 HashMap**

****![](https://ask.qcloudimg.com/http-save/yehe-1483919/9eo6v0pcky.jpeg?imageView2/2/w/1620)

正如上图所示，每个键都被转换为一个 **hash code**。由于数组的大小是有限的（如此例中是10），（如发生冲突，）我们必须使用模函数找到对应的桶（译者注：桶指的是数组的项），再循环遍历该桶（来寻找待查询的值）。每个桶内，我们存储的是一组组的键值对，如果桶内存储了多个键值对，将采用集合来存储它们。

&#x20;当使用类似数组之类的数据结构作为 HashMap 的实现时，冲突是难以避免的。因此，解决冲突的其中一种方式是在同一个桶中存储多个值。当我们试图访问某个键对应的值时，如果在对应的桶中发现多组键值对，则需要遍历它们（以寻找该键对应的值），时间复杂度为 _O(n)_。然而，在大多数（HashMap）的实现中， HashMap 会动态调整数组的长度以免冲突发生过多。因此我们可以说**分摊后**的查找时间为 _O(1)_

**具体实现**

1. 哈希函数

实现 HashMap 的第一步是写出一个哈希函数。这个函数会将键映射为对应（索引的）值。**完美的哈希函数** 是为每一个不同的键映射为不同的索引。借助理想的哈希函数，可以实现访问与查找在恒定时间内完成。然而，完美的哈希函数在实践中是难以实现的。你很可能会碰到两个不同的键被映射为同一索引的情况，也就是 _冲突_。

&#x20;  2\. HashMap 的简单实现

```javascript
class NaiveHashMap {
    constructor(initialCapacity = 2) {    
      this.buckets = new Array(initialCapacity);  
    }
    set(key, value) {    
      const index = this.getIndex(key);    
      this.buckets[index] = value;  
    }
    get(key) {    
      const index = this.getIndex(key);    
      return this.buckets[index];  
    }
    hash(key) {    
      return key.toString().length;  
    
    getIndex(key) {    
      const indexHash = this.hash(key);    
      const index = indexHash % this.buckets.length;    
      return index;  
    }
}
```

**1)** Hash function 转换太多相同的索引，如：这会产生非常多的冲突。

```
hash('cat') // 3hash('dog') // 3
```

**2)** 完全不处理**冲突**的情况。cat 与 dog 会重写彼此在 HashMap 中的值（它们均在桶 #1 中）

&#x20;**3)** **数组长度**。 即使我们有一个更好的哈希函数，由于数组的长度是2，少于存入元素的数量，还是会产生很多冲突。我们希望 HashMap 的初始容量大于我们存入数据的数量。

&#x20; 3\. 改进：HashMap 的主要目标是将数组查找与访问的时间复杂度，从 _O(n)_ 降至 _O(1)_。

1. 一个合适的哈希函数，尽可能地减少冲突。
2. 一个长度足够大的数组用于保存数据。

让我们重新设计哈希函数，不再采用字符串的长度为 hash code，取而代之是使用字符串中每个字符的ascii 码的总和为 hash code。

```javascript
hash(key) {  
    let hashValue = 0;  
    const stringKey = key.toString();  
    for (let index = 0; index < stringKey.length; index++) {    
        const charCode = stringKey.charCodeAt(index);    
        hashValue += charCode;  
    }  
    return hashValue;
}
hash('cat') // 312 (c=99 + a=97 + t=116)
hash('dog') // 314 (d=100 + o=111 + g=103)
// 单词 rat 与 art 转换后都是327
```

```javascript
// 可以通过根据它的 ascii 码左移（字符位置*8）来解决
hash(key) {  
    let hashValue = 0;  
    const stringKey = `${key}`;  
    for (let index = 0; index < stringKey.length; index++) {   
         const charCode = stringKey.charCodeAt(index);    
         hashValue += charCode << (index * 8);  
    }  
    return hashValue;
}
hash(1); // 49
hash('1'); // 49
```

```javascript
//将数据的类型作为转换 hash code 的一部分
hash(key) {  
    let hashValue = 0;  
    const stringTypeKey = `${key}${typeof key}`;  
    for (let index = 0; index < stringTypeKey.length; index++) {    
        const charCode = stringTypeKey.charCodeAt(index);    
        hashValue += charCode << (index * 8);  
    }  
    return hashValue;
}
//如果 HashMap 的容量足够大，那就不会产生任何冲突，因此查找操作的时间复杂度为  O(1)。
//然而，我们怎么知道容量多大才是足够呢，100？1000？还是一百万？
//（从开始就）分配大量的内存（去建立数组）是不合理的。
//因此，我们能做的是根据装载因子动态地调整容量。
//这操作被称为 rehash。
```

**装载因子**是用于衡量一个 HashMap 满的程度，可以通过存储键值对的数量除以 HashMap 的容量得到它。【当负载因子为1.0时，意味着只有当hashMap装满之后才会进行扩容，虽然空间利用率有大的提升，但是这就会导致大量的hash冲突，使得查询效率变低。当负载因子为0.5或者更低的时候，hash冲突降低，查询效率提高，但是由于负载因子太低，导致原来只需要1M的空间存储信息，现在用了2M的空间。最终结果就是空间利用率太低。（**负载因子是0.75的时候**，这是时间和空间的权衡，空间利用率比较高，而且避免了相当多的Hash冲突，使得底层的链表或者是红黑树的高度也比较低，提升了空间效率) 】

阈值(threshold) = 负载因子(loadFactor) x 容量(capacity)&#x20;

当HashMap中table数组(也称为桶)长度 >= 阈值(threshold) 就会自动进行扩容。

扩容的规则是这样的，因为table数组长度必须是2的次方数，扩容其实每次都是按照上一次tableSize位运算得到的就是做一次左移1位运算，

```javascript
const assert = require('assert');
const hashMap = new HashMap();
assert.equal(hashMap.getLoadFactor(), 0);
hashMap.set('songs', 2);
hashMap.set('pets', 7);
hashMap.set('tests', 1);
hashMap.set('art', 8);
assert.equal(hashMap.getLoadFactor(), 4/16);
hashMap.set('Pineapple', 'Pen Pineapple Apple Pen');
hashMap.set('Despacito', 'Luis Fonsi');
hashMap.set('Bailando', 'Enrique Iglesias');
hashMap.set('Dura', 'Daddy Yankee');
hashMap.set('Lean On', 'Major Lazer');
hashMap.set('Hello', 'Adele');
hashMap.set('All About That Bass', 'Meghan Trainor');
hashMap.set('This Is What You Came For', 'Calvin Harris ');
assert.equal(hashMap.collisions, 2);
assert.equal(hashMap.getLoadFactor(), 0.75);
assert.equal(hashMap.buckets.length, 16);
hashMap.set('Wake Me Up', 'Avicii'); // <--- Trigger
REHASHassert.equal(hashMap.collisions, 0);
assert.equal(hashMap.getLoadFactor(), 0.40625);
assert.equal(hashMap.buckets.length, 32);
```

**注意**。我们将使用 Map 而不是普通的对象，这是由于 Map 的键可以是任何东西而对象的键只能是字符串或者数字。此外，Map 可以保持插入的顺序。

进一步说，Map.set 只是将元素插入到数组，类似于 Array.push，因此可以说：

> 往 HashMap 中插入元素，时间复杂度是 _O(1)_。如果需要 rehash，那么复杂度则是 _O(n)_。

rehash 能将冲突可能性降至最低。rehash 操作时间复杂度是 _O(n)_ ，但不是每次插入操作都要执行，仅在需要时执行。

&#x20;如果并未发生冲突，那么 values 只会有一个值，访问的时间复杂度是_O(1)_。但我们也知道，冲突总是会发生的。如果 HashMap 的初始容量太小或哈希函数设计糟糕，那么大多数元素访问的时间复杂度是 _O(n)_。HashMap 访问操作的时间复杂度平均是 _O(1)_，最坏情况是 _O(n)_ 。

&#x20;修改(HashMap.set)或删除（HashMap.delete）键值对，分摊后的时间复杂度是 _O(1)_。如果冲突很多，可能面对的就是最坏情况，复杂度为 _O(n)_。然而伴随着 rehash 操作，可以大大减少最坏情况的发生的几率。

![](<../.gitbook/assets/image (10).png>)
