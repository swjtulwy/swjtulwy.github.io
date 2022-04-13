# 中心极限定理、置信区间


## 中心极限定理

介绍中心极限定理(Central Limit Theorem)之前先复习一下总体和样本以及抽样的知识

### 抽样相关知识

#### 抽样样本

 **无偏样本** 可以代表总目标总体，即该样本与总体样本具有相似特性，我们可以利用这些相似特性对总体本身做出判断，一个无偏样本的分布形状与作为其来源的总体的分布形状相似，如果我们知道样本的分布形状，就可以根据此以合理程度的置信水平预测总体的分布形状。

**偏倚样本** 无法代表目标总体，由于样本与总体特性的不相似，无法根据样本对总体做出判断。如果我们试图用样本的分布形状预测总体的分布形状，最终会得出错误的结果。

偏倚来源：

- 抽样空间中条目不齐全，因此为包含目标总体中的所有对象
- 抽样单位不正确
- 为样本选取的一个个抽样单位未出现在实际样本中
- 调查问卷的问题设计不当
- 样本缺乏随机性

#### 抽样方法

- **简单随机抽样** 就是通过随机过程选取一个大小为n的样本，所有大小为n的可能样本被选中的可能性相同
  
  - 重复抽样
    
    在选取一个抽样单位并记录下这个抽样单位的相关信息之后，再将这个单位放回总体中
  
  - 不重复抽样
    
    不再将抽样单位放回总体

- **分层抽样** 将总体分割为几个相似的组，每个组具有类似的特性，这些特性称之为层

- **整群抽样** 不是对抽样单位进行简单随机抽样，而是对群进行简单随机抽样，然后对每个群的各种特性进行调查

- **系统抽样** 按照某种顺序列出总体名单，然后每k个单位进行一次调查，其中k为一个特定的数字

### 点估计量

