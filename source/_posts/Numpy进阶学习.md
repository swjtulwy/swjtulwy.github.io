---
title: Numpy进阶学习
date: 2019-05-20 20:23:06
tags: [Python学习, Numpy]
categories: Numpy学习
---

# NumPy 常见问题(难度水平偏难)

### 1. 如何计算Softmax得分？

**问题：**计算sepallength的softmax分数。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
sepallength = np.genfromtxt(url, delimiter=',', dtype='float', usecols=[0])
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
sepallength = np.array([float(row[0]) for row in iris])

# Solution
def softmax(x):
    """Compute softmax values for each sets of scores in x.
    https://stackoverflow.com/questions/34968722/how-to-implement-the-softmax-function-in-python"""
    e_x = np.exp(x - np.max(x))
    return e_x / e_x.sum(axis=0)

print(softmax(sepallength))
# > [ 0.002  0.002  0.001  0.001  0.002  0.003  0.001  0.002  0.001  0.002
# >   0.003  0.002  0.002  0.001  0.004  0.004  0.003  0.002  0.004  0.002
# >   0.003  0.002  0.001  0.002  0.002  0.002  0.002  0.002  0.002  0.001
# >   0.002  0.003  0.002  0.003  0.002  0.002  0.003  0.002  0.001  0.002
# >   0.002  0.001  0.001  0.002  0.002  0.002  0.002  0.001  0.003  0.002
# >   0.015  0.008  0.013  0.003  0.009  0.004  0.007  0.002  0.01   0.002
# >   0.002  0.005  0.005  0.006  0.004  0.011  0.004  0.004  0.007  0.004
# >   0.005  0.006  0.007  0.006  0.008  0.01   0.012  0.011  0.005  0.004
# >   0.003  0.003  0.004  0.005  0.003  0.005  0.011  0.007  0.004  0.003
# >   0.003  0.006  0.004  0.002  0.004  0.004  0.004  0.007  0.002  0.004
# >   0.007  0.004  0.016  0.007  0.009  0.027  0.002  0.02   0.011  0.018
# >   0.009  0.008  0.012  0.004  0.004  0.008  0.009  0.03   0.03   0.005
# >   0.013  0.004  0.03   0.007  0.011  0.018  0.007  0.006  0.008  0.018
# >   0.022  0.037  0.008  0.007  0.006  0.03   0.007  0.008  0.005  0.013
# >   0.011  0.013  0.004  0.012  0.011  0.011  0.007  0.009  0.007  0.005]
```



### 2. 如何根据两个或多个条件过滤numpy数组？

**问题：**过滤具有petallength（第3列）> 1.5 和 sepallength（第1列）< 5.0 的iris_2d行

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

# Solution
condition = (iris_2d[:, 2] > 1.5) & (iris_2d[:, 0] < 5.0)
iris_2d[condition]
# > array([[ 4.8,  3.4,  1.6,  0.2],
# >        [ 4.8,  3.4,  1.9,  0.2],
# >        [ 4.7,  3.2,  1.6,  0.2],
# >        [ 4.8,  3.1,  1.6,  0.2],
# >        [ 4.9,  2.4,  3.3,  1. ],
# >        [ 4.9,  2.5,  4.5,  1.7]])
```



### 3. 如何从numpy数组中删除包含缺失值的行？

**问题：**选择没有任何nan值的iris_2d行。

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
iris_2d[np.random.randint(150, size=20), np.random.randint(4, size=20)] = np.nan

# Solution
# No direct numpy function for this.
# Method 1:
any_nan_in_row = np.array([~np.any(np.isnan(row)) for row in iris_2d])
iris_2d[any_nan_in_row][:5]

# Method 2: (By Rong)
iris_2d[np.sum(np.isnan(iris_2d), axis = 1) == 0][:5]
# > array([[ 4.9,  3. ,  1.4,  0.2],
# >        [ 4.7,  3.2,  1.3,  0.2],
# >        [ 4.6,  3.1,  1.5,  0.2],
# >        [ 5. ,  3.6,  1.4,  0.2],
# >        [ 5.4,  3.9,  1.7,  0.4]])
```



### 4. 如何在numpy中进行概率抽样？

**问题：**随机抽鸢尾属植物的种类，使得刚毛的数量是云芝和维吉尼亚的两倍

**给定：**

```python
# Import iris keeping the text column intact
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
```

**答案：**

```python
# Import iris keeping the text column intact
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')

# Solution
# Get the species column
species = iris[:, 4]

# Approach 1: Generate Probablistically
np.random.seed(100)
a = np.array(['Iris-setosa', 'Iris-versicolor', 'Iris-virginica'])
species_out = np.random.choice(a, 150, p=[0.5, 0.25, 0.25])

