---
title: Numpy基础学习
date: 2019-05-20 16:50:45
tags: [Python学习, Numpy]
categories: Numpy学习
---

# NumPy 常见问题(难度水平中等)

### 1. 如何垂直叠加两个数组？

 **给定：**

```python
a = np.arange(10).reshape(2,-1)
b = np.repeat(1, 10).reshape(2,-1)
```

**期望的输出：**

```python
# > array([[0, 1, 2, 3, 4],
# >        [5, 6, 7, 8, 9],
# >        [1, 1, 1, 1, 1],
# >        [1, 1, 1, 1, 1]])
```

**答案：**

```python
a = np.arange(10).reshape(2,-1)
b = np.repeat(1, 10).reshape(2,-1)

# Answers
# Method 1:
np.concatenate([a, b], axis=0)

# Method 2:
np.vstack([a, b]) # v 表示vertical

# Method 3:
np.r_[a, b]
# > array([[0, 1, 2, 3, 4],
# >        [5, 6, 7, 8, 9],
# >        [1, 1, 1, 1, 1],
# >        [1, 1, 1, 1, 1]])
```



### 2. 如何水平叠加两个数组？

**问题：**将数组a和数组b水平堆叠。

**给定：**

```python
a = np.arange(10).reshape(2,-1)
b = np.repeat(1, 10).reshape(2,-1)
```

**期望的输出：**

```python
# > array([[0, 1, 2, 3, 4, 1, 1, 1, 1, 1],
# >        [5, 6, 7, 8, 9, 1, 1, 1, 1, 1]])
```

**答案：**

```python
a = np.arange(10).reshape(2,-1)
b = np.repeat(1, 10).reshape(2,-1)

# Answers
# Method 1:
np.concatenate([a, b], axis=1)

# Method 2:
np.hstack([a, b]) # h 表示horizon

# Method 3:
np.c_[a, b]
# > array([[0, 1, 2, 3, 4, 1, 1, 1, 1, 1],
# >        [5, 6, 7, 8, 9, 1, 1, 1, 1, 1]])
```



### 3. 如何在无硬编码的情况下生成Numpy中的自定义序列？

**问题：**创建以下模式而不使用硬编码。只使用Numpy函数和下面的输入数组a。

**给定：**

```python
a = np.array([1,2,3])`
```

**期望的输出：**

```python
# > array([1, 1, 1, 2, 2, 2, 3, 3, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3])
```

**答案：**

```python
np.r_[np.repeat(a, 3), np.tile(a, 3)]
# > array([1, 1, 1, 2, 2, 2, 3, 3, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3])
```



### 4. 如何获取两个Numpy数组之间的公共项？

**问题：**获取数组a和数组b之间的公共项。

**给定：**

```python
a = np.array([1,2,3,2,3,4,3,4,5,6])
b = np.array([7,2,10,2,7,4,9,4,9,8])
```

**期望的输出：**

```python
array([2, 4])
```

**答案：**

```python
a = np.array([1,2,3,2,3,4,3,4,5,6])
b = np.array([7,2,10,2,7,4,9,4,9,8])
np.intersect1d(a,b)
# > array([2, 4])
```



### 5. 如何从一个数组中删除存在于另一个数组中的项？

**问题：**从数组a中删除数组b中的所有项。

**给定：**

```python
a = np.array([1,2,3,4,5])
b = np.array([5,6,7,8,9])
```

**期望的输出：**

```python
array([1,2,3,4])
```

**答案：**

```python
a = np.array([1,2,3,4,5])
b = np.array([5,6,7,8,9])

# From 'a' remove all of 'b'
np.setdiff1d(a,b)
# > array([1, 2, 3, 4])
```



### 6. 如何得到两个数组元素匹配的位置？

**问题：**获取a和b元素匹配的位置。

**给定：**

```python
a = np.array([1,2,3,2,3,4,3,4,5,6])
b = np.array([7,2,10,2,7,4,9,4,9,8])
```

**期望的输出：**

```python
# > (array([1, 3, 5, 7]),)
```

**答案：**

```python
a = np.array([1,2,3,2,3,4,3,4,5,6])
b = np.array([7,2,10,2,7,4,9,4,9,8])

