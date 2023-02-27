# 二分查找法之细节


<!--more-->

Donald Knuth（高德纳）曾经说过二分法思路很简单，细节是魔鬼。本文就来探讨下二分法中的那些“魔鬼”细节

#### Reference

[力扣-二分查找Leetbook](https://leetcode-cn.com/leetbook/read/binary-search/)

[知乎-谈二分查找-小范同学](https://zhuanlan.zhihu.com/p/441033451)

## 二分查找通用框架

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while (...) {
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

开门见山，下图就是三种主要模板的框架

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

**本模板while条件采用 <= 符号，索引初始的搜索区间右端点应为<mark>nums.length-1</mark>**

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
    while (left <= right) {
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
    while (left <= right) {
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

返回`left `或者 `right`, 因为此时`left == right`,其含义是`nums`中小于`target`的值数量，所以函数返回值的取值区间为`[0, num.length]`， 所以最后要做检查

```cpp
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length;
    while (left < right) { // 注意
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
    while (left < right) { 
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

    // 或者以下统一形式
    return (left != nums.length && nums[left] == target) ? left : -1;
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
    while (left < right) { 
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

## 若干条经验

- 当搜寻单个值时，用模板一，更易于理解，每次二分都直接进行了判断，

- 当搜寻某个最值或者区间端点时，用模板二

- 当使用模板一时，`whie(left <= right)`常常与`right = size() - 1` 连用

- 当用模板二时，`while (left < right)`常常与 `right = size() `连用（不确定，有时也看情况）

- 当使用模板二时，`int mid = left + (right - left) / 2` 与 `left = mid + 1`连用，避免死循环，同理`int mid = left + (right - left + 1) / 2`与`left = mid``连用

- 当使用模板二时，考虑采用`left = mid + 1` 还是` left = mid` 时，可以做如下思考：当前基于`mid`判断的`nums[mid]`**是否可能**出现在结果区间内，如果可能，考虑二分的那一侧应该包含`mid`, 即`left = mid `或` right = mid`,同理，当不可能出现在结果区间内，则应坚决排除，即`left = mid + 1 `或` right  = mid + 1`

## 实例研究

### 单一值查找情况

[力扣-二分查找](https://leetcode-cn.com/problems/binary-search/)

#### 模板一

初始时`right`必须是`len - 1`, 否则在`left = mid + 1`时，可能取到`left == right` 且` left == len`， 访问越界数组!

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int len = nums.size();
        int left = 0, right = len - 1; 
        while (left <= right) {
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
};
```

#### 模板二

注意： 下面文字说明中 ”=“ 不表示赋值

下面这种情况，初始时`right = len - 1`, 结束条件为`left == right`

- 当`target`大于区间所有元素时，搜索区间右端点始终固定为`len - 1`
  
  - 若最后搜索区间缩小到`[left, right]`，`left = right - 1`, 这时，`mid = left`, 下一步`left = mid + 1 = right = len - 1`, 然后有 `left == right`, 搜索结束。不会出现越界的情况

- 当`target`小于区间所有元素时，搜索区间左端点始终固定为0
  
  - 若最后搜素区间缩小到`[left, right]`，`left = right - 1,` 这时，`mid = left`, 下一步`right = mid = left`,  此时`left = right`, 搜索结束。也不会出现越界
  
  上述两种情况，最后都是`left = right`时结束，但`left`下标的值却没有验证检查是否等于`target`，所以最后要做一个补充检查。此时`left`取值范围在`[0, len - 1]`, 很安全，所以直接取值判断即可。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int len = nums.size();
        int left = 0, right = len - 1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return nums[left] == target ? left : -1;
    }
};
```

下面情况， 我们将`right`初始值取为`len`，看看会发生什么变化

- 当`target`大于区间所有元素时，搜索区间右端点始终<mark>固定为`len`</mark>
  
  - 若最后搜索区间缩小到`[left, right]`，`left = right - 1`, 这时，`mid = left`, 下一步`left = mid + 1 = right = len`, 然后有 `left == right`, 搜索结束。注意这时`left = len`，下标已经越界了, 如果最后直接取`nums[left]`会访问越界！

- 当`target`小于区间所有元素时，搜索区间左端点始终固定为0
  
  - 若最后搜素区间缩小到`[left, right]`，`left = right - 1,` 这时，`mid = left`, 下一步`right = mid = left`, 此时`left = right`, 搜索结束。也不会出现越界。
  
  上述两种情况，最后都是`left = right`时结束，但`left`下标的值却没有验证检查是否等于`target`，同样最后要做一个补充检查，但是补充检查时，因为`left`取值范围为`[0, len]`, 取值范围不安全，所以要多一个排除条件`left != len`。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int len = nums.size();
        int left = 0, right = len;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return (left != len && nums[left] == target) left : -1;
    }
};
```

再看看下面的代码，又在上面的代码上做了点小小的改动，即每次判断后下一搜索区间的取值范围，这里实际上把`nums[mid] > target`条件下的`right`更新为了`mid - 1`, 可以看出没有丝毫影响，因为我们要找的是`target`, 当指明了`target`小于某个值，当然可以跳过该`mid`索引了，想一下，当`nums[mid] == target`时，我们为什么要取`=mid`, 因为满足此条件的任何`mid`索引对应的值都有可能出现在结果区间中，我们不能用`left = mid + 1 `或 `right = mid - 1`来跳过。那为什么不能取`left=mid`，这就是前文所述的死循环问题了，与`mid`取值有关，不做赘述。

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int len = nums.size();
        int left = 0, right = len;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                right = mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid - 1;
            }
        }
        return (left != len && nums[left] == target) ? left : -1;
    }
};
```

##### 小结

可以看出对于寻找单一值得情况，模板二中初始值 `right = len` 还是 `right = len - 1`，都是可以找到结果得，只是最后`left`的取值区间不一致，当`right`取`len`时，`left`取值空间包括了`len`，应该在后处理中做出排除。

当搜索区间断点时，比如以下搜索出现数字的最小位置和最大位置的代码：

```cpp
int binarySearch(vector<int>& nums, int target) {
    int left = 0, right = nums.size();
    while(left < right) { 
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid; 
        }
    }
    return (left != nums.size() && nums[left] == target) left : -1;
}
```

下面代码中，区别在于当`nums[mid] = target`时，所采取的动作，当搜索最大索引时，收紧左边界，但是由于避免死循环，所以`left = mid + 1`,导致结果区间中，`left `为 最大目标索引的下一个位置，所以目标索引应该取`left - 1` 或`right - 1`,又由于left的取值区间为`[0, nums.size()]` ， 但我们只需关心`left - 1`的取值区间，即`[-1, nums.size() - 1]`,发现这种情况相对于搜索最小位置的代码而言，自然优化了区间右端点，却导致了左端点越界，所以要额外检查左端点越界的情况。

```cpp
int binarySearch(vector<int>& nums, int target) {
    int left = 0, right = nums.size();
    while(left < right) { 
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) {
            left = mid + 1; 
        } else (nums[mid] > target) {
            right = mid; 
        }
    }
    return (left != 0 && nums[left - 1] == target) ? (left - 1) : -1; 
}
```

所以我们又可以考虑，当搜索右端点时，right初始化为nums.size() - 1时，要作何改变，代码如下

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0, right = nums.size() - 1;
        if (left == right) return nums[0] == target ? 0 : -1;
        while(left < right) { 
            int mid = left + (right - left) / 2;
            if (nums[mid] <= target) {
                left = mid + 1; 
            } else {
                right = mid; 
            }
        }
        if (left > 0 &&  nums[left - 1] == target) {
            return left - 1;
        } 
        if ((left == nums.size() - 1) && nums[left] == target) {
            return left;
        }
        return -1; 
    }
};
```

因为，`left`最终取值区间为`[0, nums.size() - 1]`, 我们区间右端点为`left - 1`, 则，`left - 1`的取值区间为`[-1, nums.size() - 2]`, 可见出来处理越界操作外，还出现了一个问题就是，`nums.size() - 1 `索引对应的值，会被漏掉，因此要额外判断。为什么会被漏掉呢？我们可以这么想，上述搜索等于目标值的最大索引，实际上是最后得到的是目标值的下一个位置，也就是大于目标值的第一个元素索引。所以当目标值比较大时，`right`始终固定为`nums.size() - 1,` `left `一直往右移动直到`left = mid + 1 = nums.size() - 1`时， 有`left = right` ，搜索结束，注意，此时`nums.size() - 1 `索引位置的值我们并没有和`target`做过判断，就这么漏掉了，所以后处理要补充检查最后一个元素，当然右边不可能越界了。

由此，我们可以总结，在使用 `left < right`时，若采取`right` 初始化为`num.size()` 的形式，还是`num.size() - 1`的形式区别在于是否要在后处理中做额外的区间越界检查。所以我建议在寻找单个值而不是端点时，尽量初始化为`nums.size() - 1`, 可以免去越界检查，而在寻找端点时，应该取`nums.size()`，避免搜索右端点时额外的检查。