# Approach 2: Probablistic Sampling (preferred)
np.random.seed(100)
probs = np.r_[np.linspace(0, 0.500, num=50), np.linspace(0.501, .750, num=50), np.linspace(.751, 1.0, num=50)]
index = np.searchsorted(probs, np.random.random(150))
species_out = species[index]
print(np.unique(species_out, return_counts=True))

# > (array([b'Iris-setosa', b'Iris-versicolor', b'Iris-virginica'], dtype=object), array([77, 37, 36]))
```

方法2是首选方法，因为它创建了一个索引变量，该变量可用于取样2维表格数据。



### 5. 如何计算数组中所有可能值的行数？

**问题：**按行计算唯一值的计数。

**给定：**

```python
np.random.seed(100)
arr = np.random.randint(1,11,size=(6, 10))
arr
> array([[ 9,  9,  4,  8,  8,  1,  5,  3,  6,  3],
>        [ 3,  3,  2,  1,  9,  5,  1, 10,  7,  3],
>        [ 5,  2,  6,  4,  5,  5,  4,  8,  2,  2],
>        [ 8,  8,  1,  3, 10, 10,  4,  3,  6,  9],
>        [ 2,  1,  8,  7,  3,  1,  9,  3,  6,  2],
>        [ 9,  2,  6,  5,  3,  9,  4,  6,  1, 10]])
```

**期望的输出：**

```python
> [[1, 0, 2, 1, 1, 1, 0, 2, 2, 0],
>  [2, 1, 3, 0, 1, 0, 1, 0, 1, 1],
>  [0, 3, 0, 2, 3, 1, 0, 1, 0, 0],
>  [1, 0, 2, 1, 0, 1, 0, 2, 1, 2],
>  [2, 2, 2, 0, 0, 1, 1, 1, 1, 0],
>  [1, 1, 1, 1, 1, 2, 0, 0, 2, 1]]
```

输出包含10列，表示从1到10的数字。这些值是各行中数字的计数。
例如，cell(0，2)的值为2，这意味着数字3在第一行中恰好出现了2次。

**答案：**

```python
# **给定：**
np.random.seed(100)
arr = np.random.randint(1,11,size=(6, 10))
arr
# > array([[ 9,  9,  4,  8,  8,  1,  5,  3,  6,  3],
# >        [ 3,  3,  2,  1,  9,  5,  1, 10,  7,  3],
# >        [ 5,  2,  6,  4,  5,  5,  4,  8,  2,  2],
# >        [ 8,  8,  1,  3, 10, 10,  4,  3,  6,  9],
# >        [ 2,  1,  8,  7,  3,  1,  9,  3,  6,  2],
# >        [ 9,  2,  6,  5,  3,  9,  4,  6,  1, 10]])
# Solution
def counts_of_all_values_rowwise(arr2d):
    # Unique values and its counts row wise
    num_counts_array = [np.unique(row, return_counts=True) for row in arr2d]

    # Counts of all values row wise
    return([[int(b[a==i]) if i in a else 0 for i in np.unique(arr2d)] for a, b in num_counts_array])

# Print
print(np.arange(1,11))
counts_of_all_values_rowwise(arr)
# > [ 1  2  3  4  5  6  7  8  9 10]

# > [[1, 0, 2, 1, 1, 1, 0, 2, 2, 0],
# >  [2, 1, 3, 0, 1, 0, 1, 0, 1, 1],
# >  [0, 3, 0, 2, 3, 1, 0, 1, 0, 0],
# >  [1, 0, 2, 1, 0, 1, 0, 2, 1, 2],
# >  [2, 2, 2, 0, 0, 1, 1, 1, 1, 0],
# >  [1, 1, 1, 1, 1, 2, 0, 0, 2, 1]]
# Example 2:
arr = np.array([np.array(list('bill clinton')), np.array(list('narendramodi')), np.array(list('jjayalalitha'))])
print(np.unique(arr))
counts_of_all_values_rowwise(arr)
# > [' ' 'a' 'b' 'c' 'd' 'e' 'h' 'i' 'j' 'l' 'm' 'n' 'o' 'r' 't' 'y']