np.where(a == b)
# > (array([1, 3, 5, 7]),)
```



### 7. 如何从Numpy数组中提取给定范围内的所有数字？

**问题：**获取5到10之间的所有项目。

**给定：**

```python
a = np.array([2, 6, 1, 9, 10, 3, 27])
```

**期望的输出：**

```python
(array([6, 9, 10]),)
```

**答案：**

```python
a = np.arange(15)

# Method 1
index = np.where((a >= 5) & (a <= 10))
a[index]

# Method 2:
index = np.where(np.logical_and(a>=5, a<=10))
a[index]
# > (array([6, 9, 10]),)

# Method 3: (thanks loganzk!)
a[(a >= 5) & (a <= 10)]
```



### 8. 如何创建一个python函数来处理scalars并在Numpy数组上工作？

**问题：**转换适用于两个标量的函数maxx，以处理两个数组。

**给定：**

```python
def maxx(x, y):
    """Get the maximum of two items"""
    if x >= y:
        return x
    else:
        return y

maxx(1, 5)
# > 5
```

**期望的输出：**

```python
a = np.array([5, 7, 9, 8, 6, 4, 5])
b = np.array([6, 3, 4, 8, 9, 7, 1])
pair_max(a, b)
# > array([ 6.,  7.,  9.,  8.,  9.,  7.,  5.])
```

**答案：**

```python
def maxx(x, y):
    """Get the maximum of two items"""
    if x >= y:
        return x
    else:
        return y

pair_max = np.vectorize(maxx, otypes=[float])

a = np.array([5, 7, 9, 8, 6, 4, 5])
b = np.array([6, 3, 4, 8, 9, 7, 1])

pair_max(a, b)
# > array([ 6.,  7.,  9.,  8.,  9.,  7.,  5.])
```



### 9. 如何交换二维numpy数组中的两列？

**问题：**在数组arr中交换列1和2。

**给定：**

```python
arr = np.arange(9).reshape(3,3)
arr
```

**答案：**

```python
# Input
arr = np.arange(9).reshape(3,3)

# Solution
arr[:, [1,0,2]]
# > array([[1, 0, 2],
# >        [4, 3, 5],
# >        [7, 6, 8]])
```



### 10. 如何交换二维numpy数组中的两行？

**问题：**交换数组arr中的第1和第2行：

**给定：**

```python
arr = np.arange(9).reshape(3,3)
arr
```

**答案：**

```python
# Input
arr = np.arange(9).reshape(3,3)

# Solution
arr[[1,0,2], :]
# > array([[3, 4, 5],
# >        [0, 1, 2],
# >        [6, 7, 8]])
```



### 11. 如何反转二维数组的行？

**问题：**反转二维数组arr的行。

**给定：**

```python
# Input
arr = np.arange(9).reshape(3,3)
```

**答案：**

```python
# Input
arr = np.arange(9).reshape(3,3)
# Solution
arr[::-1]
array([[6, 7, 8],
       [3, 4, 5],
       [0, 1, 2]])
```



### 12. 如何反转二维数组的列？

**问题：**反转二维数组arr的列。

**给定：**

```python
# Input
arr = np.arange(9).reshape(3,3)
```

**答案：**

```python
# Input
arr = np.arange(9).reshape(3,3)

# Solution
arr[:, ::-1]
# > array([[2, 1, 0],
# >        [5, 4, 3],
# >        [8, 7, 6]])
```



### 13. 如何创建包含5到10之间随机浮动的二维数组？

**问题：**创建一个形状为5x3的二维数组，以包含5到10之间的随机十进制数。

**答案：**

```python
# Input
arr = np.arange(9).reshape(3,3)

# Solution Method 1:
rand_arr = np.random.randint(low=5, high=10, size=(5,3)) + np.random.random((5,3))
# print(rand_arr)

