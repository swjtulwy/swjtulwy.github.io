# 二分查找法之细节


<!--more-->

Donald Knuth（高德纳）曾经说过二分法思路很简单，细节是魔鬼。本文就来探讨下二分法中的那些“魔鬼”细节

Reference

[力扣](https://leetcode-cn.com/leetbook/read/binary-search/)

https://zhuanlan.zhihu.com/p/441033451

## 二分查找通用框架

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while(...) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...
        } else if (nums[mid] > target) {
            right = ...
        }
    }
    return ...;
}
```

分析二分查找的一个技巧是：不要出现 else，而是把所有情况用 else if 写清楚，这样可以清楚地展现所有细节。本文都会使用 else if，旨在讲清楚，读者理解后可自行简化。

其中 ... 标记的部分，就是可能出现细节问题的地方，当你见到一个二分查找的代码时，首先注意这几个地方。后文用实例分析这些地方能有什么样的变化。

另外声明一下，计算 mid 时需要防止溢出，代码中` left + (right - left) / 2 `就和 `(left + right) / 2 `的结果相同，但是有效防止了 left 和 right 太大直接相加导致溢出。

这里我提供两个小诀窍，第一是尝试熟练使用一种写法，比如左闭右开（满足 C++、Python 等语言的习惯）或左闭右闭（便于处理边界条件），尽量只保持这一种写法；第二是在刷题时思考如果最后区间只剩下一个数或者两个数，自己的写法是否会陷入死循环，如果某种写法无法跳出死循环，则考虑尝试另一种写法。

<img src="https://i0.hdslb.com/bfs/album/3fd3c02e25774c07bd65385dbb86a26dd7136ff4.png" title="" alt="" data-align="center">

这 3 个模板的不同之处在于：

左、中、右索引的分配。
循环或递归终止条件。
后处理的必要性。
模板 #1 和 #3 是最常用的，几乎所有二分查找问题都可以用其中之一轻松实现。模板 #2 更 高级一些，用于解决某些类型的问题。

这 3 个模板中的每一个都提供了一个特定的用例：

**模板 #1 (left <= right)**

二分查找的最基础和最基本的形式。
查找条件可以在不与元素的两侧进行比较的情况下确定（或使用它周围的特定元素）。
不需要后处理，因为每一步中，你都在检查是否找到了元素。如果到达末尾，则知道未找到该元素。 

**模板 #2 (left < right)**

一种实现二分查找的高级方法。
查找条件需要访问元素的直接右邻居。
使用元素的右邻居来确定是否满足条件，并决定是向左还是向右。
保证查找空间在每一步中至少有 2 个元素。
需要进行后处理。 当你剩下 1 个元素时，循环 / 递归结束。 需要评估剩余元素是否符合条件。

**模板 #3 (left + 1 < right)**

实现二分查找的另一种方法。
搜索条件需要访问元素的直接左右邻居。
使用元素的邻居来确定它是向右还是向左。
保证查找空间在每个步骤中至少有 3 个元素。
需要进行后处理。 当剩下 2 个元素时，循环 / 递归结束。 需要评估其余元素是否符合条件。

## 第一种模板

**本模板while条件采用 <= 符号，索引初始的搜索区间右端点应为nums.length-1**

### 搜索单个值

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return -1;
}
```

### 搜索左端点

 最后需要做如下越界检查，左边和右边

- `left`加到`nums.length`，此时`target`比所有值都大，**不符合**条件，所以检查`left >= nums.length ` 可直接返回-1

- `right`会减到 -1，分两种情况
  
  - (1) `target`比所有值都小, **不符合**条件， 此时`left = 0`, 但 `nums[left] != target`
  
  - (2) 第一个元素正好为`target`，**符合条件**，`right`也会减到-1,但 `nums[left] == target `

所以可以直接检查第一个元素即`nums[left]`是否为目标值来区分两种情况。