# > [[1, 0, 1, 1, 0, 0, 0, 2, 0, 3, 0, 2, 1, 0, 1, 0],
# >  [0, 2, 0, 0, 2, 1, 0, 1, 0, 0, 1, 2, 1, 2, 0, 0],
# >  [0, 4, 0, 0, 0, 0, 1, 1, 2, 2, 0, 0, 0, 0, 1, 1]]
```



### 6. 如何在numpy中为数组生成独热（one-hot）编码？

**问题：**计算一次性编码(数组中每个唯一值的虚拟二进制变量)

**给定：**

```python
np.random.seed(101) 
arr = np.random.randint(1,4, size=6)
arr
# > array([2, 3, 2, 2, 2, 1])
```

**期望输出：**

```python
# > array([[ 0.,  1.,  0.],
# >        [ 0.,  0.,  1.],
# >        [ 0.,  1.,  0.],
# >        [ 0.,  1.,  0.],
# >        [ 0.,  1.,  0.],
# >        [ 1.,  0.,  0.]])
```

**答案：**

```python
# **给定：**
np.random.seed(101) 
arr = np.random.randint(1,4, size=6)
arr
# > array([2, 3, 2, 2, 2, 1])

# Solution:
def one_hot_encodings(arr):
    uniqs = np.unique(arr)
    out = np.zeros((arr.shape[0], uniqs.shape[0]))
    for i, k in enumerate(arr):
        out[i, k-1] = 1
    return out

one_hot_encodings(arr)
# > array([[ 0.,  1.,  0.],
# >        [ 0.,  0.,  1.],
# >        [ 0.,  1.,  0.],
# >        [ 0.,  1.,  0.],
# >        [ 0.,  1.,  0.],
# >        [ 1.,  0.,  0.]])

# Method 2:
(arr[:, None] == np.unique(arr)).view(np.int8)
```



### 7. 如何创建按分类变量分组的行号？

**问题：**创建按分类变量分组的行号。使用以下来自鸢尾属植物物种的样本作为输入。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
species = np.genfromtxt(url, delimiter=',', dtype='str', usecols=4)
species_small = np.sort(np.random.choice(species, size=20))
species_small
# > array(['Iris-setosa', 'Iris-setosa', 'Iris-setosa', 'Iris-setosa',
# >        'Iris-setosa', 'Iris-setosa', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica'],
# >       dtype='<U15')
```

**期望的输出：**

```python
# > [0, 1, 2, 3, 4, 5, 0, 1, 2, 3, 4, 5, 0, 1, 2, 3, 4, 5, 6, 7]
```

**答案：**

```python
# **给定：**
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
species = np.genfromtxt(url, delimiter=',', dtype='str', usecols=4)
np.random.seed(100)
species_small = np.sort(np.random.choice(species, size=20))
species_small
# > array(['Iris-setosa', 'Iris-setosa', 'Iris-setosa', 'Iris-setosa',
# >        'Iris-setosa', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica'],
# >       dtype='<U15')
print([i for val in np.unique(species_small) for i, grp in enumerate(species_small[species_small==val])])
[0, 1, 2, 3, 4, 0, 1, 2, 3, 4, 5, 6, 7, 8, 0, 1, 2, 3, 4, 5]
```



### 8. 如何根据给定的分类变量创建组ID？

**问题：**根据给定的分类变量创建组ID。使用以下来自鸢尾属植物物种的样本作为输入。

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
species = np.genfromtxt(url, delimiter=',', dtype='str', usecols=4)
species_small = np.sort(np.random.choice(species, size=20))
species_small
# > array(['Iris-setosa', 'Iris-setosa', 'Iris-setosa', 'Iris-setosa',
# >        'Iris-setosa', 'Iris-setosa', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica'],
# >       dtype='<U15')
```

**期望的输出：**

```python
# > [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2]
```

**答案：**

```python
# **给定：**
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
species = np.genfromtxt(url, delimiter=',', dtype='str', usecols=4)
np.random.seed(100)
species_small = np.sort(np.random.choice(species, size=20))
species_small
# > array(['Iris-setosa', 'Iris-setosa', 'Iris-setosa', 'Iris-setosa',
# >        'Iris-setosa', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-versicolor', 'Iris-versicolor',
# >        'Iris-versicolor', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica', 'Iris-virginica', 'Iris-virginica',
# >        'Iris-virginica'],
# >       dtype='<U15')
# Solution:
output = [np.argwhere(np.unique(species_small) == s).tolist()[0][0] for val in np.unique(species_small) for s in species_small[species_small==val]]

# Solution: For Loop version
output = []
uniqs = np.unique(species_small)

for val in uniqs:  # uniq values in group
    for s in species_small[species_small==val]:  # each element in group
        groupid = np.argwhere(uniqs == s).tolist()[0][0]  # groupid
        output.append(groupid)

