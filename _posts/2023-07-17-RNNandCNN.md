---
layout: single
title: "[AI LSTM]卷积网络与循环-长与短的记忆"
categories: [Web, AI]
last_modified_at: 2023-07-17
excerpt: "世纪影响力最大的人工智能算法，语音识别、机器翻译、自动写作、NLP等应用的核心"
header: 
  teaser: https://act-upload.mihoyo.com/bh3-wiki/2023/07/11/75216984/3f57387838c28d771d033a4c507126c1_6812554801947496218.png
---

## 卷积的神经网络 - 放大主要矛盾，减少训练量

![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/3391997e-3420-4df3-9141-626778146bf0)


水平方向延申

卷积神经网络（Convolutional Neural Network，CNN）是一种深度学习模型，专门用于处理具有网格结构的数据，例如图像和视频。CNN的基本层级包括输入层、卷积层、池化层和全连接层。
- 输入层：接收原始图像作为输入数据。
- 卷积层：通过应用一系列卷积核（也称为滤波器）来提取图像的特征。每个卷积核在输入图像上滑动，将局部区域与卷积核进行卷积操作，产生特征图。卷积层的输出是一系列特征图，每个特征图对应一个卷积核。
- 池化层：用于减小特征图的空间维度，同时保留主要特征。常用的池化操作是最大池化，它从每个局部区域中选择最大值作为池化后的值。
- 全连接层：将池化层输出的特征图转换为一维向量，并通过全连接层进行分类或回归等任务。
  
主要概念包括：
- 感受野（Receptive Field）：在卷积神经网络中，每个神经元的感受野指的是它在输入图像上影响的区域大小。感受野随着网络的层数增加而增加，较浅层的神经元感受野较小，较深层的神经元感受野较大。
- 特征图（Feature Map）：卷积层的输出称为特征图，每个特征图对应一个卷积核。特征图表示图像中的某种特定特征，例如边缘、纹理等。
- 参数共享（Parameter Sharing）：在卷积层中，每个卷积核在整个输入图像上共享参数。这意味着卷积核在提取特征时使用相同的权重，减少了模型的参数量，同时增强了模型对平移不变性的学习能力。 

## 循环的神经网络 - 增加隐藏层时序关联

![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/367b1808-7ec5-45f1-954f-f2bfadcd2b24)


添加了一个维度的变化（由上一次学习而来的短期记忆）

循环神经网络（Recurrent Neural Network，RNN）是一种神经网络模型，用于处理序列数据，如语言、音频、时间序列等。RNN的特点是具有循环连接，允许信息在网络中进行传递和持久化。
RNN的基本结构包括输入层、隐藏层和输出层。与其他神经网络不同的是，RNN的隐藏层在每个时间步都会接收上一时间步的输出作为输入，形成了时间上的循环连接。这种设计使得RNN能够对序列数据进行建模和记忆，具备处理时序信息的能力。

RNN的主要概念包括：
- 隐藏状态（Hidden State）：隐藏层在每个时间步的输出，包含了之前时间步的信息，并作为当前时间步的输入。隐藏状态可以看作是RNN对序列数据的内部表示。
- 门控单元（Gate Units）：为了增强RNN的建模能力和解决长期依赖问题，引入了门控单元，如长短期记忆（LSTM）和门控循环单元（GRU）。这些单元通过控制信息的流动和遗忘，有效地处理长序列数据。
- 双向循环神经网络（Bidirectional RNN）：为了更好地捕捉上下文信息，双向RNN结合了正向和反向两个方向的隐藏层，分别从两个方向对序列进行处理。

循环神经网络在自然语言处理、语音识别、机器翻译等任务中取得了显著的成功。它能够处理变长序列数据，并具备对上下文信息进行建模的能力，使得它在处理时序数据方面具有优势。

## LSTM长短期记忆网络 - 悠久的告别

![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/a36fab02-2d7e-41a4-a33e-d4fe85c3a0f9)

![image](https://github.com/Sumalene/Sumalene.github.io/assets/124686994/73c37d56-b7d0-4d22-b171-cdd9f40cef87)


多了一条与短期记忆交互的日记本链

LSTM（Long Short-Term Memory）是一种特殊类型的循环神经网络（RNN），用于解决传统RNN在处理长期依赖问题上的限制。LSTM通过引入门控机制，能够更好地捕捉和记忆长期的序列信息。
LSTM的核心思想是使用称为"门"的结构来控制信息的流动。它由一个输入门、遗忘门和输出门组成，每个门都有一个可学习的权重，用于决定信息是否通过。
- 输入门（Input Gate）：决定要将多少新信息纳入到当前时间步的隐藏状态中。 sigma*tanh 躺正切，梳理归纳
- 遗忘门（Forget Gate）：决定要从当前时间步的隐藏状态中丢弃多少旧信息。sigma函数，有点像relu取值0到1，矩阵相乘抹掉零，过滤重要的，忽略无关的
- 输出门（Output Gate）：决定要将多少隐藏状态的信息输出到下一个时间步。
  
通过这些门控机制，LSTM可以选择性地记住和忘记信息，从而有效地处理长期依赖关系。相比于传统RNN，LSTM在处理长序列数据和解决梯度消失/爆炸问题方面更为有效。
  LSTM在天气股市预测，自然语言处理、语音识别、机器翻译等任务中广泛应用。它能够捕捉上下文信息、处理长期依赖关系，并在处理时序数据方面展现出卓越的性能。