当找到`target`时，`right = mid - 1` 即目标索引 `mid = right + 1 = left` ， 所以返回`left`.

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            // 缩小右边搜索区间
            // 锁定左侧边界
            right = mid - 1; 
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    // 做检查
    if (left >= nums.length || nums[left] != target)
        return -1;
    return left;
}
```

### 搜索右端点

做如下越界检查

- `left`加到`nums.length`，分两种情况
  
  - （1）`target`比所有值都大, **不符合条件**，此时`right = nums.length - 1`,但`nums[right] != target`
  
  - （2）最后元素正好为`target`，**符合条件**，`left`也会加到`nums.length`,但`nums[right] == target`

- `right`会减到-1，此时`target`比所有值都小，**不符合条件**，所以首先检查左边`right < 0`可直接返回-1。

所以直接检查最后一个元素即`nums[right]`是否为目标值来区分上述两种情况。

当找到`target`时，`left = mid + 1 `即目标索引 `mid = left - 1 = right`， 返回`right`

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            // 缩小左边搜索区间
            // 锁定右侧边界
            left = mid + 1; 
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    // 做检查
    if (right < 0 || nums[right] != target)
        return -1;
    return right;
}
```

## 第二种模板

**本模板while条件采用 < 符号，索引初始的搜索区间右端点应为nums.length**

### 搜索单个值

返回`left `或者 `righ`, 因为此时`left == right`,其含义是`nums`种小于`target`的值数量，所以函数返回值的取值区间为`[0, num.length]`， 所以最后要做检查

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length;
    while(left < right) { // 注意
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid; 
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    // Post-processing:
    // End Condition: left == right
    if(left != nums.length && nums[left] == target) return left;
    return -1; 
}
```

### 搜索左端点

直接返回`left `是因为找到目标时，有`right == mid`, 又因为`left == right`所以返回`left`

考虑以下两种情况：

- 当 `left `一直向右移动，最终`left == nums.length`(每次`left = mid + 1`), 意味着 `target`比所有数都大。**不符合条件**,检查这种情况返回-1即可。

- 当`right`一直向左移动，最终`right == 0`, (每次`right= mid`), 意味着
  
  - `target `比所有数都小，**不符合条件**， `nums[left] != target`
  
  - 最终剩余的`nums[right]`未检查，但`nums[left] == target`, **符合条件**
  
  为了区分上述两种情况，直接检查`nums[left] == target` 即可

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length;
    while(left < right) { 
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            right = mid; 
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; 
        }
    }
    // 额外检查target比所有数都大
    if (left == nums.length) return -1;
    // 检查区间残留元素
    return nums[left] == target ? left : -1; 

    // 或者可以这样写
    // if (left != nums.length && nums[left] == target) return left;
    // return -1;
}
```

### 搜索右端点

注意这里是返回`left - 1`, 因为目标找到时有 `left = mid + 1`; 所以`mid = left - 1`;

考虑以下两种情况：

因为我们对 `left `的更新必须是 `left = mid + 1`，就是说 `while `循环结束时，`nums[left] `一定不等于 `target `了，而 `nums[left-1] `可能是 `target`

- 当 `left` 一直向右移动，最终`left == nums.length`(每次`left = mid + 1`), 意味着
  
  - `target`比所有数都大 。**不符合条件**,` nums[left - 1] != target`, 检查这种情况返回-1即可
  
  - 恰好最后一个元素等于`target`,即 `nums[left - 1] == target`, **符合条件**
  
  为了区分上述两种情况，直接检查`nums[left - 1] == target` 即可

- 当`right`一直向左移动，最终`right == 0`, (每次`right= mid`), 意味着`target` 比所有数都小，**不符合条件**， `nums[left] != target`

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length;
    while(left < right) { 
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            left = mid + 1; 
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; 
        }
    }
    if (left == 0) return -1;
    return nums[left - 1] == target ? (left - 1) : -1; 
}
```

## 第三种模板

```cpp
int binarySearch(vector<int>& nums, int target){
    if (nums.size() == 0)
        return -1;

    int left = 0, right = nums.size() - 1;
    while (left + 1 < right){
        // Prevent (left + right) overflow
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid;
        } else {
            right = mid;
        }
    }
    // Post-processing:
    // End Condition: left + 1 == right
    if(nums[left] == target) return left;
    if(nums[right] == target) return right;
    return -1;
}
```