print(output)
# > [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2]
```



### 9. 如何使用numpy对多维数组中的项进行排名？

**问题：**创建与给定数字数组a相同形状的排名数组。

**给定：**

```python
np.random.seed(10)
a = np.random.randint(20, size=[2,5])
print(a)
# > [[ 9  4 15  0 17]
# >  [16 17  8  9  0]]
```

**期望输出：**

```python
# > [[4 2 6 0 8]
# >  [7 9 3 5 1]]
```

**答案：**

```python
# **给定：**
np.random.seed(10)
a = np.random.randint(20, size=[2,5])
print(a)

# Solution
print(a.ravel().argsort().argsort().reshape(a.shape))
# > [[ 9  4 15  0 17]
# >  [16 17  8  9  0]]
# > [[4 2 6 0 8]
# >  [7 9 3 5 1]]
```



### 10. 如何计算二维numpy数组每行的最小值？

**问题：**为给定的二维numpy数组计算每行的最小值。

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

# Solution
np.apply_along_axis(lambda x: np.min(x)/np.max(x), arr=a, axis=1)
# > array([ 0.44444444,  0.125     ,  0.5       ,  1.        ,  0.11111111])
```



### 11. 如何在numpy数组中找到重复的记录？

**问题：**在给定的numpy数组中找到重复的条目(第二次出现以后)，并将它们标记为True。第一次出现应该是False的。

**给定：**

```python
# Input
np.random.seed(100)
a = np.random.randint(0, 5, 10)
print('Array: ', a)
# > Array: [0 0 3 0 2 4 2 2 2 2]
```

**期望的输出：**

```python
# > [False  True False  True False False  True  True  True  True]
```

**答案：**

```python
# Input
np.random.seed(100)
a = np.random.randint(0, 5, 10)

## Solution
# There is no direct function to do this as of 1.13.3

# Create an all True array
out = np.full(a.shape[0], True)

# Find the index positions of unique elements
unique_positions = np.unique(a, return_index=True)[1]

# Mark those positions as False
out[unique_positions] = False

print(out)
# > [False  True False  True False False  True  True  True  True]
```



### 12. 如何找出数字的分组均值？

**问题：**在二维数字数组中查找按分类列分组的数值列的平均值

**给定：**

```python
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
```

**理想的输出：**

```python
# > [[b'Iris-setosa', 3.418],
# >  [b'Iris-versicolor', 2.770],
# >  [b'Iris-virginica', 2.974]]
```

**答案：**

```python
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')


# Solution
# No direct way to implement this. Just a version of a workaround.
numeric_column = iris[:, 1].astype('float')  # sepalwidth
grouping_column = iris[:, 4]  # species

# List comprehension version
[[group_val, numeric_column[grouping_column==group_val].mean()] for group_val in np.unique(grouping_column)]

# For Loop version
output = []
for group_val in np.unique(grouping_column):
    output.append([group_val, numeric_column[grouping_column==group_val].mean()])

output
# > [[b'Iris-setosa', 3.418],
# >  [b'Iris-versicolor', 2.770],
# >  [b'Iris-virginica', 2.974]]
```



### 13. 如何将PIL图像转换为numpy数组？

**问题：**从以下URL导入图像并将其转换为numpy数组。

```
URL = 'https://upload.wikimedia.org/wikipedia/commons/8/8b/Denali_Mt_McKinley.jpg'
```

**答案：**

```python
from io import BytesIO
from PIL import Image
import PIL, requests

# Import image from URL
URL = 'https://upload.wikimedia.org/wikipedia/commons/8/8b/Denali_Mt_McKinley.jpg'
response = requests.get(URL)

# Read it as Image
I = Image.open(BytesIO(response.content))

# Optionally resize
I = I.resize([150,150])

# Convert to numpy array
arr = np.asarray(I)

# Optionaly Convert it back to an image and show
im = PIL.Image.fromarray(np.uint8(arr))
Image.Image.show(im)
```



### 14. 如何计算两个数组之间的欧氏距离？

**问题：**计算两个数组a和数组b之间的欧氏距离。

**给定：**

```python
a = np.array([1,2,3,4,5])
b = np.array([4,5,6,7,8])
```

**答案：**

```python
# Input
a = np.array([1,2,3,4,5])
b = np.array([4,5,6,7,8])

# Solution
dist = np.linalg.norm(a-b)
dist
# > 6.7082039324993694
```



### 15. 如何在一维数组中找到所有的局部极大值(或峰值)？

**问题：**找到一个一维数字数组a中的所有峰值。峰顶是两边被较小数值包围的点。

**给定：**

```python
a = np.array([1, 3, 7, 1, 2, 6, 0, 1])
```

**期望的输出：**

```python
# > array([2, 5])
```

其中，2和5是峰值7和6的位置。

**答案：**

