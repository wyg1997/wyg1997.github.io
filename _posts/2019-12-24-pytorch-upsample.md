---
title: pytorch的interpolate函数
layout: post
date: '2019-12-24 14:05:21'
tags: pytorch deep-learning upsample
color: rgb(255,90,90)
cover: "../assets/imgs/pytorch.png"
subtitle: pytorch中的上下采样
---

# torch.nn.functional.interpolate函数

* any list
{:toc}

## 参考文档

  1. [pytorch官方文档](https://pytorch.org/docs/stable/nn.functional.html#interpolate)
  2. [立夏之光的知乎文章：详细介绍了align_corners的涵义](https://zhuanlan.zhihu.com/p/87572724)

## interpolate

### 定义

```python
torch.nn.functional.interpolate(input, size=None, scale_factor=None, mode='nearest', align_corners=None)
```

### 说明

  - 根据给定的`size`或`scale_factor`参数对输入图像进行上/下采样操作。
  - 采样方式根据`mode`参数指定的算法。
  - 目前支持的数据格式有：
    * temporal：向量数据。期待**3D**张量的输入，格式为`mini-batch * channels * width`。
    * spatial：空间数据，例如图片。期待**4D**张量的输入，格式为`mini-batch * channels * height * width`。
    * volumetric：体积数据，例如点云数据。期待**5D**张量的输入，格式为`mini-batch * channels * depth * height * width`。
  - 可用的`mode`为：
    * 3D数据：nearest，linear。
    * 4D数据：bilinear，bicubic。
    * 5D数据：trilinear。
    * area：对于不同情况处理的方式有所不同，参考[Aaron Dong的知乎文章](https://zhuanlan.zhihu.com/p/38493205)，介绍的非常详细！

### 参数

  - input(Tensor)：输入张量。
  - size(int或Tuple[int, int]或Tuple[int, int, int])：表示输出的尺度。
  - scale_factor(float或Tuple[float])：缩小或放大的倍数。和`size`是互斥参数，只能指定一个。
  - mode(str)：可使用`nearest` `linear` `bilinear` `bicubic` `trilinear`和`area`。默认是`nearest`。
  - align_corners(bool, 可选参数)：几何上，我们认为输入和输出的每个像素是**正方形**而不是**点**。如果设置为`True`，输入和输出的张量在角像素**中心**对齐，这样可以**保留边界像素的值**。如果设置为`False`，输入和输出张量由它们的角点的角像素值对齐(具体说明见下面图片)，边界部分使用边界值的插值填充，**边界的像素值可能会丢失**。受影响的`mode`有`linear` `bilinear` `bicubic`和`trilinear`。默认为`False`。

#### 注意

如果`mode`使用`bicubic`时，可能生成的图像有**大于255或小于0**的值，可以使用`tensor.clamp(min=0, max=255)`做截断。

#### 警告

`0.3.1`版本之前默认值是`True`，之后默认值为`False`，使用别人的源码时要注意一下这个问题。

### 图解align_corners参数

参考 [立夏之光的知乎文章](https://zhuanlan.zhihu.com/p/87572724)。

#### 原图
![原图](/assets/imgs/v2-5787b6d21991d48a74873a2af3acef59_hd.jpg)

#### 上采样
![上采样](/assets/imgs/v2-17ac2006901e413a15274cf29567e8df_hd.jpg)

**左边为False，右边为True**。

#### 说明

两种方式在中间像素的插值上区别不大，主要差别还是边界像素。一般的任务中边界的值不那么重要，但是会影响语义分割的`mIoU`指标，使用`True`能更好的保留边界。

### 插值方式的区别

见[飞信天下的csdn blog](https://blog.csdn.net/google0802/article/details/8938849)。

### 已知bug

具体改动见[我的pull request](https://github.com/pytorch/pytorch/pull/29894)。

**CPU**模式下，如果`scale_factor`为1时，使用`bicubic`插值方式，本来预计的处理结果是把原图复制一下，不做任何处理。但是处理的时候忘了乘`mini-batch`了，导致复制的时候只复制了`n==1`的图，其它的图均为初始值`0`。