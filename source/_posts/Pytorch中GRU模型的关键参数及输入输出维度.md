---
title: Pytorch中GRU模型的关键参数及输入输出维度
date: 2023-02-28 16:07:23
mathjax: true
tags:

 - Pytorch
---

## 一、GRU模型的关键参数

1. input_size：输入的特征数，代表输入序列数据中每个值由多少个特征表示，比如NLP中，每个单次由几维的词向量表示；

2. hidden_size：隐藏层特征数；
3. num_layers：代表GRU的层数；
4. bias：是否带有偏置， False则没有偏置，即$b_{ih}=0, b_{hh}=0$. 默认为True
5. batch_first：输入序列中，第一个维度是否为batch. 默认为False;
6. dropout：除最后一层，每一层的输出都进行dropout，默认为: 0
7. bidirectional：是否为双向GRU，默认为False；

<!-- more -->

## 二、GRU模型的输入和输出及维度

### 2.1 输入

1. GRU模型的输入为input和$h_0$，其中input是输入的序列数据，$h_0$是第一个GRU cell的隐藏层在每个batch中的初始值，$h_0$可以不提供，系统会默认生成一个符合维度要求的全零tensor。
2. 输入维度：
	1. input，当参数batch_first为False时，input的维度为(seq_size, batch_size, feature_size)，当batch_first为True时，input的维度为(batch_size, seq_size,  feature_size)；
	2. $h_0$的维度为(num_layers×GRU方向数，batch_size，hidden_size)；

### 2.2 输出

1. GRU模型的输出为output和$h_n$，其中output为模型输出值，$h_n$为模型最后一个GRU cell的隐藏层状态；
2. 输出维度：
	1. output的维度为(seq_size，batch_size，hidden_size×GRU方向数)
	2. $h_n$的维度为(num_layers×GRU方向数，batch_size，hidden_size)

> 1. GRU方向数：当bidirectional为True时，GRU方向数为2，当bidirectional为False时，GRU方向数为1；
> 2. seq_size为输入的单个序列长度，对于NLP代表了每个句子的长度；

## 三、对应关系

1. GRU设置参数中input_size与输入序列input的最后一个维度即feature_size对应；

2. GRU设置参数中hidden_size与输入$h_0$、输出output和$h_n$的最后一个维度对应；

3. GRU设置参数中num_layers与输入$h_0$、输出$h_n$的第一个维度中num_layers对应；

4. GRU设置参数中bidirectional与输入$h_0$、输出output和$h_n$的第一个维度中GRU方向数对应；

5. GRU输入和输出中seq_size、batch_size不变，但是最后一个维度由输入的feature_size变为输出的hidden_size×GRU方向数；

   

[循环神经网络 - torch.nn.GRU（）参数详解](https://blog.51cto.com/u_11299290/4727876)

[RNN, LSTM, GRU中输入输出维度](https://blog.csdn.net/sophicchen/article/details/108005115)
