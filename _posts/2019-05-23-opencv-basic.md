---
author: wulei
comments: true
date: 2019-05-23 21:14:00+00:00
slug: opencv-basic
title: OpenCV-Python基本操作
tags:
- OpenCV
- 图像处理
---

### OpenCV-Python安装
OpenCV是广泛使用的开源计算机视觉库，提供了Python接口，方便进行快速的原型开发。在Ubuntu上安装OpenCV-Python，使用如下命令
``` bash
sudo apt install opencv-python
```
在程序中使用OpenCV，做如下的包引入即可：
``` python
import cv2
import numpy as np
```
在使用OpenCV的同时，几乎都需要同时使用Numpy，Numpy是广泛使用的数值计算库，OpenCV的使用离不开Numpy，比如使用OpenCV打开的图片返回的是一个标准的Numpy数组。

### 图像的基本操作
#### 读取图像
使用`cv2.imread()`函数可以读取一个图像，这个函数指定两个参数，第一个参数指定图像，第二个参数指定打开方式。第二个参数可选如下一种：
+ `cv2.IMREAD_COLOR` 默认的打开方式，加载一个彩色图像，图像的任何透明度都会被忽略
+ `cv2.IMREAD_GRAYSCALE` 以灰度模式打开一个图像
+ `cv2.IMREAD_UNCHANGED` 读入一个图像并且包括alpha通道，alpha通道是一个特殊的通道，主要用来保存图像的非彩色信息。
第二个参数也可以使用１、０、-1来代替以上的参数，当然也可以不指定，则用默认的打开方式。

打开图片示例：
``` python
import numpy as np
import cv2

# Load an color image
img = cv2.imread('Lenna.png')
```
其中img是一个Numpy数组。
 
#### 显示图像
OpenCV提供方便的GUI接口来显示图像，使用`cv2.imshow()`函数即可，这个函数有两个参数，第一个参数用来指定一个窗口名，第二个参数指定显示的图像：
``` python
cv2.imshow('image',img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
这样创建了一个名为image的窗口并在其中显示img图像，并且监听键盘，然后摧毁所有的窗口。

#### 色彩转换
OpenCV提供了接口转换图像的色彩，比如将RGB图像转换为灰度图像，使用`cv2.cvtColor()`函数，第一个参数指定图片，第二个参数指定要转换的色彩，比如
``` python
grayImg = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```
这行代码将图像转换为灰度图像。使用`img.shape`即可查看Numpy数组的结构，可以看到，img是一个三维数组，包含了RGB三个通道的二维数组，而转换后的grayImg则是一个二维数组，存储了图像的灰度值。

#### 转换图像尺寸
使用`cv2.resize()`函数可以按指定的宽高转换图像的尺寸。第一个参数指定图像，第二个参数是一个包含了宽高的像素值的元组。比如
``` python
smallImg = cv2.resize(grayImg, (100, 100))
```
使用`smallImg.shape`可以看到图像的结构已经转换为`(100,100)`

### 视频的基本操作
OpenCV提供了接口可以方便的读取和显示视频，使用`cv2.VideoCapture()`函数打开一个视频，使用`read()`函数可以从视频对象中提取帧，这样就可以将每一帧显示出来以显示视频。如下
``` python
import numpy as np
import cv2

cap = cv2.VideoCapture(0)

while(True):
    # Capture frame-by-frame
    ret, frame = cap.read()

    # Our operations on the frame come here
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # Display the resulting frame
    cv2.imshow('frame',gray)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# When everything done, release the capture
cap.release()
cv2.destroyAllWindows()
```
### Example-视频转字符动画
``` python
import cv2
import numpy as np
import os

charSet = ['@', '#', '&', '$', '%', '*', 'o', '!', ';', '.']

def getChar(gray):
    return charSet[9 - int(gray / 25)]

if __name__ == '__main__':
    cap = cv2.VideoCapture(0)

    i = 1
    while(True):
        ret, frame = cap.read()

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        smallimg = cv2.resize(gray, (100, 100))
        
        charImg = ''
        for i in range(100):
            for j in range(100):
                charImg += getChar(smallimg[i, j])
            charImg += '\n'

        print(charImg)
        os.system('clear')

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    
    cap.release()
    cv2.destroyAllWindows()
```
