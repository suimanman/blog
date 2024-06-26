---
title: Machine learning
date: 2024-04-21 10:00:00
author: 岁漫
img: /images/ml.png
top: true
hide: false
cover: true
coverImg: /images/ml.png
toc: false
mathjax: false
summary: 机器学习入门
categories: 自学
tags:
  - 机器学习
---

# 定义

两种类型：

* 监督学习（supervised learning)
* 无监督学习（Unsupervised learning) 

# 监督学习

## 回归

给数据集，包括输入和正确的输出，让机器进行学习并对未出现过的新的input进行预测并给出output。期间可以选择不同的算法，如图：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2013.47.27.png" style="zoom: 25%;" />

`这种数据预测称为回归(regression algorithms)`

| Input                      | Output                   | Application                    |
| -------------------------- | ------------------------ | ------------------------------ |
| Email                      | Spam(0/1)                | Spam filtering(垃圾邮件过滤)   |
| audio                      | Text transcripts         | Speech recognition（语音识别） |
| English                    | Chinese                  | Machine translating            |
| Ad,user info               | Click?(0/1)              | Online advertising（在线广告） |
| Image,rader info(雷达信息) | Position of other cars   | Self-driving car（自动驾驶）   |
| Image of phone             | Defect?(0/1)(是否有缺陷) | visual learning                |

## 分类

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2013.57.43.png" style="zoom: 25%;" />

`这种称为分类（classification)，结果是固定的几种分类，且结果不必须为数字，也可以是其他，比如预测是猫或者狗`

且input可以有很多种，在上述例子中，input：肿瘤大小，患者年龄，如下图：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2014.03.03.png" style="zoom:25%;" />

>在这些数据中，我们使用某种学习算法可以让机器找到类似上图的边界线以达到分类的效果。

## 区别

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2014.06.45.png" style="zoom: 25%;" />

# 无监督学习

和监督学习的区别：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2014.15.28.png" style="zoom:25%;" />

我们可以从上图中看到，无监督学习和监督学习的区别就是并没有给出输入相应的输出类别，而其目的就是通过分析给出的数据，找到数据之间存在的某种分类，也就是通过单纯的数据找到类似左图的分类。

## 应用举例：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2014.24.10.png" style="zoom:25%;" />

>在上图的例子中，因为新闻每天都在不断的更新变化，话题也就是分类会不断有新的出现，因此，我们无法及时的给出分类让其进行监督学习，无监督学习就显得尤为重要，上述例子用到了**聚类算法**，对于每天更新的新闻，机器自动找到新闻存在的共同点，并为其聚类，在用户进行搜索时，我们可以从上图中看到，网站为我们自动聚类出下面同样含有“panda”、“twin”、“zoo”单词的文章集群，并显示出来。

应用：

1. 聚类：anomaly detection,异常检测，比如金融系统中的欺诈检测。
2. 降维：dimensionality reduction,对于一个大数据集，将其压缩成一个小得多的数据集，并且丢失尽可能少的数据集

## training set（训练集）

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-21%2017.39.24.png" style="zoom: 25%;" />

监督学习过程：

例如对于房子面积和价格预测对例子，我们将数据用一个线性函数f将数据模拟到一条一次函数直线上。那么f就称为model（模型），

而y'即“y尖”称为预测值。其中f一般写为
$$
\begin{array}{c}
    f_{w,b}(x)=wx+b
\end{array}
$$

# 代价函数公式

Cost function:Squared error cost function

一般对于线性回归来说，平方差代价函数用的最多：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-22%2009.20.39.png" alt="image-20240422092046671" style="zoom: 33%;" />

选择合适的w,b来minimizeJ(w,b),即使得J最小

# 梯度下降

Gradient Descent:用来尝试最小化任何函数

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-22%2010.12.30.png" alt="image-20240422101240658" style="zoom: 25%;" />

>每次环顾360度，找到梯度下降的最大值的方向，然后前进，将会逐渐走到最小值。
>
>有趣的特性：在不同的起点，可能会走到不同的局部最小值。

## 梯度下降实现

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-24%2010.06.32.png" style="zoom: 33%;" />

>我们每次更新w和b,其中\alpha 为学习率（Learning rate):（0～1之间） 决定了你每次迈的步子的大小，也就导致下降的速度变快或慢。而后面的导数决定下降的方向，逐步更新w、b直到收敛，也就到达了局部最小值。注意我们需要同时更新w,b,见下图：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-24%2010.11.23.png" style="zoom:33%;" />

> 显然，左边是正确的，右边没有同时更新，而是用了更新后的w来更新b，这显然不正确。

## 梯度下降理解

我们先画出损失函数J(w)-w的曲线，并在曲线上选择一个初始点，如下图所示，此时，偏导数为正数，那么乘以一个学习率还是正数，最终结果就是w会不断减小，直至导数为0，到达局部最小值。

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-25%2013.00.06.png" style="zoom:25%;" />

## 学习率

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-25%2013.30.27.png" style="zoom:25%;" />

如图，学习率太小会导致梯度下降速度很慢，需要更多的迭代次数。学习率太大，会导致每次梯度跨度很大，甚至超过局部最小值，从而导致损失会越来越大。

# 多维特征(Multiple)

通常来说，feature都有很多，并不止一个，对于多维特征的符号表示如下图所示：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/image-20240426151023641.png" style="zoom: 25%;" />

## 多元线性回归(multiple linear regression)

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-26%2015.22.22.png" alt="image-20240426152230322" style="zoom: 25%;" />

## 向量化

