# 回溯法


<!--more-->

## 回溯法的本质

回溯法的本质是对穷举法的一个改进，但和穷举法不同的是，回溯法每次只构造候选解的一个分量，然后评估这个**部分构造解**：如果加上剩下的分量也不能求得一个解，就不会生成剩下得分量。最坏得情况下，依然存在解空间指数爆炸的问题，但这种方法使得我们至少可以对某些组合难题的较大实例进行求解。

回溯法是以构造一棵**状态空间树**为基础的，树的根节点代表了查找解之前的最初状态。树的第一层节点代表了对解的第一个分量所做的选择，第二层节点代表了对解的第二个分量所作的选择。以此类推，如果一个部分构造解仍然有可能导致一个完整解，我们说这个部分解在树中的相应节点是有希望的；否则，我们说它是没有希望的。叶子则要么代表了没希望的死胡同，要么代表了算法找到的完整解。

大多数情况下，一个回溯算法的状态空间树是按照**深度优先**的方式来构造的。如果当前节点是有希望的，通过向部分解添加下一个分量的第一个合法选择，就生成了节点的第一个子女，而处理也会转向这个子女节点。如果当前节点变得没希望了，该算法回溯到该节点的父母，考虑部分解的最后一个分量下一个可能的选择。如果这种解不存在，它再回溯到树的上一层，以此类推。最后，如果该算法找到了问题的一个完整解，它要么就停止了（如果只需要一个解）,要么继续查找其他可能的解。

## 回溯法框架

回溯法，一般可以解决如下几种问题：

- 组合问题：N个数里面按一定规则找出k个数的集合
- 切割问题：一个字符串按一定规则有几种切割方式
- 子集问题：一个N个数的集合里有多少符合条件的子集
- 排列问题：N个数按一定规则全排列，有几种排列方式
- 棋盘问题：N皇后，解数独等等

其大致框架为：

定义一个回溯函数：

```cpp
// 参数要修改就应该用引用传递
// 一般没有返回值
void backtrack(参数) {
    // 回溯函数本身就是一个递归函数
    // 所以在函数开头就要定义递归出口
    if (终止条件) {
        存放结果; // 
        return;
    }
    // 开始遍历解空间
    for (选择：状态空间树本层中的元素（树中当前节点的子节点数量）) {
        // 处理节点
        backtrack(参数, 修改后的参数);  // 递归
        // 回溯，撤销处理结果 
    }
}
```

这里面有几个细节：

1. 终止条件的选择，一般情况下都是当前解的每个分量都选择完了，就说明到达了终点，因为没有下一个解分量了。如果返回的结果要求包含解，那么我们可以用存放解的容器的长度来判断即可。当我们不需要保留解集时，我们可以专门定义一个变量指示当前进行选择的是第几个分量（或者说是状态空间树的第几层）。

2. 处理节点时后，代表当前解的分量已经做出了选择，接下来进入递归函数，在递归函数中要更新的参数是最关键的，根据不同的场景，更新不同的参数，因为在回溯阶段需要撤销所有的更改，也就意味退回到当前解分量的选择，选择其他的值作为当前解的分量。所以递归函数中有的参数若不是引用的话，天然地就是值传递，并不改变原来的值，那么实现了自动回溯，这类值往往不是我们要保存的值。我们要保存修改的值一般都是要使用引用传递的，那么回溯阶段，就需要做一些pop的操作了。

**状态空间树的高度就是解的分量数目，每个分量可选择的范围就是该分量对应层的宽度**

## 经典例题

### 组合问题

如果是**一个集合**来求组合的话，也就是解的各个分量的取值集合是同一个的话，就需要startIndex，例如LeetCode77和216

如果是**多个集合**取组合，各个集合之间相互不影响，那么就不用startIndex，例如LeetCode17



> 例题1： 给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。

```cpp
class Solution {
public:

    void backtrack(vector<vector<int>>& res, vector<int>& path, 
                    int startIndex, int n, int k) {
        if (path.size() == k) {
            res.push_back(path);
            return;
 
       }
        // 每次从startIndex开始, 下次递归startIndex加一
        // 那么下一个解分量选择就排除startIndex这个元素了
        for (int i = startIndex; i <= n - (k - path.size()) + 1; ++i) {
            path.push_back(i);
            backtrack(res, path, i + 1, n, k);
            path.pop_back();
        }
    }

    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> res;
        vector<int> path;
        backtrack(res, path, 1, n, k);
        return res;
    }
};
```

### 排列问题

> 例题2 ： 给定一个可包含重复数字的序列 nums ，按任意顺序 返回所有不重复的全排列。

```cpp
class Solution {
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> path;
        vector<bool> used(nums.size(), false);
        sort(nums.begin(), nums.end());
        backtrack(res, path, nums, used);
        return res;
    }

    void backtrack(vector<vector<int>>& res, vector<int>& path, 
                vector<int>& nums, vector<bool>& used) {
        if (path.size() == nums.size()) {
            res.emplace_back(path);
            return;
        } 
        // 使用used数组来检测是否已选择某个元素
        // 前一个元素没选取代表是从当前层横向遍历
        // 前一个元素被选取了代表是往下一层遍历
        for (int i = 0; i < nums.size(); ++i) {
            if (used[i] || i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            used[i] = true;
            path.emplace_back(nums[i]);
            backtrack(res, path, nums, used);
            path.pop_back();
            used[i] = false;
        }
    }
};
```