# Solution Method 2:
rand_arr = np.random.uniform(5,10, size=(5,3))
print(rand_arr)
# > [[ 8.50061025  9.10531502  6.85867783]
# >  [ 9.76262069  9.87717411  7.13466701]
# >  [ 7.48966403  8.33409158  6.16808631]
# >  [ 7.75010551  9.94535696  5.27373226]
# >  [ 8.0850361   5.56165518  7.31244004]]
```



### 14. 如何导入数字和文本的数据集保持文本在numpy数组中完好无损？

**问题：**导入鸢尾属植物数据集，保持文本不变。

**答案：**

```python
# Solution
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object') 
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')

# Print the first 3 rows
iris[:3]
# > array([[b'5.1', b'3.5', b'1.4', b'0.2', b'Iris-setosa'],
# >        [b'4.9', b'3.0', b'1.4', b'0.2', b'Iris-setosa'],
# >        [b'4.7', b'3.2', b'1.3', b'0.2', b'Iris-setosa']], dtype=object)
```



### 15. 如何从1维元组数组中提取特定列？

**问题：**从前面问题中导入的一维鸢尾属植物数据集中提取文本列的物种。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_1d = np.genfromtxt(url, delimiter=',', dtype=None)
```

**答案：**

```python
# 给定：
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_1d = np.genfromtxt(url, delimiter=',', dtype=None, encoding=None) # None代表默认编码
print(iris_1d.shape)

# Solution:
species = np.array([row[4] for row in iris_1d])
species[:5]
# > (150,)
# > array(['Iris-setosa', 'Iris-setosa', 'Iris-setosa', 'Iris-setosa',
# >        'Iris-setosa'], dtype='<U15')
```



### 16. 如何将1维元组数组转换为2维numpy数组？

**问题：**通过省略鸢尾属植物数据集种类的文本字段，将一维鸢尾属植物数据集转换为二维数组iris_2d。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_1d = np.genfromtxt(url, delimiter=',', dtype=None)
```

**答案：**

```python
# **给定：**
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_1d = np.genfromtxt(url, delimiter=',', dtype=None)

# Solution:
# Method 1: Convert each row to a list and get the first 4 items
iris_2d = np.array([row.tolist()[:4] for row in iris_1d])
iris_2d[:4]

# Alt Method 2: Import only the first 4 columns from source url
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
iris_2d[:4]
# > array([[ 5.1,  3.5,  1.4,  0.2],
# >        [ 4.9,  3. ,  1.4,  0.2],
# >        [ 4.7,  3.2,  1.3,  0.2],
# >        [ 4.6,  3.1,  1.5,  0.2]])
```



### 17. 如何规范化数组，使数组的值正好介于0和1之间？

**问题：**创建一种标准化形式的鸢尾属植物间隔长度，其值正好介于0和1之间，这样最小值为0，最大值为1。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
sepallength = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0])
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
sepallength = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0])

# Solution
Smax, Smin = sepallength.max(), sepallength.min()
S = (sepallength - Smin)/(Smax - Smin)
# or 
S = (sepallength - Smin)/sepallength.ptp()  # Thanks, David Ojeda!
print(S)
# > [ 0.222  0.167  0.111  0.083  0.194  0.306  0.083  0.194  0.028  0.167
# >   0.306  0.139  0.139  0.     0.417  0.389  0.306  0.222  0.389  0.222
# >   0.306  0.222  0.083  0.222  0.139  0.194  0.194  0.25   0.25   0.111
# >   0.139  0.306  0.25   0.333  0.167  0.194  0.333  0.167  0.028  0.222
# >   0.194  0.056  0.028  0.194  0.222  0.139  0.222  0.083  0.278  0.194
# >   0.75   0.583  0.722  0.333  0.611  0.389  0.556  0.167  0.639  0.25
# >   0.194  0.444  0.472  0.5    0.361  0.667  0.361  0.417  0.528  0.361
# >   0.444  0.5    0.556  0.5    0.583  0.639  0.694  0.667  0.472  0.389
# >   0.333  0.333  0.417  0.472  0.306  0.472  0.667  0.556  0.361  0.333
# >   0.333  0.5    0.417  0.194  0.361  0.389  0.389  0.528  0.222  0.389
# >   0.556  0.417  0.778  0.556  0.611  0.917  0.167  0.833  0.667  0.806
# >   0.611  0.583  0.694  0.389  0.417  0.583  0.611  0.944  0.944  0.472
# >   0.722  0.361  0.944  0.556  0.667  0.806  0.528  0.5    0.583  0.806
# >   0.861  1.     0.583  0.556  0.5    0.944  0.556  0.583  0.472  0.722
# >   0.667  0.722  0.417  0.694  0.667  0.667  0.556  0.611  0.528  0.444]
```



