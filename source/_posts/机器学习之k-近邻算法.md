---
title: 机器学习之k-近邻算法
date: 2019-05-20 20:54:31
tags: [机器学习]
categories: 机器学习实战
---

# k-近邻分类算法（kNN）

## 原理概述

k-近邻（kNN, k-NearestNeighbor）算法是一种基本**分类与回归**方法，我们这里只讨论分类问题中的 k-近邻算法。

k 近邻算法的输入为实例的特征向量，对应于特征空间的点；输出为实例的类别，可以取多类。k 近邻算法假设给定一个训练数据集，其中的实例类别已定。<!--more-->分类时，对新的实例，根据其 k 个最近邻的训练实例的类别，通过**多数表决等方式**进行预测。因此，k近邻算法不具有显式的学习过程。
k 近邻算法实际上利用训练数据集对特征向量空间进行划分，并作为其分类的“模型”。 **k值的选择**、**距离度量**以及**分类决策规则**是k近邻算法的三个基本要素。

### KNN 工作原理

1. 假设有一个带有标签的样本数据集（训练样本集），其中包含每条数据与所属分类的对应关系。
2. 输入没有标签的新数据后，将新数据的每个特征与样本集中数据对应的特征进行比较。
   1. 计算新数据与样本数据集中每条数据的距离。
   2. 对求得的所有距离进行排序（从小到大，越小表示越相似）。
   3. 取前 k （k 一般小于等于 20 ）个样本数据对应的分类标签。
3. 求 k 个数据中出现次数最多的分类标签作为新数据的分类。

### KNN 通俗理解

给定一个训练数据集，对新的输入实例，在训练数据集中找到与该实例最邻近的 k 个实例，这 k 个实例的多数属于某个类，就把该输入实例分为这个类。

## 算法的一般流程 

1. 收集数据：任何方法
2. 准备数据：距离计算所需要的数值，最好是结构化的数据格式
3. 分析数据：任何方法
4. 训练算法：此步骤不适用于 k-近邻算法
5. 测试算法：计算错误率
6. 使用算法：输入样本数据和结构化的输出结果，然后运行 k-近邻算法判断输入数据分类属于哪个分类，最后对计算出的分类执行后续处理

## 算法特点

优点：精度高、对异常值不敏感、无数据输入假定
缺点：计算复杂度高、空间复杂度高
适用数据范围：数值型和标称型

## 算法实现

代码中定义了两个简单的函数，一个是数据集构造函数，另一个则是分类器

```python
from numpy import *
import operator  # 运算符模块


def createDataSet():
    group = array([[1.0, 1.1], [1.0, 1.0], [0, 0], [0, 0.1]])
    labels = ['A', 'A', 'B', 'B']
    return group, labels

# inX - - 用于分类的输入向量 / 测试数据
# dataSet - - 训练数据集的features
# labels - - 训练数据集的labels
# k - - 选择最近邻的数目
def classify(inX, dataSet, labels, k):
    dataSetSize = dataSet.shape[0]
    # tile用于将inX向量在列方向上（向右）重复1次，行方向上（向下）重复dataSize次
    diffMat = tile(inX, (dataSetSize, 1)) - dataSet  # 计算向量各维之差
    sqDiffmat = diffMat ** 2  # 差取平方
    sqDistances = sqDiffmat.sum(axis=1)  # 再求和
    distances = sqDistances ** 0.5  # 开根号就是欧几里得距离
    #  argsort()函数是将x中的元素从小到大排列，提取其对应的index(索引)，然后输出到y
    sortedDistIndicies = distances.argsort()
    classCount = {}  # 类别计数，这个是字典，不是集合
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1  # 0 是默认返回值
        # 2.7版本有iteritems方法，3.x版本用items代替
        # 逆向排序，reverse 为True
        # key - - 主要是用来进行比较的元素，只有一个参数，
        # 具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。
        # operator.itemgetter函数获取的不是值，而是定义了一个函数，通过该函数作用到对象上才能获取值。
    sortedClassCount = sorted(classCount.items(), key=operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]  # 返回计数最多的一个类（排序后的第一行）的类名（第一列）

```

## 实例1-改进约会网站配对效果

### 项目概述

海伦使用约会网站寻找约会对象。经过一段时间之后，她发现曾交往过三种类型的人:

- 不喜欢的人
- 魅力一般的人
- 极具魅力的人