```python
a = np.array([1, 3, 7, 1, 2, 6, 0, 1])
doublediff = np.diff(np.sign(np.diff(a)))
peak_locations = np.where(doublediff == -2)[0] + 1
peak_locations
# > array([2, 5])
```



### 16. 如何计算numpy数组的移动平均值？

**问题：**对于给定的一维数组，计算窗口大小为3的移动平均值。

**给定：**

```python
np.random.seed(100)
Z = np.random.randint(10, size=10)
```

**答案：**

```python
# Solution
# Source: https://stackoverflow.com/questions/14313510/how-to-calculate-moving-average-using-numpy
def moving_average(a, n=3) :
    ret = np.cumsum(a, dtype=float)
    ret[n:] = ret[n:] - ret[:-n]
    return ret[n - 1:] / n

np.random.seed(100)
Z = np.random.randint(10, size=10)
print('array: ', Z)
# Method 1
moving_average(Z, n=3).round(2)

# Method 2:  # Thanks AlanLRH!
# np.ones(3)/3 gives equal weights. Use np.ones(4)/4 for window size 4.
np.convolve(Z, np.ones(3)/3, mode='valid') . 


# > array:  [8 8 3 7 7 0 4 2 5 2]
# > moving average:  [ 6.33  6.    5.67  4.67  3.67  2.    3.67  3.  ]
```



### 17. 如何填写不规则系列的numpy日期中的缺失日期？

**问题：**给定一系列不连续的日期序列。填写缺失的日期，使其成为连续的日期序列。

**给定：**

```python
# Input
dates = np.arange(np.datetime64('2018-02-01'), np.datetime64('2018-02-25'), 2)
print(dates)
# > ['2018-02-01' '2018-02-03' '2018-02-05' '2018-02-07' '2018-02-09'
# >  '2018-02-11' '2018-02-13' '2018-02-15' '2018-02-17' '2018-02-19'
# >  '2018-02-21' '2018-02-23']
```

**答案：**

```python
# Input
dates = np.arange(np.datetime64('2018-02-01'), np.datetime64('2018-02-25'), 2)
print(dates)

# Solution ---------------
filled_in = np.array([np.arange(date, (date+d)) for date, d in zip(dates, np.diff(dates))]).reshape(-1)

# add the last day
output = np.hstack([filled_in, dates[-1]])
output

# For loop version -------
out = []
for date, d in zip(dates, np.diff(dates)):
    out.append(np.arange(date, (date+d)))

filled_in = np.array(out).reshape(-1)

# add the last day
output = np.hstack([filled_in, dates[-1]])
output
# > ['2018-02-01' '2018-02-03' '2018-02-05' '2018-02-07' '2018-02-09'
# >  '2018-02-11' '2018-02-13' '2018-02-15' '2018-02-17' '2018-02-19'
# >  '2018-02-21' '2018-02-23']

# > array(['2018-02-01', '2018-02-02', '2018-02-03', '2018-02-04',
# >        '2018-02-05', '2018-02-06', '2018-02-07', '2018-02-08',
# >        '2018-02-09', '2018-02-10', '2018-02-11', '2018-02-12',
# >        '2018-02-13', '2018-02-14', '2018-02-15', '2018-02-16',
# >        '2018-02-17', '2018-02-18', '2018-02-19', '2018-02-20',
# >        '2018-02-21', '2018-02-22', '2018-02-23'], dtype='datetime64[D]')
```



### 18. 如何从给定的一维数组创建步长？

**问题：**从给定的一维数组arr中，利用步进生成一个二维矩阵，窗口长度为4，步距为2，类似于 `[[0,1,2,3], [2,3,4,5], [4,5,6,7]..]`

**给定：**

```python
arr = np.arange(15) 
arr
# > array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14])
```

**期望的输出：**

```python
# > [[ 0  1  2  3]
# >  [ 2  3  4  5]
# >  [ 4  5  6  7]
# >  [ 6  7  8  9]
# >  [ 8  9 10 11]
# >  [10 11 12 13]]
```

**答案：**

```python
def gen_strides(a, stride_len=5, window_len=5):
    n_strides = ((a.size-window_len)//stride_len) + 1
    # return np.array([a[s:(s+window_len)] for s in np.arange(0, a.size, stride_len)[:n_strides]])
    return np.array([a[s:(s+window_len)] for s in np.arange(0, n_strides*stride_len, stride_len)])

print(gen_strides(np.arange(15), stride_len=2, window_len=4))
# > [[ 0  1  2  3]
# >  [ 2  3  4  5]
# >  [ 4  5  6  7]
# >  [ 6  7  8  9]
# >  [ 8  9 10 11]
# >  [10 11 12 13]]
```