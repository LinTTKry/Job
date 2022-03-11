# 十大排序算法

|   算法   |    平均   |   最差  |   最好  |
| :----: | :-----: | :---: | :---: |
|   冒泡   |   n^2   |  n^2  |  n^2  |
|   选择   |   n^2   |  n^2  |  n^2  |
| 插入（稳定） |   n^2   |   n   |  n^2  |
|   希尔   | n^(3/2) |       |       |
| 归并（稳定） |  nlogn  | nlogn | nlogn |
|   快排   |  nlogn  |  n^2  | nlogn |
|    堆   |  nlogn  | nlogn | nlogn |
|   基数   |    n    |   n   |   n   |
|    桶   |    n    |   n   |   n   |
|   计数   |    n    |   n   |   n   |

## 冒泡

1：比较相邻的元素，如果第一个比第二个大，就交换他们两个；

2：从开始第一对到结尾的最后一对。逐步完成后，最后元素就是最大的数；

&#x20;3：针对所有的元素重复上述步骤，除了最后一个；

```javascript
function bubbleSort(arr){
  if(arr.length<=1) return arr;
  const len = arr.length;
  for(let i = 0; i< len-1; i++){
     for(let j = 0; j<len-1-i;j++){
       if(arr[j+1]<arr[j]){
           [arr[j],arr[j+1]]  = [arr[j+1],arr[j]] 
       }  
     }
  }
  return arr
}
console.log(bubbleSort([5,3,2,7,8,4,6,3,1]))
```

## 插入

从头到尾依次扫描来排序序列，将扫描到每个元素插入 有序序列的适当位置。

```javascript
function insertSort(arr){
  if(arr.length<=1) return arr;
  const len = arr.length;
  let pre, cur;
  for(let i =1;i <len;i++){
      pre = i-1;
      cur = arr[i];
      while(pre>=0&&arr[pre]>cur){
        arr[pre+1]= arr[pre];
        pre--
      }
      arr[pre+1] = cur
  }
  return arr
}
console.log(insertSort([5,3,2,7,8,4,6,3,1]))
```

## 选择

每轮从剩余未排序的选出最大（小）元素，放在排好序的末尾；

```javascript
function selectSort(arr){
  if(arr.length<=1) return arr;
  let minIndex;
  for(let i =0; i<arr.length; i++){
      minIndex = i;
      for(let j = i; j <arr.length; j++){
         if(arr[j]<arr[minIndex]) minIndex = j;
      }
      [arr[minIndex],arr[i]] = [arr[i],arr[minIndex]]
  }
   return arr
}
console.log(selectSort([5,3,2,7,8,4,6,3,1]))

```

## 希尔

1：选择一个增量序列依次递减

2：按增量序列个数k，对序列进行k趟排序；

3：每趟排序，根据对应的增量，将待排序分割为长度为m的子序列，分别对子序列进行插入排序；仅当增量为1的时候，整个序列作为一个表处理，表长度为整个序列的长度；

```javascript
function shellSort(arr){
  if(arr.length<=1) return arr.length;
  let gap = 1;
  const len = arr.length;
  while(gap<(len/3)){
     gap = gap *3 +1;
  }
  let pre,cur;
  for(gap; gap>0; gap = Math.floor(gap/3)){
     for(let i = gap; i<len; i++){
        pre = i-gap;
        cur = arr[i]
        while(pre>=0 && arr[pre]>cur){
            arr[pre+gap] = arr[pre];
            pre -= gap;
        }
        arr[pre+gap] = cur
     }
  }
  return arr
}
console.log(shellSort([5,3,2,7,8,4,6,3,1]))
```

归并排序

递归的将数组不断二分再合并

```javascript
function mergeSort(arr){
  if(arr.length<=1) return arr;
  let middle = Math.floor(arr.length/2);
  let left = arr.slice(0,middle);
  let right = arr.slice(middle);
  return merge(mergeSort(left),mergeSort(right))
}
function merge(left,right){
  let res = [];
  while(left.length && right.length){
      if(left[0]<right[0]) res.push(left.shift())
      else res.push(right.shift())
  }
  while(left.length) res.push(left.shift())
  while(right.length) res.push(right.shift())
  return res
}
var arr = mergeSort([1,0,56,14,13,45,1]);
console.log(arr);
```

## 快排

1：从数列中挑出一个基准pivot

2:  重新排序数列，所有元素比基准小的八方在基准前面，所有的元素比基准值大的摆在基准的后面；在这个分区退出之后，基准就在数列的中间位置；

3：递归的把小于基准的子序列和大于基准的子序列排序；

