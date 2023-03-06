---
layout: post
title: 'Binary Search Algorithm'
data: 2023-03-05
author: 刘浩光
cover: 'assets/images/binary-search.png'
tags: 算法
---

## 二分查找算法的 C++ 实现与 STL 库调用实现

二分查找（折半查找）即二分搜索，是一种在**有序数组**中查找某一特定元素的搜索算法。二分查找有两个要求，一个是数列有序，另一个是数列使用顺序存储结构，例如数组。其时间复杂度为 $O(log_2 n)$。

使用 C++ 实现代码如下

```C++
#include <iostream>
using namespace std;

// 迭代二分查找
int iterationBinarySearch(int *a, int x) {
  	int left = 0;
    int right = n - 1;
    while(left <= right) {
        int mid = (left + right) / 2;
        if(x > a[mid]) left = mid + 1;
        else if(x < a[mid]) right = mid - 1;
      	else return mid;
    }
    return -1;
}

// 递归二分查找
int recurisonBinarySearch(int *a, int left, int right, int x) {
    if(left > right) return -1;
    int mid = (left + right) / 2;
  	if(x > a[mid]) return recurisonBinarySearch(a, mid + 1, right, x);
  	else if(x < a[mid]) return recurisonBinarySearch(a, left, mid - 1, x);
    else return mid;
}

int main() {
  	int a[10] = {1, 5, 9, 14, 17, 21, 25, 30, 37, 40};
  	iterationBinarySearch(a, 30);
  	recurisonBinarySearch(a, 0, 9, 30);
  	return 0;
}
```

在 STL 中，也提供了相应的函数接口，使用前进行头文件引入：`#include <algorithm>`

```C++
#include <iostream>
#include <algorithm>
using namespace std;

int main(){
  	int a[10] = {1, 5, 9, 14, 17, 21, 25, 30, 37, 40};
  	binary_search(a, a+9, 30);	// 参数列表（首地址，尾地址，查找元素），返回bool类型
  	
  	lower_bound(a, a+9, 9);		// 返回第一个大于或等于某元素的位置(地址)，pos-a 为所需要的索引
  	upper_bound(a, a+9, 9);		// 返回第一个大于某元素的位置(地址)，pos-a 为所需要的索引
  	return 0;
}
```

`lower_bound` 和 `upper_bound` 都在 **前闭后开** 的区间中进行二分查找，如果所有元素均小于等于（lower）或小于（upper）该元素，则返回 last 位置。（***注意：*** 数组会发生越界）