当我们不知道总体参数的确切数值，我们无法通过总体计算这些参数，而只能通过样本数据估计这些参数，于是我们用“[点估计量](https://zh.wikipedia.org/wiki/%E7%82%B9%E4%BC%B0%E8%AE%A1)”对总体参数进行最接近的预测

一个总体参数的点估计量就是可用于估计总体参数数值的某个函数或算式，例如，我们能用样本均值估计总体均值，因此样本均值就是总体均值的点估计量

我们用符号$\hat{}$ 区别实际总体参数和它的点估计值，例如用符号 $\mu$ 表示总体均值，而用$\hat{\mu}$ 表示样本均值，即为了指出你正在使用的是某一个总体参数的点估计量，则在该总体参数符号上标$\hat{ }$ ,因此有 $\hat{\mu}=\overline{x}$ ， $\hat{\sigma^2}=s^2$

**方差的点估计**时，当手上有总体的数据时，分母为n,当用样本数据来估计总体方差时，分母为n-1

$$
\sigma^2=\frac{\sum(x-\mu)^2}{n}
$$

$$
\hat{\sigma^2}=s^2=\frac{\sum(x-\overline{x})^2}{n-1}
$$

### 预测总体比例

(总体成功比例的点估计量)$\hat{p}=p_s$ (样本成功比例)

$$
ps=\frac{成功数目}{样本数目}=\frac{X}{n}
$$

计算概率的方法和计算比例的方法完全一样$p=probability(概率)=proportion(比例)​$

**计算样本比例本身的概率：**

1. 查看与我们的特定样本大小相同的所有样本
2. 观察所有样本比例形成的分布，然后求出比例的期望和方差
3. 得出上述比例的分布后，利用该分布求出概率

利用比例的抽样分布，能够求出某一个随机选择的、大小为n的样本的“成功比例”的概率

**期望**：

$$
\begin{aligned}
E(p_s)&=E\left(\frac{X}{n}\right)\\\
&=\frac{E(x)}{n}
\end{aligned}
$$

$X$为成功数目，则 $X\sim{B(n,p)}$ ,$E(x)=np$ , 所以 $E(P_s)=p$

**方差**：

$$
\begin{aligned}
Var(P_s)&=\left(\frac{X}{n}\right)\\\
&=\frac{Var(X)}{n^2}
\end{aligned}
$$

二项分布的方差$Var(X)=npq$ ,所以，$Var(P_s)=\frac{pq}{n}$  

比例的标准差也叫**比例标准误差**，它能指出样本比例的可能误差

 当n很大(大于30)时，$P_S$ 的分布接近正态分布，n越大越接近正态分布

此时，

$$
P_s\sim{N\left(p,\frac{pq}{n}\right)}
$$

如果要用正态分布近似计算$P_s$的概率，一定要用$\pm(1/2n)$ 进行连续性修正；连续型修正的确切数值取决于数值n

随着n增大，连续性修正变得很小，于是对整个概率带来的变化极小。

### 样本均值的抽样分布

样本均值期望：

$$
\begin{aligned}
E(\overline{X})&=E\left(\frac{X_1+X_2+\cdot\cdot\cdot+X_n}{n}\right)\\\
&=E\left(\frac{1}{n}X_1\right)+E\left(\frac{1}{n}X_2\right)+\cdot\cdot\cdot+E\left(\frac{1}{n}X_n\right)\\\
&=\frac{1}{n}(E(X_1)+E(X_2)+\cdot\cdot\cdot+E(X_n))
\end{aligned}
$$

由于每一个$X_i$ 都是$X$的一个独立观察值，且我们已知$E(X)=\mu$ ,于是

$$
E(\overline{X})=\mu
$$

同样计算方法得到

$$
Var(\overline{X})=\frac{\sigma^2}{n}
$$

均值的标准差为均值的标准误差，n越大，均值标准误差越小

### 中心极限定理定义

[中心极限定理](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%BF%83%E6%9E%81%E9%99%90%E5%AE%9A%E7%90%86)是指：如果每次从一个非正态总体X中取出一个样本，容量为n，且样本很大，则$\overline{X}$ 的分布近似为正态分布。如果总体的均值和方差为$\mu$ 和$\sigma^2$ , 且n很大，例如大于30，则

辛钦中心极限定理：

$$
\overline{X}\sim{N(\mu,\frac{\sigma^2}{n})}
$$

设随机变量$X_1,X_2,...X_n$ 相互独立，服从同一分布且有有限的数学期望 $\mu$ 和方差 $\sigma ^2$ ,则随机变量 $\overline{X} =\frac{\sum{X_i}}{n}$, 在$n$ 无限大的时候，服从参数为 $\mu$ 和 $\frac{\sigma^2}{n}$ 的正态分布。

中心极限定理告诉我们，当样本量足够大时，样本均值的分布慢慢变成正态分布，就像这个图(其中黄色的是标准正态分布的密度函数)

<img title="" src="https://i0.hdslb.com/bfs/album/667086622c8526ff3b285f41127b8d9572327b6d.png@1e_1c.webp" alt="" data-align="center">

注意的几个点：

1. **总体本身的分布不要求正态分布**
   上面的例子中，人的体重是正态分布的。但如果我们的例子是掷一个骰子（平均分布），最后每组的平均值也会组成一个正态分布。
2. **样本每组要足够大，但也不需要太大**
   取样本的时候，一般认为，每组大于等于30个，即可让中心极限定理发挥作用。

### 中心极限定理的数据展示

假设我们现在观测一个人掷骰子。这个骰子是公平的，也就是说掷出1~6的概率都是相同的：1/6。他掷了一万次。我们用python来模拟投掷的结果：

- **第一步， 生成数据**
  
  假设我们现在观测一个人掷骰子。这个骰子是公平的，也就是说掷出1~6的概率都是相同的：1/6。他掷了一万次。我们用python来模拟投掷的结果：
  
  ```python
  import numpy as np
  import matplotlib.pyplot as plt
  import matplotlib
  
  random_data = np.random.randint(1, 7, 10000)
  print ("总体均值： ", random_data.mean()) # 打印平均值
  print ("总体标准差： ", random_data.std()) # 打印标准差
  ```
  
  生成出来的平均值：**3.5079**（每次重新生成都会略有不同）*
  生成出来的标准差：**1.7027441**
  
  平均值接近3.5很好理解。 因为每次掷出来的结果是1、2、3、4、5、6。 每个结果的概率是1/6。所以加权平均值就是3.5。

- **第二步，画出来看看**
  
  我们把生成的数据用直方图画出来直观地感受一下：
  
  ```python
  # 设置matplotlib正常显示中文和负号
  matplotlib.rcParams['font.sans-serif']=['SimHei']   # 用黑体显示中文
  matplotlib.rcParams['axes.unicode_minus']=False     # 正常显示负号
  # 随机生成（10000,）服从正态分布的数据
  # samples_mean_np = np.random.randn(10000)
  """
  绘制直方图
  data:必选参数，绘图数据
  bins:直方图的长条形数目，可选项，默认为10
  alpha:透明度
  """
  plt.hist(random_data, bins=6, facecolor="blue", edgecolor="black", alpha=0.7)
  # 显示横轴标签
  plt.xlabel("总体数据")
  # 显示纵轴标签
  plt.ylabel("频数/频率")
  # 显示图标题
  plt.title("频数/频率分布直方图")
  plt.show()
  ```
  
  ![](https://i0.hdslb.com/bfs/album/c830a2ec0ea83c2792b365dd49de0d6958ed6160.png@1e_1c.webp)
  
  可以看到1~6分布都比较平均，不错。

- **第三步，抽一组抽样来试试**
  
  我们接下来随便先拿一组抽样，手动算一下。例如我们先从生成的数据中随机抽取10个数字：
  
  ```python
  sample = []
  for i in range(0, 10):
      sample.append(random_data[int(np.random.random() * len(random_data))])
  print("抽样一次： ", sample)
  ```
  
  这10个数字的结果是：**[6, 4, 1, 3, 5, 6, 1, 5, 1, 4]**
  平均值：**3.4**
  标准差：**1.56**
  
  可以看到，我们只抽10个的时候，样本的平均值（3.4）会距离总体的平均值（3.5）有所偏差。
  有时候我们运气不好，抽出来的数字可能偏差很大，比如抽出来10个数字都是6。那平均值就是6了。 为什么会出现都是6的情况呢？因为我比较6…哦不是，因为这就是随机的魅力呀！
  
  不过不要担心，接下去就是见证奇迹的时刻

- **第四步，见证奇迹的时刻**
  
  我们让中心极限定理发挥作用。现在我们抽取1000组，每组50个。
  我们把每组的平均值都算出来。
  
  ```python
  samples = []
  samples_mean = []
  samples_std = []
  
  for i in range(0, 1000):
      sample = []
      for j in range(0, 50):
          sample.append(random_data[int(np.random.random() * len(random_data))])
      sample_np = np.array(sample)
      samples_mean.append(sample_np.mean())
      samples_std.append(sample_np.std())
      samples.append(sample_np)
  
  samples_mean_np = np.array(samples_mean)
  samples_std_np = np.array(samples_std)
  ```
  
  将结果样本均值分布和样本标准差分布画出来
  
  ```python
  matplotlib.rcParams['font.sans-serif']=['SimHei']  
  matplotlib.rcParams['axes.unicode_minus']=False    
  plt.hist(samples_mean_np, bins=400, facecolor="blue", edgecolor="black", alpha=0.7)
  plt.xlabel("样本均值")
  plt.ylabel("频数/频率")
  plt.title("频数/频率分布直方图")
  plt.show()
  ```
  
  <img src="https://i0.hdslb.com/bfs/album/3445b61e86cc7b7b822644ce5ad188e2e1f5a0e7.png@1e_1c.webp" title="" alt="" data-align="center">
  
  ```python
  matplotlib.rcParams['font.sans-serif']=['SimHei']  
  matplotlib.rcParams['axes.unicode_minus']=False    
  plt.hist(samples_std_np, bins=400, facecolor="blue", edgecolor="black", alpha=0.7)
  plt.xlabel("样本标准差")
  plt.ylabel("频数/频率")
  plt.title("频数/频率分布直方图")
  plt.show()
  ```
  
  ![](https://i0.hdslb.com/bfs/album/901427e3ea440988202488ade8b41fd447db1573.png@1e_1c.webp)
  
  从图中可以看到，两者的分布都倾向于正态分布

### 总结

<img src="https://i0.hdslb.com/bfs/album/18fe923c17e3464982323fad1c7363afb17e0953.png@1e_1c.webp" title="" alt="" data-align="center">

## 置信区间

我们希望通过选择a和b，使得总体统计量介于a与b之间这一结果具有特定的概率，假设这个概率为0.95，那么有

$$
P(a<\mu<b)=0.95
$$

我们用(a, b)表示这个区间，由于a与b的确切数值取决于你希望自己对于“该区间包含总体统计量”这一结果具有的可信程度，因此，(a,b)被称为置信区间。

求解置信区间的四个步骤：

1. **选择总体统计量**
   
   取决于要解决的实际问题，如果我们要想估计总体均值,那么就需要为总体均值构建置信区间

2. **求出其抽样分布**
   
   为了求出总体统计量的抽样分布，我们需要知道统计量的抽样分布，即需要知道样本统计量的期望和方差以及其分布，以均值作为统计量为例，其抽样分布的期望和方差为：
   
   $$
   E(\overline{X})=\mu,Var(\overline{X})=\frac{\sigma^2}{n}
   $$
   
   我们可以代入除想要求的统计量之外的所有数值，以均值为例，代入n,而$\sigma^2$ 真值未知，必须根据样本进行估计。
   
   那么用什么作为$\sigma^2$  的值呢，可以用它的点估计量进行估计，于是可以代入$\hat\sigma^2$ ,或者叫做$s^2$ 
   
   于是求得了我们所要的统计量的分布

3. **决定置信水平(常用95%)**
   
   置信水平越高，区间越宽，置信区间包含总体统计量越大
   
    **置信水平表明你希望自己对于“置信区间包含总体统计量”这一说法有多大把握，例如，假设我们希望总体均值的置信水平为95%，这表示总体均值处于置信区间中的概率为0.95**

4. **求出置信上下限**
   
   以置信水平95%为例，接着就是求a与b,即置信区间的上下限，上下限指出一个范围的左右边界——统计量有95%的概率落入这个范围中。a与b的确切值取决于需要使用的抽样分布以及需要具有的置信水平
   
   以总体均值作为统计量为例，我们需要让均值具有95%的置信度，即，$\mu$ 位于我们求得的a与b之间的概率必须为0.95，我们还知道样本均值符合正态分布。
   
   利用均值的分布可以求出a与b的值，即算出标准分，查询标准正态分布概率表，得出所需要的结果，计算如下：
   
   - 为了利用正态分布表，先对统计量进行标准化，以均值统计量为例，我们已知$\overline{X}\sim{N(\mu,0.25)}​$ ,于是，经过标准化计算，得到
     
     $$
     Z=\frac{\overline{X}-\mu}{\sqrt{0.25}}
     $$
     
     那么Z 服从标准正态分布，我们需要求出 $z_a$ 和 $z_b$ ，其中 $P(z_a<Z<z_b)=0.95$ ,其中 $P(Z<z_a)=0.025,P(Z>z_b)=0.0255$ ,利用概率表可以求出。
   
   - 用 $\mu$ 改写不等式，既可以得到 $\mu$的置信区间

有时候，总体的分布为正态分布时，样本统计量的分布并不是正态分布，比如当总体符合正态分布时，$\sigma^2$ 未知，且可供支配的样本很小时，$\overline{X}$ 符合t分布。

### 总结

<img src="https://i0.hdslb.com/bfs/album/5212fe35e2c6c17ab166984ab77c62eaed108231.png@1e_1c.webp" title="" alt="" data-align="center">

参考资料：<https://zhuanlan.zhihu.com/p/25241653>
