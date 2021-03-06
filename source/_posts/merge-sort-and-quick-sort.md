title: 归并排序与快速排序的简明实现及对比
date: 2017-12-07 20:33:54
comments: true
tags: [排序, 算法]
categories: 编程
---

## 前言

归并排序与快速排序是两种有实际应用的排序算法，它们有一些共同的特点，整体思路上也比较相近。本文会从更简单的一些排序算法开始，过渡到归并排序和快速排序的实现，并对它们做一些简单的对比思考和总结。在这之前，先简单介绍一下排序算法的意义。

排序算法就是将一串数据依照特定排序方式进行排列，它们在计算机科学中有大量研究以及应用。

想象一下下列场景：

1. 从通讯录中寻找某个联系人
2. 从一大堆文件中寻找某个文件
3. 到了影厅之后，寻找电影票上指定的座位

如果以上情况中，联系人、文件、影厅座位这些“数据”没有按照需要的顺序组织，如何找到想要的特定“数据”呢？会非常麻烦！所以说，对于需要搜索的数据，往往应该先排个序！


## 热身一：选择排序

本文的示例都是数值排序，对于这个问题，最简单直观的方法是：先找出最小的、再找出第二小的、接着找出第三小的……这就是选择排序的思路。<!-- more -->

```javascript
function selectionSort(array) {
  const len = array.length;

  for (let i = 0; i < len - 1; i++) {
    let min = i;

    for (let j = i + 1; j < len; j++) {
      if (array[min] > array[j]) {
        min = j;
      }
    }

    [ array[min], array[i] ] = [ array[i], array[min] ];
  }

  return array;
}
```

实现解析：

1. 遍历数组
2. 找到当前范围内最小的元素，用 minIndex 记录它的下标，第一次遍历时范围就是整个数组
3. 将下标为 minIndex 的元素的值与当前最小下标的元素交换，第一次遍历时下标最小的元素就是 a[0]
4. 第二次遍历时，范围就从第二个数据元素的下标开始，那么当前最小下标元素就是 a[1]
5. 重复交换直至遍历结束

用一段辅助代码，做一些展示用的示例。

```javascript
function createUnsortedArray(size) {
  const array = [];

  for (let i = size; i > 0; i--) {
    const num = (i / 10 > 1) ? i : 10;
    array.push( Math.round(Math.random(i) * num + Math.round(Math.random(i)) * Math.random(i) * num * 10) );
  }

  return array;
}

function show(fn, size = 11) {
  console.log('------------------------------------------');
  console.log(`Method: ${fn.name}`);
  console.log('------------------------------------------');
  const array = createUnsortedArray(size);
  console.log('before:');
  console.log(array.toString());
  console.log('after:');
  console.log(fn(array).toString());
}
```

先创建一个随机生成的未排序的数组，然后打印结果。

```javascript
show(selectionSort);

// ------------------------------------------
// Method: selectionSort
// ------------------------------------------
// before:
// 9,22,3,27,74,54,8,41,80,74,3
// after:
// 3,3,8,9,22,27,41,54,74,74,80
```


## 热身二：冒泡排序

冒泡排序与选择排序有些类似，区别在于冒泡排序是先将最大值冒泡到最后的位置。早在 1956 年，就已经有人研究冒泡排序。

```javascript
function bubbleSort(array) {
  for (let first = 0, len = array.length; first < len; first++) {
    let isSorted = true;

    for (let second = 0; second < len - first - 1; second++) {
      if (array[second] > array[second + 1]) {
        let temp = array[second];
        array[second] = array[second + 1]
        array[second + 1] = temp;
        isSorted = false;
      }
    }

    if (isSorted) {
      break;
    }
  }

  return array;
}

show(bubbleSort);

// ------------------------------------------                                      
// Method: bubbleSort                                                              
// ------------------------------------------                                      
// before:
// 35,8,2,2,8,1,3,4,2,10,4
// after:
// 1,2,2,2,3,4,4,8,8,10,35
```

实现解析：

1. 遍历数组
2. 做第二层遍历，从前到后依次对比相邻两项，前一项的值大于后一项，则交换（冒泡）。第一遍冒泡，将最大的元素值冒泡至最后
3. 由于每一遍冒泡都确定一个当前最大值并放到当前范围的最后的位置，每一遍的冒泡就可以少检查一个位置
4. 可以使用一个变量记录当前一遍的冒泡有没有产生元素交换，如果没有，说明当前已经是排序完成的状态，终止循环


## 热身三：插入排序

插入排序的思想在日常生活其实很常见，例如如何排定卢俊义的座次？综合出身、能力、江湖地位、形势人心等各项指标，他在梁山泊排名第二，地位仅次于宋江。这就是插入排序的思路。数据量很小，或类似“给卢俊义排座次”这种在已排序数据中增加一条数据的情况，插入排序优于本文提到的其他排序方式。