### 18. 如何在数组中的随机位置插入值？

**问题：**在iris_2d数据集中的20个随机位置插入np.nan值

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='object')
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='object')

# Method 1
i, j = np.where(iris_2d)

# i, j contain the row numbers and column numbers of 600 elements of iris_x
np.random.seed(100)
iris_2d[np.random.choice((i), 20), np.random.choice((j), 20)] = np.nan

# Method 2
np.random.seed(100)
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan

# Print first 10 rows
print(iris_2d[:10])
# > [[b'5.1' b'3.5' b'1.4' b'0.2' b'Iris-setosa']
# >  [b'4.9' b'3.0' b'1.4' b'0.2' b'Iris-setosa']
# >  [b'4.7' b'3.2' b'1.3' b'0.2' b'Iris-setosa']
# >  [b'4.6' b'3.1' b'1.5' b'0.2' b'Iris-setosa']
# >  [b'5.0' b'3.6' b'1.4' b'0.2' b'Iris-setosa']
# >  [b'5.4' b'3.9' b'1.7' b'0.4' b'Iris-setosa']
# >  [b'4.6' b'3.4' b'1.4' b'0.3' b'Iris-setosa']
# >  [b'5.0' b'3.4' b'1.5' b'0.2' b'Iris-setosa']
# >  [b'4.4' nan b'1.4' b'0.2' b'Iris-setosa']
# >  [b'4.9' b'3.1' b'1.5' b'0.1' b'Iris-setosa']]
```



### 19. 如何在numpy数组中找到缺失值的位置？

**问题：**在iris_2d的sepallength中查找缺失值的数量和位置（第1列）

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float')
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan

# Solution
print("Number of missing values: \n", np.isnan(iris_2d[:, 0]).sum())
print("Position of missing values: \n", np.where(np.isnan(iris_2d[:, 0])))
# > Number of missing values: 
# >  5
# > Position of missing values: 
# >  (array([ 39,  88,  99, 130, 147]),)
```



### 20. 如何找到numpy数组的两列之间的相关性？

**问题：**在iris_2d中找出SepalLength（第1列）和PetalLength（第3列）之间的相关性

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])

# Solution 1
np.corrcoef(iris[:, 0], iris[:, 2])[0, 1]

# Solution 2
from scipy.stats.stats import pearsonr  
corr, p_value = pearsonr(iris[:, 0], iris[:, 2])
print(corr)

# Correlation coef indicates the degree of linear relationship between two numeric variables.
# It can range between -1 to +1.

# The p-value roughly indicates the probability of an uncorrelated system producing 
# datasets that have a correlation at least as extreme as the one computed.
# The lower the p-value (<0.01), stronger is the significance of the relationship.
# It is not an indicator of the strength.
# > 0.871754157305
```



### 21. 如何查找给定数组是否具有任何空值？

**问题：**找出iris_2d是否有任何缺失值。

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])

np.isnan(iris_2d).any()
# > False
```



### 22. 如何在numpy数组中用0替换所有缺失值？

**问题：**在numpy数组中将所有出现的nan替换为0

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0,1,2,3])
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan

# Solution
iris_2d[np.isnan(iris_2d)] = 0
iris_2d[:4]
# > array([[ 5.1,  3.5,  1.4,  0. ],
# >        [ 4.9,  3. ,  1.4,  0.2],
# >        [ 4.7,  3.2,  1.3,  0.2],
# >        [ 4.6,  3.1,  1.5,  0.2]])
```



### 23. 如何在numpy数组中查找唯一值的计数？

**问题：**找出鸢尾属植物物种中的独特值和独特值的数量

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**答案：**

```python
# Import iris keeping the text column intact
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')

