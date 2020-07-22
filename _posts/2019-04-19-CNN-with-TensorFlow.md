---
layout: post
title: 'TensorFlow中CNN有关（基础）'
date: 2019-04-19
author: Dave
tags: TensorFlow
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/imgTF-CNNbasic.jpg)

博客迁移，原文链接[TensorFlow中CNN有关（基础）](https://blog.csdn.net/ddpiccolo/article/details/89392084)

>最近学习，主要记录
### Input
>```tf```中一般用一个```4-D```的```tensor```来表示
其```shape```为```[batch_size, in_width, in_height, channels]```，
### 卷积核
>一般也用一个```4-D```的```tensor```来表示，其```shape```为```[filter_width, filter_height, input_channels, output_channels]```，卷积核移动的步长```strides```一般为```[1, strides, strides, 1]```.

举例
输入为```6x6```的单通道（1）图片，每一次输入```batch```数目
>```[batch_size, 6, 6, 1]```

### Convolution
```python
h_conv2d = tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')
```

> + 卷积计算函数
> + ```stride [1, x步长, y步长, 1]```
> + ```padding:SAME/FULL/VALID```（边距处理方式）

```32```个```5*5```卷积核，其```shape```为```[5,5,1,32]```
### Padding 
可参考[TensorFlow中CNN的两种padding方式“SAME”和“VALID”](https://blog.csdn.net/wuzqchom/article/details/74785643)
* ```padding='SAME'```(尺寸填充为原尺寸)
* >模型尺寸分析：卷积层全都采用了(填充)补0，所以经过卷积层长和宽不变，只有深度加深。池化层全都没有补0，所以经过池化层长和宽均减小，深度不变。
  ```python
  out_height = ceil(float(in_height) / float(strides[1])
  out_width  = ceil(float(in_width) / float(strides[2]))
  ```


* ```padding='VALID'```(尺寸不够就缩小)
* >模型尺寸分析：经过卷积层长和宽往往都缩小，
  ```python
  out_height = ceil(float(in_height - filter_height + 1) / float(strides[1]))
  out_width  = ceil(float(in_width - filter_width + 1) / float(strides[2]))
  ```

  
### Max Pooling
 
 ```python
 h_pool = tf.nn.max_pool(x, ksize=[1,2,2,1], strides=[1,2,2,1], padding='SAME')#2x2的池化窗口 步长为2  
 ```

```2*2```的```max_pooling```，其形状为```[1,2,2,1```]，其步长```strides```为```[1,2,2,1]```

> + max池化函数
> + ```ksize [1, x边长, y边长,1]```池化窗口大小
> + ```stride [1, x步长, y步长, 1]```
> + ```padding:SAME/FULL/VALID```（边距处理方式）

### 激活函数
全连接层一般使用```sigmoid```或者```tanh```
卷积层一般使用```tanh```，```ReLu```及其改进型
可参考
[常用激活函数（激励函数）理解与总结](https://blog.csdn.net/tyhj_sf/article/details/79932893)
[深度学习之损失函数与激活函数的选择](https://blog.csdn.net/lien0906/article/details/78430162)