```javascript
function insertionSort(array) {
  for (let i = 0, len = array.length; i < len; i++) {
    let j = i;
    const temp = array[i]

    while (j > 0 && array[j - 1] > temp) {
      array[j] = array[j - 1];
      j--;
    }

    array[j] = temp;
  }

  return array;
}

show(insertionSort);

// ------------------------------------------                                      
// Method: insertionSort                                                              
// ------------------------------------------                                      
// before:
// 3,8,68,30,28,56,35,30,2,4,13
// after:
// 2,3,4,8,13,28,30,30,35,56,68
```

实现解析：

1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
6. 重复步骤2~5


## 归并排序（递归实现）

选择排序和冒泡排序的时间复杂度都是 O(n^2)，很少用在实际工程中；归并排序的时间复杂度是 O(nlog(n))，是实际工程中可选的排序方案。

```javascript
function mergeSort(unsorted) {
  function merge(leftArr, rightArr) {
    const lenL = leftArr.length;
    const lenR = rightArr.length;
    let indexL = 0;
    let indexR = 0;
    const result = [];

    while (indexL < lenL && indexR < lenR) {
      if (leftArr[indexL] < rightArr[indexR]) {
        result.push(leftArr[indexL++]);
      } else {
        result.push(rightArr[indexR++]);
      }
    }

    while (indexL < lenL) {
      result.push(leftArr[indexL++]);
    }

    while (indexR < lenR) {
      result.push(rightArr[indexR++]);
    }

    return result;
  }

  function split(array) {
    const len = array.length;

    if (len <= 1) {
      return array;
    }

    const mid = Math.floor(len / 2);

    const leftArr = array.slice(0, mid);
    const rightArr = array.slice(mid, len);

    return merge( split(leftArr), split(rightArr) );
  }

  return split(unsorted);
}

show(mergeSort);

// ------------------------------------------
// Method: mergeSort
// ------------------------------------------
// before:
// 86,55,0,31,104,6,5,49,89,19,6
// after:
// 0,5,6,6,19,31,49,55,86,89,104
```

实现分析：

1. 将数组从中间切分为两个数组
2. 切分到最小之后，开始归并操作，即合并两个已排序的数组
3. 递归合并的过程，由于是从小到大合并，所以待合并的两个数组总是已排序的，一直做同样的归并操作就可以


## 快速排序（递归实现）

快速排序是实际应用非常多的排序算法，它通常比其他 O(nlog(n)) 时间复杂度的算法更快。

```javascript
function quickSort(unsorted) {
  function partition(array, left, right) {
    const pivot = array[ Math.floor((left + right) / 2) ];

    while (left <= right) {
      while (array[left] < pivot) {
        left++;
      }

      while (array[right] > pivot) {
        right--;
      }

      if (left <= right) {
        [array[left], array[right]] = [array[right], array[left]];
        left++;
        right--;
      }
    }

    return left;
  }

  function quick(array, left, right) {
    if (array.length <= 1) {
      return array;
    }

    const index = partition(array, left, right);

    if (left < index - 1) {
      quick(array, left, index - 1);
    }

    if (right > index) {
      quick(array, index, right);
    }

    return array;
  }

  return quick(unsorted, 0, unsorted.length - 1);
}

show(quickSort);

// ------------------------------------------
// Method: quickSort
// ------------------------------------------
// before:
// 41,9,22,4,1,32,10,28,4,94,3
// after:
// 1,3,4,4,9,10,22,28,32,41,94
```

实现分析：

1. 将当前数组分区
2. 分区时先选择一个基准值，再创建两个指针，左边一个指向数组第一个项，右边一个指向数组最后一个项。移动左指针直至找到一个比基准值大的元素，再移动右指针直至找到一个比基准值小的元素，然后交换它们，重复这个过程，直到左指针的位置超过了右指针。如此分区、交换使得比基准值小的元素都在基准值之前，比基准值大的元素都在基准值之后，这就是分区（partition）操作。
3. 对于上一次分区后的两个区域重复进行分区、交换操作，直至分区到最小。


## 对比归并排序与快速排序

1. 都用了分治的思想。相比选择排序和冒泡排序，归并排序与快速排序使用了切分而不是直接遍历，这有效减少了交换次数。
2. 归并排序是先切分、后排序，过程可以描述为：切分、切分、切分……排序、排序、排序……
3. 快速排序是分区、排序交替进行，过程可以描述为：分区、排序、分区、排序……
4. 上两条所说的“排序”，在归并排序与快速排序中并非同样的操作，归并排序中的操作是将两个数组合并为一（归并操作），而快速排序中的操作是交换。


## 参考资料

1. [学习JavaScript数据结构与算法（第2版）](http://www.ituring.com.cn/book/2029)
2. [Sorting algorithm](https://en.wikipedia.org/wiki/Sorting_algorithm)