# Solution
# Extract the species column as an array
species = np.array([row.tolist()[4] for row in iris])

# Get the unique values and the counts
np.unique(species, return_counts=True)
# > (array([b'Iris-setosa', b'Iris-versicolor', b'Iris-virginica'],
# >        dtype='|S15'), array([50, 50, 50]))
```



### 24. 如何将数字转换为分类（文本）数组？

**问题：**将iris_2d的花瓣长度（第3列）加入以形成文本数组，这样如果花瓣长度为：

- Less than 3 --> 'small'
- 3-5 --> 'medium'
- '>=5 --> 'large'

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')

# Bin petallength 
petal_length_bin = np.digitize(iris[:, 2].astype('float'), [0, 3, 5, 10])

# Map it to respective category
label_map = {1: 'small', 2: 'medium', 3: 'large', 4: np.nan}
petal_length_cat = [label_map[x] for x in petal_length_bin]

# View
petal_length_cat[:4]
<# > ['small', 'small', 'small', 'small']
```



### 25. 如何从numpy数组的现有列创建新列？

**问题：**在iris_2d中为卷创建一个新列，其中volume是`（pi x petallength x sepal_length ^ 2）/ 3`

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris_2d = np.genfromtxt(url, delimiter=',', dtype='object')

# Solution
# Compute volume
sepallength = iris_2d[:, 0].astype('float')
petallength = iris_2d[:, 2].astype('float')
volume = (np.pi * petallength * (sepallength**2))/3

# Introduce new dimension to match iris_2d's
volume = volume[:, np.newaxis]

# Add the new column
out = np.hstack([iris_2d, volume])

# View
out[:4]
# > array([[b'5.1', b'3.5', b'1.4', b'0.2', b'Iris-setosa', 38.13265162927291],
# >        [b'4.9', b'3.0', b'1.4', b'0.2', b'Iris-setosa', 35.200498485922445],
# >        [b'4.7', b'3.2', b'1.3', b'0.2', b'Iris-setosa', 30.0723720777127],
# >        [b'4.6', b'3.1', b'1.5', b'0.2', b'Iris-setosa', 33.238050274980004]], dtype=object)
```





### 26. 如何在按另一个数组分组时获取数组的第二大值？

**问题：**第二长的物种setosa的价值是多少

**给定：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**答案：**

```python
# Import iris keeping the text column intact
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')

# Solution
# Get the species and petal length columns
petal_len_setosa = iris[iris[:, 4] == b'Iris-setosa', [2]].astype('float')

# Get the second last value
np.unique(np.sort(petal_len_setosa))[-2]
# > 1.7
```



### 27. 如何按列对2D数组进行排序

**问题：**根据sepallength列对虹膜数据集进行排序。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**答案：**

```python
# Sort by column position 0: SepalLength
print(iris[iris[:,0].argsort()][:20])
# > [[b'4.3' b'3.0' b'1.1' b'0.1' b'Iris-setosa']
# >  [b'4.4' b'3.2' b'1.3' b'0.2' b'Iris-setosa']
# >  [b'4.4' b'3.0' b'1.3' b'0.2' b'Iris-setosa']
# >  [b'4.4' b'2.9' b'1.4' b'0.2' b'Iris-setosa']
# >  [b'4.5' b'2.3' b'1.3' b'0.3' b'Iris-setosa']
# >  [b'4.6' b'3.6' b'1.0' b'0.2' b'Iris-setosa']
# >  [b'4.6' b'3.1' b'1.5' b'0.2' b'Iris-setosa']
# >  [b'4.6' b'3.4' b'1.4' b'0.3' b'Iris-setosa']
# >  [b'4.6' b'3.2' b'1.4' b'0.2' b'Iris-setosa']
# >  [b'4.7' b'3.2' b'1.3' b'0.2' b'Iris-setosa']
# >  [b'4.7' b'3.2' b'1.6' b'0.2' b'Iris-setosa']
# >  [b'4.8' b'3.0' b'1.4' b'0.1' b'Iris-setosa']
# >  [b'4.8' b'3.0' b'1.4' b'0.3' b'Iris-setosa']
# >  [b'4.8' b'3.4' b'1.9' b'0.2' b'Iris-setosa']
# >  [b'4.8' b'3.4' b'1.6' b'0.2' b'Iris-setosa']
# >  [b'4.8' b'3.1' b'1.6' b'0.2' b'Iris-setosa']
# >  [b'4.9' b'2.4' b'3.3' b'1.0' b'Iris-versicolor']
# >  [b'4.9' b'2.5' b'4.5' b'1.7' b'Iris-virginica']
# >  [b'4.9' b'3.1' b'1.5' b'0.1' b'Iris-setosa']
# >  [b'4.9' b'3.1' b'1.5' b'0.1' b'Iris-setosa']]
```



### 28.  如何找到第一次出现的值大于给定值的位置？

**问题：**在虹膜数据集的petalwidth第4列中查找第一次出现的值大于1.0的位置。

```python
# **给定：**
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
```

**答案：**

```python
# **给定：**
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')

