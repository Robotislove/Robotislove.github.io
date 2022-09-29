---
layout: post
title: AI嵌入式和智能计算系统总结学习笔记
date: 2022-09-29
author: lau
tags: [AI, Blog]
comments: true
toc: false
pinned: false



---

AI嵌入式和智能计算系统总结学习笔记。

## 概述

![](https://tuchuangs.com/imgs/2022/09/29/601eda7ed25768f9.png)

## 卷积及其矩阵加速

### 卷积操作的定义

卷积神经网络依旧是层级网络，只是层的功能和形式做了变化，可以说是传统神经网络的一个改进。卷积网络在本质上是一种输入到输出的映射，它能够学习大量的输入与输出之间的映射关系，而不需要任何输入和输出之间的精确的数学表达式，只要用已知的模式对卷积网络加以训练，网络就具有输入输出对之间的映射能力。

卷积就是卷积核和另外一个矩阵 (图片) 对应位置相乘然后结果相加。

![](https://www.mobibrw.com/wp-content/uploads/2019/04/conv.gif)

上图展示的是 `stride=1` 的情形, 既每次移动一个像素的位置, 根据需求, 也可以使用其他的 `stride`。

### 卷积计算的加速

对于大的卷积核, 加速方法一般是使用傅里叶变换 (或者其加强版: 快速傅里叶变换), 但是, 对于比较小的卷积核, 其转换到频域的计算量已经大于直接在空域进行卷

积的计算量, 所以, 我们会发现在主流的深度学习框架中, 一般是直接在空域中进行卷积计算, 其加速计算的方法就是把卷积操作转换成矩阵相乘 (因为有很多优化了

的线性代数计算库和 CUDA), 下面这张图充分说明了具体过程 (2 维的情形).

![](https://www.mobibrw.com/wp-content/uploads/2019/04/conv_matrix.png)

- 在上图中, `input features`每一个二维矩阵对应与 RGB 图像的 channel 或者是 feature map 中的 channel
- 目前常见的卷积都是 cross channel 的卷积, 即卷积核矩阵是 3 维的 (width, height, depth), depth 的大小和 feature map 的 depth.(depth 就是有几张 feature map, 不要被高大上的名词迷惑了)
- 三维卷积核的方法与 2 维的类似, 也是与 feature map 中相应的一个 3 维的矩阵对应位置元素相乘然后相加
- 2 维的卷积相当于 depth=1 的 3 维的卷积

### im2col in Python

```c++
import numpy as np
def get_im2col_indices(x_shape, field_height, field_width, padding=1, stride=1):
  # First figure out what the size of the output should be
  N, C, H, W = x_shape
  assert (H + 2 * padding - field_height) % stride == 0
  assert (W + 2 * padding - field_height) % stride == 0
  out_height = (H + 2 * padding - field_height) / stride + 1
  out_width = (W + 2 * padding - field_width) / stride + 1
  i0 = np.repeat(np.arange(field_height), field_width)
  i0 = np.tile(i0, C)
  i1 = stride * np.repeat(np.arange(out_height), out_width)
  j0 = np.tile(np.arange(field_width), field_height * C)
  j1 = stride * np.tile(np.arange(out_width), out_height)
  i = i0.reshape(-1, 1) + i1.reshape(1, -1)
  j = j0.reshape(-1, 1) + j1.reshape(1, -1)
  k = np.repeat(np.arange(C), field_height * field_width).reshape(-1, 1)
  return (k, i, j)
 
 
def im2col_indices(x, field_height, field_width, padding=1, stride=1):
  """ An implementation of im2col based on some fancy indexing """
  # Zero-pad the input
  p = padding
  x_padded = np.pad(x, ((0, 0), (0, 0), (p, p), (p, p)), mode='constant')
  k, i, j = get_im2col_indices(x.shape, field_height, field_width, padding, stride)
  cols = x_padded[:, k, i, j]
  C = x.shape[1]
  cols = cols.transpose(1, 2, 0).reshape(field_height * field_width * C, -1)
  return cols
```





## **参考链接**

- [CNN 基础之卷积及其矩阵加速](http://shuokay.com/2016/06/08/convolution/)
- [convnet-benchmarks](https://github.com/soumith/convnet-benchmarks)
- [卷积神经网络CNN总结](http://bigdata.51cto.com/art/201705/538792.htm)
- [卷积总结](https://www.mobibrw.com/2017/7557)



