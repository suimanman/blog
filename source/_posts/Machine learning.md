---
title: Machine learning
date: 2024-04-21 10:00:00
author: 岁漫
img: /images/ml
top: true
hide: false
cover: true
coverImg: /images/ml
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