# Solution: (edit: changed argmax to argwhere. Thanks Rong!)
np.argwhere(iris[:, 3].astype(float) > 1.0)[0]
# > 50
```



### 29. 如何将大于给定值的所有值替换为给定的截止值？

**问题：**从数组a中，替换所有大于30到30和小于10到10的值。

**给定：**

```python
np.random.seed(100)
a = np.random.uniform(1,50, 20)
```

**答案：**

```python
# Input
np.set_printoptions(precision=2)
np.random.seed(100)
a = np.random.uniform(1,50, 20)

# Solution 1: Using np.clip
np.clip(a, a_min=10, a_max=30)

# Solution 2: Using np.where
print(np.where(a < 10, 10, np.where(a > 30, 30, a)))
# > [ 27.63  14.64  21.8   30.    10.    10.    30.    30.    10.    29.18  30.
# >   11.25  10.08  10.    11.77  30.    30.    10.    30.    14.43]
```



### 30. 如何从numpy数组中获取最大n值的位置？

**问题：**获取给定数组a中前5个最大值的位置。

```python
np.random.seed(100)
a = np.random.uniform(1,50, 20)
```

**答案：**

```python
# Input
np.random.seed(100)
a = np.random.uniform(1,50, 20)

# Solution:
print(a.argsort())
# > [18 7 3 10 15]

# Solution 2:
np.argpartition(-a, 5)[:5]
# > [15 10  3  7 18]

# Below methods will get you the values.
# Method 1:
a[a.argsort()][-5:]

# Method 2:
np.sort(a)[-5:]

# Method 3:
np.partition(a, kth=-5)[-5:]

# Method 4:
a[np.argpartition(-a, 5)][:5]
```



### 31. 如何将数组转换为平面一维数组？

**问题：**将array_of_arrays转换为扁平线性1d数组。

**给定：**

```python
# **给定：**
arr1 = np.arange(3)
arr2 = np.arange(3,7)
arr3 = np.arange(7,10)

array_of_arrays = np.array([arr1, arr2, arr3])
array_of_arrays
# > array([array([0, 1, 2]), array([3, 4, 5, 6]), array([7, 8, 9])], dtype=object)
```

**期望的输出：**

```python
# > array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

**答案：**

```python
 # **给定：**
arr1 = np.arange(3)
arr2 = np.arange(3,7)
arr3 = np.arange(7,10)

array_of_arrays = np.array([arr1, arr2, arr3])
print('array_of_arrays: ', array_of_arrays)

# Solution 1
arr_2d = np.array([a for arr in array_of_arrays for a in arr])

# Solution 2:
arr_2d = np.concatenate(array_of_arrays)
print(arr_2d)
# > array_of_arrays:  [array([0, 1, 2]) array([3, 4, 5, 6]) array([7, 8, 9])]
# > [0 1 2 3 4 5 6 7 8 9]
```



### 32. 如何使用numpy对数组中的项进行排名？

**问题：**为给定的数字数组a创建排名。

**给定：**

```python
np.random.seed(10)
a = np.random.randint(20, size=10)
print(a)
# > [ 9  4 15  0 17 16 17  8  9  0]
```

