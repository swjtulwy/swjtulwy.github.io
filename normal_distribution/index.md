# 正态分布


## 大数定律

大数定律(Law of Large Numbers)简单地说，就是当样本量足够大时，样本均值会趋近总体期望值

辛钦大数定律定义：

设 $\{a_i,i\geq 1\}$ 为独立同分布的随机变量序列，若 $a_i$ 的数学期望存在，则服从大数定律：即对任意的 $\epsilon\gt 0$ ,有公式：

$$
\lim_{n\to\infty}P\left(\left|\frac{1}{n}\sum_{i=1}^na_i-\mu\right|\lt\epsilon\right)=1
$$

## 概率知识补充

研究一个随机变量，不只是要看它能取哪些值，更重要的是它取各种值的概率如何！

### 概率函数

概率函数就是用函数的形式来表示概率：例如 $P_i=P(X=a_i)(i=1,2,3,4,5,6)$ 。在这个函数里，自变量 X 是随机变量的取值，因变量 $P_i$ 是取值的概率。这就叫用数学语言来表示自然现象！它就代表了每个取值的概率，所以顺理成章的它就叫做了X的概率函数。**从公式上来看，概率函数一次只能表示一个取值的概率。比如P(X=1)=1/6,这代表用概率函数的形式来表示，当随机变量取值为1的概率为1/6，一次只能代表一个随机变量的取值。**

有时候也会把概率函数称作分布律

### 概率分布

顾名思义就是概率的分布，这个概率分布还是讲概率的。**我认为在理解这个概念时，关键不在于“概率”两个字，而在于“分布”这两个字。**离散型随机变量的概率分布可以用列表表示

### 分布函数

实际上就是**概率分布函数(Cumulative Distribution Function)**，它就是概率函数取值的累加结果！所以它又叫累积概率函数！其实，我觉得叫它累积概率函数还更好理解！！**概率函数和概率分布函数就像是一个硬币的两面，它们都只是描述概率的不同手段！**

分布函数是针对**离散**型随机变量的

### 概率密度函数

实际上就是**连续型随机变量**的概率函数，**概率密度函数用数学公式表示就是一个定积分的函数，定积分在数学中是用来求面积的，而在这里，你就把概率表示为面积即可！**处理连续数据时，所计算的是一个数值范围的概率

概率密度函数是分布函数的**导函数**。

下面进入正文

## 正态分布(Normal Distribution)

### 正态分布定义

若随机变量服从一个数学期望为 $\mu$、方差为$\sigma^2$ 的正态分布，记为$X\sim{N(\mu，\sigma^2)}$。其概率密度函数为：

$$
f(x)=\frac{1}{\sqrt{2\pi}\cdot\sigma}e^{-\frac{1}{2}\left(\frac{x-\mu}{\sigma}\right)^2}
$$

正态分布的期望值 $\mu$ 决定了其位置，其标准差 $\sigma$ 决定了分布的幅度。当 $\mu$  = 0, $\sigma$  = 1时的正态分布是标准正态分布

$\sigma$ 描述正态分布资料数据分布的离散程度，$\sigma$ 越大，数据分布越分散，$\sigma$ 越小，数据分布越集中。也称为是正态分布的形状参数，$\sigma$ 越大，曲线越扁平，反之，$\sigma$ 越小，曲线越瘦高。

二项分布是离散分布，而正态分布是连续分布，使用正态分布近似代替二项分布时，必须进行连续性修正

### 正态分布性质

- 如果$X\sim{N(\mu,\sigma^2)}$ ,且a与b是实数，那么 $aX+b\sim{N(a\mu+b,(a\sigma)^2)}$ 
- 两个统计独立的正态随机变量之和或差也满足正态分布，对应的参数也是它们的参数之和或差

正态概率计算三步法：

1. 确定分布与范围
2. 使其标准化
3. 查找概率

### 总结

<img src="https://i0.hdslb.com/bfs/album/9ae3b0d7650ad0995eb5ebfe7b20a0e17d06fd95.png@1e_1c.webp" title="" alt="" data-align="center">
