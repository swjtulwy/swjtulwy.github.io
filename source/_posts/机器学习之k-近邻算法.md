---
title: 机器学习之k-近邻算法
date: 2019-05-20 20:54:31
tags: [机器学习]
categories: 机器学习实战
---

# k-近邻算法（kNN）

## 原理概述

k-近邻（kNN, k-NearestNeighbor）算法是一种基本**分类与回归**方法，我们这里只讨论分类问题中的 k-近邻算法。

k 近邻算法的输入为实例的特征向量，对应于特征空间的点；输出为实例的类别，可以取多类。k 近邻算法假设给定一个训练数据集，其中的实例类别已定。分类时，对新的实例，根据其 k 个最近邻的训练实例的类别，通过**多数表决等方式**进行预测。因此，k近邻算法不具有显式的学习过程。
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