**期望输出：**

```python
[4 2 6 0 8 7 9 3 5 1]
```

**答案：**

```python
np.random.seed(10)
a = np.random.randint(20, size=10)
print('Array: ', a)

# Solution
print(a.argsort().argsort())
print('Array: ', a)
# > Array:  [ 9  4 15  0 17 16 17  8  9  0]
# > [4 2 6 0 8 7 9 3 5 1]
# > Array:  [ 9  4 15  0 17 16 17  8  9  0]
```



### 33. 如何在二维numpy数组的每一行中找到最大值？

**问题：**计算给定数组中每行的最大值。

**给定：**

```python
np.random.seed(100)
a = np.random.randint(1,10, [5,3])
a
# > array([[9, 9, 4],
# >        [8, 8, 1],
# >        [5, 3, 6],
# >        [3, 3, 3],
# >        [2, 1, 9]])
```

**答案：**

```python
# Input
np.random.seed(100)
a = np.random.randint(1,10, [5,3])
a

# Solution 1
np.amax(a, axis=1)

# Solution 2
np.apply_along_axis(np.max, arr=a, axis=1)
# > array([9, 8, 6, 3, 9])
```



### 34. 如何删除numpy数组中所有缺少的值？

**问题：**从一维numpy数组中删除所有NaN值

**给定：**

```python
np.array([1,2,3,np.nan,5,6,7,np.nan])
```

**期望的输出：**

```python
array([ 1.,  2.,  3.,  5.,  6.,  7.])
```

**答案：**

```python
a = np.array([1,2,3,np.nan,5,6,7,np.nan])
a[~np.isnan(a)]
# > array([ 1.,  2.,  3.,  5.,  6.,  7.])
```



### 35. 如何从二维数组中减去一维数组，其中一维数组的每一项从各自的行中减去？

**问题：**从2d数组a_2d中减去一维数组b_1D，使得b_1D的每一项从a_2d的相应行中减去。

```python
a_2d = np.array([[3,3,3],[4,4,4],[5,5,5]])
b_1d = np.array([1,1,1]
```

**期望的输出：**

```python
# > [[2 2 2]
# >  [2 2 2]
# >  [2 2 2]]
```

**答案：**

```python
# Input
a_2d = np.array([[3,3,3],[4,4,4],[5,5,5]])
b_1d = np.array([1,2,3])

# Solution
print(a_2d - b_1d[:,None])
# > [[2 2 2]
# >  [2 2 2]
# >  [2 2 2]]
```



### 36. 如何查找数组中项的第n次重复索引？

**问题：**找出x中数字1的第5次重复的索引。

```python
x = np.array([1, 2, 1, 1, 3, 4, 3, 1, 1, 2, 1, 1, 2])
```

**答案：**

```python
x = np.array([1, 2, 1, 1, 3, 4, 3, 1, 1, 2, 1, 1, 2])
n = 5

# Solution 1: List comprehension
[i for i, v in enumerate(x) if v == 1][n-1]

# Solution 2: Numpy version
np.where(x == 1)[0][n-1]
# > 8
```



### 37. 如何将numpy的datetime 64对象转换为datetime的datetime对象？

**问题：**将numpy的`datetime64`对象转换为datetime的datetime对象

```python
# **给定：** a numpy datetime64 object
dt64 = np.datetime64('2018-02-25 22:10:10')
```

**答案：**

```python
# **给定：** a numpy datetime64 object
dt64 = np.datetime64('2018-02-25 22:10:10')

# Solution
from datetime import datetime
dt64.tolist()

# or

dt64.astype(datetime)
# > datetime.datetime(2018, 2, 25, 22, 10, 10)
```



### 38. 如何在给定起始点、长度和步骤的情况下创建一个numpy数组序列？

**问题：**创建长度为10的numpy数组，从5开始，在连续的数字之间的步长为3。

**答案：**

```python
length = 10
start = 5
step = 3

def seq(start, length, step):
    end = start + (step*length)
    return np.arange(start, end, step)

seq(start, length, step)
# > array([ 5,  8, 11, 14, 17, 20, 23, 26, 29, 32])
```