她希望分类软件能够更好地帮助她将匹配对象划分到确切的分类中，她还收集了一些约会网站未曾记录的数据信息，她认为这些数据有助于匹配对象的归类。

### 准备数据（从文本文件中解析数据）

海伦把这些约会对象的数据存放在文本文件 [datingTestSet2.txt](https://github.com/apachecn/AiLearning/blob/master/data/2.KNN/datingTestSet2.txt) 中，每个样本数据占一行，总共有 1000 行。海伦约会的对象（样本）主要包含以下 3 种特征：

- 每年获得的飞行常客里程数
- 玩视频游戏所耗时间百分比
- 每周消费的冰淇淋公升数

文本文件数据格式如下：

```
40920	8.326976	0.953952	3
14488	7.153469	1.673904	2
26052	1.441871	0.805124	1
75136	13.147394	0.428964	1
38344	1.669788	0.134296	1
```

解析程序如下：

```python
def file2matrix(filename):
    fr = open(filename)
    arrayOfLines = fr.readlines()
    numberOfLines = len(arrayOfLines)  # 文件行数
    returnMat = zeros((numberOfLines, 3))  # 创建返回的Numpy矩阵
    classLabelVector = []
    index = 0
    for line in arrayOfLines:
        line = line.strip()  # 截取回车字符
        listFromLine = line.split('\t')  # 以制表符为界分割生成列表
        returnMat[index, :] = listFromLine[0:3]  # 处理后的每一行的样本
        classLabelVector.append(int(listFromLine[-1]))  # 处理后的每一行标签，强制整型，否则是字符串
        index += 1
    return returnMat, classLabelVector
```

查看数据抽取结果：

![](机器学习之k-近邻算法\20190521152851.png)

### 分析数据（使用Matplotlib创建散点图） 

```python
import matplotlib
import matplotlib.pyplot as plt
fig = plt.figure()  # 画图对象
ax = fig.add_subplot(111) # 添加子图，111表示1行一列第1个图
ax.scatter(datingDataMat[:, 1], datingDataMat[:, 2])  # 样本的第1与第2列
plt.show()
```

根据以上代码生成的是没有样本类别标签的散点图

![](机器学习之k-近邻算法\Figure_1.png)

当修改其中一行代码时

```python
ax.scatter(datingDataMat[:, 1], datingDataMat[:, 2], 15.0*np.array(datingLabels), 15.0*np.array(datingLabels))  # 样本的第1与第2列
```

下图是修改代码后带样本类别标签的散点图，可以看出不同类别用不同颜色表示，很直观

![](机器学习之k-近邻算法\Figure_2.png)

带有样本分类标签的约会数据散点图虽然能够比较容易地区分数据点从属类，但依然很难根据这张图得出结论性信息，采用不同的属性值也许可以得到更好效果，上图采用矩阵属性列1和2，下图我们采用列0和1表示，代码如下：

```python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import kNN

datingDataMat, datingLabels = kNN.file2matrix('datingTestSet2.txt')
fig = plt.figure()
ax = fig.add_subplot(111) 
datingLabels = np.array(datingLabels)
# 标签为1的簇
idx_1 = np.where(datingLabels == 1)
p1 = ax.scatter(datingDataMat[idx_1, 0], datingDataMat[idx_1,1],c = 'r',  label = 'Did Not Like', s = 20)
# 标签为2的簇
idx_2 = np.where(datingLabels == 2)
p2 = ax.scatter(datingDataMat[idx_2, 0], datingDataMat[idx_2,1],c = 'g',  label = 'Liked in Large Doses', s = 10)
# 标签为3的簇
idx_3 = np.where(datingLabels == 3)
p3 = ax.scatter(datingDataMat[idx_3, 0], datingDataMat[idx_3,1], c = 'b', label = 'Liked in Small Doses', s = 30)
# 铭牌
plt.legend(loc = 'upper left')
plt.show()
```

可以看出图中清晰地标识了三个不同的样本分类区域，具有不同爱好的人其类别区域也不同。

![](机器学习之k-近邻算法\Figure_3.png)

### 准备数据：归一化数值

归一化是为了消除特征之间量级不同导致的影响，避免出现有些属性数值很大以造成对结果的权重过大。

#### 归一化定义

我是这样认为的，归一化就是要把你需要处理的数据经过处理后（通过某种算法）限制在你需要的一定范围内。首先归一化是为了后面数据处理的方便，其次是保正程序运行时收敛加快。

#### 归一化方法

- $min-max$标准化（$Min-Max Normalization$）

  也称为离差标准化，是对原始数据的线性变换，使结果值映射到[0 ,1]之间。转换函数如下：

$$
x^*=\frac{x-min}{max-min}
$$

​	其中$max$为样本数据的最大值，$min$为样本数据的最小值。这种方法有个缺陷就是当有新数据加入时，可能	导致$max$和$min$的变化，需要重新定义。

- $Z-score$标准化方法

  这种方法给予原始数据的均值（$mean$）和标准差（$standard deviation$）进行数据的标准化。经过处理的数据符合标准正态分布，即均值为0，标准差为1，转化函数为：
  $$
  x^*=\frac{x-\mu}{\sigma}
  $$
  其中$\mu$ 为所有样本数据的均值，$\sigma ​$为所有样本数据的标准差。

本实例采用$min-max$标准化

```python
# 归一化特征值
def autoNorm(dataSet):
    minVals = dataSet.min(0) # 选取最小值，0表示从列中选，得到每列的最小值
    maxVals = dataSet.max(0) # 选取最大值，0表示从列中选，得到每列的最大值
    ranges = maxVals - minVals # 全矩
    
    # # -------第一种实现方式---start---------------------------------------
    normDataSet = zeros(shape(dataSet))  # 存储结果
    m = dataSet.shape[0] # 行数
    normDataSet = dataSet - tile(minVals, (m, 1))  # 生成与最小值之差组成的矩阵
    normDataSet = normDataSet / tile(ranges, (m, 1)) # 将最小值之差除以范围组成矩阵
    # # -------第一种实现方式---end---------------------------------------------
    
    # # -------第二种实现方式---start---------------------------------------
    # norm_dataset = (dataset - minvalue) / ranges
    # # -------第二种实现方式---end---------------------------------------------
    return  normDataSet, ranges, minVals 

```

### 测试算法：作为完整程序验证分类器

接下来对算法进行测试，划分训练数据集为90%的原数据集

```python
# 测试代码
def datingClassTest():
    # 设置测试数据的的一个比例（训练数据集比例=1-hoRatio）
    hoRatio = 0.10
    datingDataMat, datingLabels = file2matrix('datingTestSet.txt')
    normMat, ranges, minVals = autoNorm(datingDataMat)
    # m 表示数据的行数，即矩阵的第一维
    m = normMat.shape[0]
    # 设置测试的样本数量， numTestVecs:m表示训练样本的数量
    numTestVecs = int(m*hoRatio)
    errorCount = 0.0
    for i in range(numTestVecs):
        # 对数据测试
        classifierResult = classify0(normMat[i], normMat[numTestVecs: m], datingLabels[numTestVecs: m], 3)
        print("the classifier came back with: %d, the real answer is: %d" % (classifierResult, datingLabels[i]))
        errorCount += classifierResult != datingLabels[i]
    print("the total error rate is: %f" % (errorCount / numTestVecs))
    print(errorCount)
```

结果为：

```
the classifier came back with: 3, the real answer is: 3
the classifier came back with: 2, the real answer is: 2
the classifier came back with: 1, the real answer is: 1
the classifier came back with: 1, the real answer is: 1
……
the classifier came back with: 2, the real answer is: 2
the classifier came back with: 1, the real answer is: 1
the classifier came back with: 3, the real answer is: 1
the total error rate is: 0.050000
5.0
```

### 使用算法：构建完整可用系统

将新得到要预测的样本，进行同样标准化之后，输入分类器即可。



## 实例2-手写识别系统

### 项目概述

构造一个能识别数字 0 到 9 的基于 KNN 分类器的手写数字识别系统。

需要识别的数字是存储在文本文件中的具有相同的色彩和大小：宽高是 32 像素 * 32 像素的黑白图像。

### 准备数据：将图像转换为测试向量

样本训练集在目录 [trainingDigits](https://github.com/apachecn/AiLearning/blob/master/data/2.KNN/trainingDigits) 中，包含了大约 2000 个例子，每个例子内容如下图所示，每个数字大约有 200 个样本；测试集目录 [testDigits](https://github.com/apachecn/AiLearning/blob/master/data/2.KNN/testDigits) 中包含了大约 900 个测试数据。

![](机器学习之k-近邻算法\20190521211436.png)

编写函数$ img2vector()$, 将图像文本数据$（32\times32）$转换为分类器使用的向量$（1\times24）$ 。

```python
# 手写数字识别
def img2vector(filename):
    returnVect = zeros((1, 1024))
    fr = open(filename, 'r')
    for i in range(32):
        lineStr = fr.readline()
        for j in range(32):
            returnVect[0, 32 * i + j] = int(lineStr[j])
    return returnVect
```

### 测试算法：使用k-近邻算法识别手写数字

```python
# 手写识别分类器
def handwritingClassTest():
    # 1. 导入数据
    hwLabels = []
    # os.listdir() 方法用于返回指定的文件夹包含的文件或文件夹的名字的列表。
    trainingFileList = os.listdir("trainingDigits") # load the training set
    m = len(trainingFileList)
    trainingMat = zeros((m, 1024))
    # hwLabels存储0～9对应的index位置， trainingMat存放的每个位置对应的图片向量
    for i in range(m):
        fileNameStr = trainingFileList[i]
        fileStr = fileNameStr.split('.')[0]  # take off .txt
        classNumStr = int(fileStr.split('_')[0])
        hwLabels.append(classNumStr)
        # 将 32*32的矩阵->1*1024的矩阵
        trainingMat[i] = img2vector('trainingDigits/%s' % fileNameStr)

    # 2. 导入测试数据
    testFileList = os.listdir('testDigits')  # iterate through the test set
    errorCount = 0
    mTest = len(testFileList)
    for i in range(mTest):
        fileNameStr = testFileList[i]
        fileStr = fileNameStr.split('.')[0]  # take off .txt
        classNumStr = int(fileStr.split('_')[0])
        vectorUnderTest = img2vector('testDigits/%s' % fileNameStr)
        classifierResult = classify0(vectorUnderTest, trainingMat, hwLabels, 3)
        print("the classifier came back with: %d, the real answer is: %d" % (classifierResult, classNumStr))
        errorCount += classifierResult != classNumStr
    print("\nthe total number of errors is: %d" % errorCount)
    print("\nthe total error rate is: %f" % (errorCount / mTest))
```

```python
kNN.handwritingClassTest()
```

```
the total number of errors is: 10

the total error rate is: 0.010571
```

实际使用这个算法时，算法的执行效率并不高。因为算法需要为每个向量做2000次距离计算，每个距离计算包含1024个维度浮点运算，总共要执行900次，此外我们还要为测试向量准备2MB的存储空间。

于是$k-$决策树算法应运而生，可以减少存储空间和计算开销，$k-$决策树是$k-$近邻算法的优化版。$k-$近邻算法的另一个缺陷是它无法给出任何数据的基础结构信息，因此我们也无法知晓平均实例样本和典型实例样本具有什么特征。

## $k$值3要素

-  $K$的取值

  对查询点标签影响显著（效果拔群）。$k$值小的时候 近似误差小，估计误差大。$ k$值大 近似误差大，估计误差小。

  如果选择较小的 k 值，就相当于用较小的邻域中的训练实例进行预测，“学习”的近似误差（approximation error）会减小，只有与输入实例较近的（相似的）训练实例才会对预测结果起作用。但缺点是“学习”的估计误差（estimation error）会增大，预测结果会对近邻的实例点非常敏感。如果邻近的实例点恰巧是噪声，预测就会出错。换句话说，$k$ 值的**减小**就意味着整体模型变得复杂，容易发生**过拟合**。

  如果选择较大的 $k $值，就相当于用较大的邻域中的训练实例进行预测。其优点是可以减少学习的估计误差。但缺点是学习的近似误差会增大。这时与输入实例较远的（不相似的）训练实例也会对预测起作用，使预测发生错误。 $k$ 值的**增大**就意味着整体的模型变得**简单** ,容易发生**欠拟合**。

  太小都不太好，可以用交叉验证（cross validation）来选取适合的$k$值。

- 距离度量 Metric/Distance Measure​

  距离度量 通常为 欧式距离（Euclidean distance），还可以是 Minkowski ​距离 或者曼哈顿距离。也可以是地理空间中的一些距离公式。（更多细节可以参看 sklearn 中valid_metric 部分）

- 分类决策 （decision rule​）

  分类决策在分类问题中通常为通过少数服从多数来选取票数最多的标签，在**回归问题**中通常为 $K$个最邻点的标签的平均值。