在代码实现运算时，我们直接采用Numpy的向量运算函数dot，比写循环效率要高很多（硬件适配，会将每个乘积进行同时运算，并相加，这相比于逐个计算再相加要快很多）

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-26%2015.38.37.png" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-26%2015.39.15.png" style="zoom: 50%;" />

## 用于多元线性回归的梯度下降法

不同于仅有一个特征的梯度下降，多元梯度下降更新每个特征Wi是需要分别更新的，对每个特征属性分别求偏导，如下图：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-26%2021.12.43.png" style="zoom: 33%;" />

## 特征缩放——feature scaling

当我们有多个特征时，就会发现特征之间的取值大小范围是有差异的，而当这个差异很大时，对每个特征的w的取值大小就会很明显的影响到f的大小，如下图的上面两个是特征缩放之前，可以发现损失函数的等高线图变得高瘦，这是因为房子尺寸通常来说取值很大，如果采取正常的单位的话，相比较而言，纵轴的卧室数量一般就0-10之间很小的数。因此，当对于房子尺寸来说，w如果取一个很大的值的话，会导致损失函数变化很大，所以，我们需要对数据集的数值进行合理取值，最好是一个数量级，这样得到的损失函数等高线图就会变的更加圆，从而更容易训练，以更少的训练次数达到结果。

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-26%2021.30.43.png" style="zoom: 25%;" />

实现特征缩放：

1. 直接除以最大值即可缩放到0-1之间

2. 减去均值再除以最大减最小。

   <img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2015.54.53.png" style="zoom:33%;" />

3. 归一化函数z-score：正态分布

​	<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2015.55.59.png" style="zoom:50%;" />

## 判断梯度下降收敛

通过成本函数—训练次数图来判断：成本函数值随训练次数增加逐渐减少，最后趋于持平。

## 特征工程

通过经验和直觉对已有的特征变换或组合来创建新的特征，这个新特征更容易做出准确的预测。

# 分类

## 逻辑回归-logistic regression

对于一个二分类问题，如果用回归算法来分类，我们倾向于用下图这种曲线：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2017.44.28.png" style="zoom:33%;" />

这种曲线相较于直线来说，能更好的反应分类结果，那我们该如何简单的得到这个曲线呢？

`sigmoid function`：我们将属性z作为横轴，对于g(z)，下图中有，就能轻易得到这种曲线：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2017.48.06.png" style="zoom:33%;" />

所以，我们将f和g结合：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2017.50.18.png" style="zoom:33%;" />

就能很容易得到逻辑回归算法的式子，而当f结果为0.7（假设），代表分类结果为1的概率是70%，那么实际而言，假如是对于患者的肿瘤大小来推测肿瘤是良性还是恶性，就是良性的概率为70%。

## 决策边界

通常来说，对于像二分类这种问题，我们一般需要制定一个边界值来分类，比如我们采取的是f=0.5，那么也就是z>0的被分类为1，相反为0，那么f=0.5就是这个边界,也就是z=0。

而对于z=0来说，因为z=wx+b，我们目前的任务就变成找到合适的w和b来很好的将两种分类分开，并且w x+b=0的那条线就是数据分类的分界线。

举例：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2018.26.43.png" style="zoom: 25%;" />

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-28%2018.27.48.png" style="zoom: 25%;" />

## 逻辑回归中的代价函数

这里先给出两张图用来对比：在平方差损失函数下线性回归和逻辑回归的损失值随w,b的变化

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-29%2009.32.36.png" style="zoom: 25%;" />

我们可以看到逻辑回归中出现了很多的局部最小值，显然平方差损失函数就不再适用于逻辑回归。

这里我们采用单训练损失函数：对于训练集中y的值来确定损失函数用哪个。（里面L是loss损失函数，外层J是cost成本函数）

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-29%2009.45.19.png" style="zoom:25%;" />

`为什么上图中的cost函数是凸函数，也就是类似于线性回归中的平方差损失函数的函数图？`

在上面逻辑回归的介绍中，我们已经了解到f的输出一定是介于0～1之间，那么我们先分析y=1的情况，将-log(f)画出来：

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-29%2009.52.14.png" style="zoom:25%;" />

如右图所示：紫色的线是log(f)，蓝色的线就是我们的损失函数-log(f)，因为f是介于0～1之间，我们将这个区间取出来放大，左边就是放大之后的图，因为此时是对于y=1的损失函数-f的图，可以看到当f->1时，损失函数值Loss在逐步减小直到为0，相反f->0时，损失函数趋于∞，而我们的f就是对于y值的预测，可以发现时完全贴合。

同理，我们分析y=0:

<img src="https://raw.githubusercontent.com/suimanman/imgs/main/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/%E6%88%AA%E5%B1%8F2024-04-29%2009.58.01.png" style="zoom:25%;" />

因为log里面是1-f，而对于对数来说，1-f>0====>f<1;可以得到上图中右边的图，我们截取0～1区间并放大，左边是放大之后的图。因为此时预测的是y=0，所以就恰好和y=1时相反，f=0时预测很准，因此Loss为0，相反为∞。

## 简化逻辑回归的代价函数

我们先将两种情况合并，可以很容易得到下图第一行的式子（分别带入y=0/1即可得到之前的函数），再将这个损失函数代入到成本函数中，将负号提出：

`这个成本函数是经过统计学的最大似然估计得出的`

<img src="/Users/wangmeice/Library/Application Support/typora-user-images/image-20240429133107692.png" alt="image-20240429133107692" style="zoom:25%;" />

## 实现梯度下降