```javascript
function quickSort(arr,left,right){
  const len = arr.length;
  if(len <=1) return arr;
  left = left===undefined ? 0:left
  right = right===undefined ? (len-1):right;
  if(left<right){
     let pivot = partition(arr,left,right);
     quickSort(arr.slice(left,pivot))
     quickSort(arr.slice(pivot,right))
  }
  return arr
}
function partition(arr,left,right){
   let pivot = arr[left];
   while(left<right){
       while(left<right && arr[right]>pivot){
        right--;
       } 
       arr[left]  = arr[right]
       while(left<right && arr[left]<pivot){
        left++;
       } 
       arr[right]  = arr[left]
   }
   arr[left] = pivot;
   return left
}
var arr = quickSort([32,12,11,78,76,45,36]);
console.log(arr);
```

## 基数排序（LSD）

将整数按位数分割成不同的数字，然后按每个位数分别比较；

```javascript
function radixSort(arr,maxDigit){
  const len = arr.length;
  if(len <= 1) return arr;
  let count = [];
  let dev = 1, mod = 10;
  for(let i = 0; i<maxDigit; i++,dev*=10,mod*=10){
     for(let j = 0; j< len; j++){
       let digit = parseInt((arr[j] % mod) / dev)
       if(!count[digit]) count[digit] = [];
       count[digit].push(arr[j])
     }  
     let pos = 0;
     for(let j = 0; j < count.length; j++){
        if(count[j]){
         let value
           while((value = count[j].shift())){
              arr[pos++] = value
           }
        }
     }
  }
  return arr
}
var arr = radixSort([1,5,4,3,26,8,6],5)
console.log(arr)
```

## 计数排序

将输入的数据转化为键储存在额外开辟的数组空间中

```javascript
function countSort(arr){
  const len = arr.length;
  if(len <= 1) return arr;
  let count = [];
  for(let num of arr){
     if(!count[num]) count[num]= 0;
     count[num]++;
  }
  let pos = 0;
  for(let i = 0; i< count.length; i++){
     while(count[i]){
        arr[pos++] = i;
        count[i]--;
     }
  }
  return arr
}
console.log(countSort([1,2,5,4,7,2,3]))
```

## 桶排序

将数组里面的数分配到相应的桶里；

```javascript
function bucketSort(arr,bucketSize){
  const len = arr.length;
  if(len<=1) return arr;
  let max = 0, min  = 0;
  for(let num of arr){
    if(num < min) min = num;
    if(num > max) max = num
  };
  let bucketCount = Math.floor((max-min)/bucketSize)+1;
  let buckets = Array.from({length:bucketCount},x=>[]);
  for(let num of arr){
     buckets[Math.floor((num-min)/bucketSize)].push(num)
  };
  let pos = 0;
  for(let i = 0; i<buckets.length; i++){
     if(buckets[i]){
        insertSort(buckets[i])
        for(let j = 0; j < buckets[i].length; j++){
           arr[pos++] = buckets[i][j]
        }
     }
  }
  return arr
}
function insertSort(arr){
  if(arr.length <=1) return arr;
  var pre, cur;
  for(let i = 1; i < arr.length; i++){
    pre = i-1;
    cur = arr[i];
    while(pre >=0 && arr[pre]>cur){
      arr[pre+1] = arr[pre];
      pre--;
    }
    arr[pre+1] = cur;
  }
  return arr
}
var arr = bucketSort([1,5,4,3,26,8,6],3)
console.log(arr)
```

## 堆排序

将待排序的数组建成一个最大（小）堆，

迭代：交换堆首尾，并把堆尺寸较小；并重新最小化堆

```javascript
function heapSort(arr){
  const len = arr.length;
  if(len<=1) return arr;
  buildMaxHeap(arr);
  for(let i = len-1; i>=0;i--){
    [arr[i],arr[0]]= [arr[0],arr[i]]
    heapify(arr,0,i)
  }
  return arr
}
function buildMaxHeap(arr){
  const len = arr.length;
  if(len<=1) return arr;
  let middle = parseInt(arr.length/2);
  for(let i = middle; i>=0; i--){
    heapify(arr,i,len)
  }
}
function heapify(arr,i,len){
   let left = 2*i+1,right = 2*i+2,largest = i;
   if(left < len && arr[left]>arr[largest]) largest = left
   if(right< len && arr[right]>arr[largest]) largest = right
   if(largest!=i){
      [arr[i],arr[largest]] = [arr[largest],arr[i]]
      heapify(arr,largest,len)
   }
}
console.log(heapSort([5,3,2,7,8,4,6,3,1]))
```
