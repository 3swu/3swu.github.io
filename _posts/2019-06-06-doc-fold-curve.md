---
author: wulei
comments: true
date: 2019-06-07 22:47:00+00:00
slug: doc-fold-curve
title: 如何生成折叠和弯曲的文档图像
tags:
- OpenCV
- 图像处理
---

最近为了复现一个文档反变形的论文，第一步需要准备数据集。论文中数据集使用生成的各种弯曲和折叠的文档，原平整的数据集作为ground truth，生成后数据作为训练集。

### 折叠和弯曲变形的原理
折叠和变形都是通过像素的位移来实现的，实现折叠和弯曲效果的原理如下：
1. 随机选取一个变形像素$p$，作为初始的变形像素
2. 随机选取$p$像素的初始变形方向和程度，使用$v$表示
3. $v$会根据权重传播到其它的每一个像素，每一个像素的变形由$p_{i}+wv,\forall i$计算
4. 初始变形像素和变形方向构成了一条直线，对所有的像素，计算像素到这条直线的距离$d$，变形权重$w$是关于$d$的函数
5. 定义$w$，对于每一种变形种类，有其对应的$w$定义方式。对于折叠变形：$$ w=\frac {\alpha}{d+\alpha}, $$
对于弯曲变形：$$ w=1-d^{\alpha}, $$
$\alpha$控制了变形传播的程度。如在折叠变形中，$\alpha$越大，$w$越接近1，使得变形传播的损失越小

### 实现
首先导入程序需要的包以及读入灰度图像，并且生成目标图像的矩阵，初始化为白色图像：
``` python
import cv2
import numpy as np
import random
import math

src_img = cv2.imread('test.jpg', cv2.IMREAD_GRAYSCALE)
dst_img = np.ones(src_img.shape) * 255

rows = src_img.shape[0]
cols = src_img.shape[1]
```
获取初始的变形像素，为了使这个随机像素不落在图像的边缘，对生成范围做了一些约定
``` python
# get a random vertex
vertex = (random.randint(100, rows - 100), random.randint(100, cols - 100))
print('random vertex: ', vertex)
```
随机生成变形的方向和程度，方向$\theta \in(0, 2\pi)$，变形距离可根据具体的图像尺寸调试，并且计算斜率
``` python
# get random deformation direction and strength
v = (random.uniform(0, 2 * math.pi), random.uniform(40, 60))
print('direction and strength: ', v)
# evaluate the slope
k = math.tan(v[0])
print('slope: ', k)
```
计算每一个像素到直线的距离，使用一个和图像相同形状的二维Numpy数组存储每一个点的距离
``` python
# function for distance
def distance(k, vertex, point):
    c, b, a = k * vertex[0] - vertex[1], -1 * k, 1
    return abs(a * point[0] + b * point[1] + c) / math.sqrt(a**2 + b**2)

distance_array_2d = np.array([distance(k, vertex, (x, y)) for x in range(rows) for y in range(cols)]).reshape((rows, cols))
```
对于原图像的每一个像素，计算其变形之后在目标图像中的位置，但是这样的方法生成的目标图像会有撕裂，因为有一些像素进过计算之后会落在目标图像中的同一个像素，这样目标图像中的一些像素不会被映射，造成图像撕裂。所以反过来计算目标图像中的每一个像素是原图像中的哪一个像素变形而来，就可以解决图像撕裂的问题
``` python
for x in range(rows):
    for y in range(cols):
        alpha_folds, alpha_curves = 250, 2
        # for folds
        w = alpha_folds / (distance_array_2d[x][y] + alpha_folds)
        # for curves
        # distance normalization
        # w = 1 - (distance_array_2d[x][y] / (rows / 2))**alpha_curves

        src_x, src_y = x - int(v[1] * math.cos(v[0]) * w), y - int(v[1] * math.sin(v[0]) * w)

        if (src_x < 0 or src_x >= rows) or (src_y < 0 or src_y >= cols):
            continue
        else:
            dst_img[x][y] = src_img[src_x][src_y]

cv2.imwrite('dst.jpg', dst_img)
```
其中折叠和弯曲的参数$\alpha$可以通过调试得到。值得一提的是，在弯曲的$w$计算公式中，需要先对距离进行归一化处理。在图像中，距离往往可以达到几百上千，而在公式$w=1-d^{\alpha}$的函数图像中，当$d>1$时，$w$为负且指数级下降，这样会造成图像过度弯曲，最后的效果中，图像近似只有一条直线。
