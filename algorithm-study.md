title: 基础不能丢-排序算法复习
date: 2016-02-24 08:34
tags: [算法,在成为最厉害最厉害最厉害的路上]
categories: 算法
---

慢慢总结、实现一些排序算法。

<!-- more -->

## 快速排序(QUICK)

### 原理

> 快速排序是图灵奖得主 C. R. A. Hoare 于 1960 年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。

![enter image description here](http://bubkoo.qiniudn.com/C.R.A.Hoare.jpg)
C. R. A. Hoare

### 分治法的基本思想：

> 将原问题分解为若干个规模更小但结构与原问题相似的子问题。递归地解这些子问题，然后将这些子问题的解组合为原问题的解。

利用分治法可将快速排序的分为三步：

1. 在数据集之中，选择一个元素作为”基准”（pivot）。
2. 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为分区 (partition) 操作，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
3. 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。

如下图：
![enter image description here](http://bubkoo.qiniudn.com/Sorting_quicksort_anim.gif)

### 伪代码实现


```javascript
	//一趟： 参数 a:数组、left 左侧index、右侧index、基准值index
	function partition(a, left, right, pivotIndex)
	     pivotValue := a[pivotIndex] // 获取基准值
	     swap(a[pivotIndex], a[right]) // 把 pivot 移到結尾
	     storeIndex := left //即将被交换的位置坐标。
	     for i from left to right-1
	         if a[i] < pivotValue //如果小于基准值
	             swap(a[storeIndex], a[i]) //交换位置
	             storeIndex := storeIndex + 1  
	     swap(a[right], a[storeIndex]) // 把 pivot 移到它最後的地方（把基准值移动到正确的位置上，即 storeindex上。）
	     return storeIndex // 返回 pivot 的最终位置

	//递归调用partition()
	procedure quicksort(a, left, right)
	    if right > left
	        select a pivot value a[pivotIndex]
	        pivotNewIndex := partition(a, left, right, pivotIndex)
	        quicksort(a, left, pivotNewIndex-1)
	        quicksort(a, pivotNewIndex+1, right)
```

### 举个栗子：

举例来说，现有数组 arr = [3,7,8,5,2,1,9,5,4]，分区可以分解成以下步骤：

首先选定一个基准元素，这里我们元素 5 为基准元素（基准元素可以任意选择）：

```javascript
          pivot
            ↓
3   7   8   5   2   1   9   5   4
```

将基准元素与数组中最后一个元素交换位置，如果选择最后一个元素为基准元素可以省略该步：
```javascript
                              pivot
                                ↓
3   7   8   4   2   1   9   5   5
```

从左到右（除了最后的基准元素），循环移动小于基准元素 5 的所有元素到数组开头，留下大于等于基准元素的元素接在后面。在这个过程它也为基准元素找寻最后摆放的位置。循环流程如下：

循环 i == 0 时，storeIndex == 0，找到一个小于基准元素的元素 3，那么将其与 storeIndex 所在位置的元素交换位置，这里是 3 自身，交换后将 storeIndex 自增 1，storeIndex == 1：

```javascript
                                pivot
                                  ↓
  3   7   8   4   2   1   9   5   5
  ↑
storeIndex
```

循环 i == 3 时，storeIndex == 1，找到一个小于基准元素的元素 4：

```javascript
     ┌───────┐                 pivot
     ↓       ↓                   ↓
 3   7   8   4   2   1   9   5   5
     ↑       ↑
storeIndex   i
```

交换位置后，storeIndex 自增 1，storeIndex == 2：

```javascript
                              pivot
                                ↓
3   4   8   7   2   1   9   5   5
        ↑           
   storeIndex

```

循环 i == 4 时，storeIndex == 2，找到一个小于基准元素的元素 2：

```javascript
        ┌───────┐             pivot
        ↓       ↓               ↓
3   4   8   7   2   1   9   5   5
        ↑       ↑
   storeIndex   i
```

交换位置后，storeIndex 自增 1，storeIndex == 3：

```javascript
                              pivot
                                ↓
3   4   2   7   8   1   9   5   5
            ↑           
       storeIndex
```
循环 i == 5 时，storeIndex == 3，找到一个小于基准元素的元素 1：

```javascript
            ┌───────┐         pivot
            ↓       ↓           ↓
3   4   2   7   8   1   9   5   5
            ↑       ↑
       storeIndex   i
```

交换后位置后，storeIndex 自增 1，storeIndex == 4：

```javascript
                              pivot
                                ↓
3   4   2   1   8   7   9   5   5
                ↑           
           storeIndex
```

循环 i == 7 时，storeIndex == 4，找到一个小于等于基准元素的元素 5：

```javascript
                ┌───────────┐ pivot
                ↓           ↓   ↓
3   4   2   1   8   7   9   5   5
                ↑           ↑
           storeIndex       i
```

交换后位置后，storeIndex 自增 1，storeIndex == 5：

```javascript
                              pivot
                                ↓
3   4   2   1   5   7   9   8   5
                    ↑           
               storeIndex
```
循环结束后交换基准元素和 storeIndex 位置的元素的位置：

```javascript
                  pivot
                    ↓
3   4   2   1   5   5   9   8   7
                    ↑           
               storeIndex
```

那么 storeIndex 的值就是基准元素的最终位置，这样整个分区过程就完成了。

![enter image description here](http://bubkoo.qiniudn.com/Partition_example.svg.png)


### PHP代码实现

```php
<?php
function quicksort(&$arr, $left, $right)
{
    $i = $left;
    $j = $right;
    
    //设定基准值
    $separator = $arr[$left];
     
    //以 $separator 进行划分。结束条件：比$separator小的被分配到（形式上）左侧数组，大的分配到（形式上）右侧数组。
    while ($i <= $j) {
        while ($arr[$i] < $separator) {
            $i++;
        }
         
        while($arr[$j] > $separator) {
            $j--;
        }
         
        if ($i <= $j) {
            $tmp = $arr[$i];
            $arr[$i] = $arr[$j];
            $arr[$j] = $tmp;
            $i++;
            $j--;
        }
    }
     
    //递归排序形式上的两组（小于 $separator 的组、大于等于 $separator 的分组。）
    if ($left < $j) {
        quicksort($arr, $left, $j);
    }
     
    if ($right > $i) {
        quicksort($arr, $i, $right);
    }
}
 
// Example:
$arr = array(1,20,12,4,13,5);
$result = quicksort($arr, 0, (sizeof($arr)-1));
print_r($arr);
```

## 归并排序(MERGE)

### 原理

> 归并排序（Merge Sort，台湾译作：合并排序）是建立在归并操作上的一种有效的排序算法。该算法是采用**分治法**（Divide and Conquer）的一个非常典型的应用。
> 
> 归并操作(Merge)，也叫归并算法，指的是将两个已经排序的序列合并成一个序列的操作。归并排序算法依赖归并操作。归并排序有多路归并排序、两路归并排序 , 可用于内排序，也可以用于外排序。这里仅对内排序的两路归并方法进行讨论。


算法思路：

1. 把 n 个记录看成 n 个长度为 l 的有序子表
2. 进行两两归并使记录关键字有序，得到 n/2 个长度为 2 的有序子表
3. 重复第 2 步直到所有记录归并成一个长度为 n 的有序表为止。

![enter image description here](http://bubkoo.qiniudn.com/merge-sort-animation.gif)


### 举个栗子



以数组 array = [6, 5, 3, 1, 8, 7, 2, 4] 为例，首先将数组分为长度为 2 的子数组，并使每个子数组有序：
```php
[6, 5]  [3, 1]  [8, 7]  [2, 4]
   ↓       ↓       ↓       ↓
[5, 6]  [1, 3]  [7, 8]  [2, 4]
```
然后再两两合并：

```php
[6, 5, 3, 1]  [8, 7, 2, 4]
      ↓             ↓
[1, 3, 5, 6]  [2, 4, 7, 8]
```
最后将两个子数组合并：

```php
[6, 5, 3, 1, 8, 7, 2, 4]
            ↓
[1, 2, 3, 4, 5, 6, 7, 8]
```

### 图示

![enter image description here](http://bubkoo.qiniudn.com/merge-sort-example-300px.gif)


![enter image description here](http://bubkoo.qiniudn.com/merge-sort-example.gif)

### PHP实现
```php
function compare_merge($arr1, $arr2) {
    $arr = array();
    
    while(!empty($arr1) && !empty($arr2)) $arr[] = $arr1[0] <= $arr2[0] ? array_shift($arr1) : array_shift($arr2);
    
    //合并数组
    return array_merge($arr, $arr1, $arr2);
}

function merge_sort($arr) {
    //分成两组
    $length = count($arr);
    
    //临界值
    if ($length <= 1) return $arr;
    
    $mid    = floor($length/2);
    $arr1   = array_slice($arr, 0, $mid);
    $arr2   = array_slice($arr, $mid);
    
    //分别进行排序，排序后合并
    return compare_merge(merge_sort($arr1), merge_sort($arr2));
}


$arr = array(12,123,23,1,1,124,5,2,23,53);
print_r(merge_sort($arr));
```


## 选择排序（select）

### 原理

> 选择排序（Selection Sort）是一种简单直观的排序算法。它的工作原理如下，首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

### 图片演示

![enter image description here](http://bubkoo.qiniudn.com/selection_sort_animation.gif)

### 举个栗子

以数组 arr = [8, 5, 2, 6, 9, 3, 1, 4, 0, 7] 为例，先直观看一下每一步的变化，后面再介绍细节
> 
第一次从数组 [8, 5, 2, 6, 9, 3, 1, 4, 0, 7] 中找到最小的数 0，放到数组的最前面（与第一个元素进行交换）：

```php
                               min
                                ↓
8   5   2   6   9   3   1   4   0   7
↑                               ↑
└───────────────────────────────┘
```

交换后：
```php
0   5   2   6   9   3   1   4   8   7
```
在剩余的序列中 `[5, 2, 6, 9, 3, 1, 4, 8, 7]` 中找到最小的数 1，与该序列的第一个个元素进行位置交换：

```php
                       min
                        ↓
0   5   2   6   9   3   1   4   8   7
    ↑                   ↑
    └───────────────────┘
```

交换后：
```php
0   1   2   6   9   3   5   4   8   7
```

在剩余的序列中 [2, 6, 9, 3, 5, 4, 8, 7] 中找到最小的数 2，与该序列的第一个个元素进行位置交换（实际上不需要交换）：
```php
       min
        ↓
0   1   2   6   9   3   5   4   8   7
        ↑

```
重复上述过程，直到最后一个元素就完成了排序。
```php
                   min
                    ↓
0   1   2   6   9   3   5   4   8   7
            ↑       ↑
            └───────┘

                           min
                            ↓
0   1   2   3   9   6   5   4   8   7
                ↑           ↑
                └───────────┘

                       min
                        ↓
0   1   2   3   4   6   5   9   8   7
                    ↑   ↑
                    └───┘


                       min
                        ↓
0   1   2   3   4   5   6   9   8   7
                        ↑   

                                   min
                                    ↓
0   1   2   3   4   5   6   9   8   7
                            ↑       ↑
                            └───────┘  

                               min
                                ↓
0   1   2   3   4   5   6   7   8   9
                                ↑      

                                   min
                                    ↓
0   1   2   3   4   5   6   7   8   9
                                    ↑

```

![](http://bubkoo.qiniudn.com/Selection-Sort-Animation.gif)

### PHP实现

```php
<?php

//交换坐标位置
function swap(&$arr, $index1, $index2) {
    $temp           = $arr[$index2];
    $arr[$index2]   = $arr[$index1];
    $arr[$index1]   = $temp;
}


function select_sort($arr) {
    $length     = count($arr);
    $minIndex   = 0;
    
    for($i = 0; $i < $length - 1; $i++) {
        $minIndex = $i;
        
        for ($j = $i; $j < $length; $j++) {
            if ($arr[$j] < $arr[$minIndex]) $minIndex = $j;
        }
        
        if ($i != $minIndex) {
            swap($arr, $i, $minIndex);
        }
    }
    
    return $arr;
}

//Example
$arr = array(1,42,12,412,124,2,1,24,12,131);
print_r(select_sort($arr));
```


## 插入排序（Insert）

### 原理

> 设有一组关键字｛K1， K2，…， Kn｝；排序开始就认为 K1 是一个有序序列；让 K2 插入上述表长为 1 的有序序列，使之成为一个表长为 2 的有序序列；然后让 K3 插入上述表长为 2 的有序序列，使之成为一个表长为 3 的有序序列；依次类推，最后让 Kn 插入上述表长为 n-1 的有序序列，得一个表长为 n 的有序序列。

### 举个栗子

现有一组数组 arr = [5, 6, 3, 1, 8, 7, 2, 4]，共有八个记录，排序过程如下：

```php
[5]   6   3   1   8   7   2   4
  ↑   │
  └───┘

[5, 6]   3   1   8   7   2   4
↑        │
└────────┘

[3, 5, 6]  1   8   7   2   4
↑          │
└──────────┘

[1, 3, 5, 6]  8   7   2   4
           ↑  │
           └──┘

[1, 3, 5, 6, 8]  7   2   4
            ↑    │
            └────┘

[1, 3, 5, 6, 7, 8]  2   4
   ↑                │
   └────────────────┘

[1, 2, 3, 5, 6, 7, 8]  4
         ↑             │
         └─────────────┘

[1, 2, 3, 4, 5, 6, 7, 8]

```
![enter image description here](http://bubkoo.qiniudn.com/Insertion-sort-example-300px.gif)

### PHP实现
```php
<?php

//交换指定坐标位置
function swap(&$arr, $index1, $index2) {
    $temp           = $arr[$index2];
    $arr[$index2]   = $arr[$index1];
    $arr[$index1]   = $temp;
}

function insert_sort($arr) {
    $length = count($arr);
    
    for ($i = 1; $i < $length; $i++) {
        for ($j = $i; $j > 0; $j--) {
            if ($arr[$j - 1] > $arr[$j]) swap($arr, $j - 1, $j);
        }
    }
    
    return $arr;
}

//Example
$arr = array(1,42,12,412,124,2,1,24,12,131);
print_r(insert_sort($arr));
```


