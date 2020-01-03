---
author: wulei
comments: true
date: 2020-01-03 22:10:00+00:00
link: http://wuleiaty.github.io/2020/01/03/knn-character-trajectories/
slug: knn-character-trajectories
title: 使用k-NN算法进行Character Trajectories数据集分类
tags:
- k-NN
- 分类
- 动态时间规整
---

# 使用k-NN算法进行Character Trajectories数据集分类
## 算法原理
### k-NN算法
K最近邻算法是机器学习中常用的分类和回归算法之一，核心思想是样本的类别或预测值可以由其在样本空间中k个最接近的邻居来代表。

+ 在k-NN分类中，输出是一个分类族群。一个对象的分类是由其邻居的“多数表决”确定的，k个最近邻居（k为正整数，通常较小）中最常见的分类决定了赋予该对象的类别。若k = 1，则该对象的类别直接由最近的一个节点赋予。
+ 在k-NN回归中，输出是该对象的属性值。该值是其k个最近邻居的值的平均值。

在k-NN中，通过计算样本间的距离来作为样本间的相似性指标，通常使用欧式距离：
$$
d(x, y) = \sqrt{\sum_{k=1}^{n}{(x_k-y_k)^2}}
$$
但是在这个手写字母轨迹的数据集中，样本为长度不一样的时序数据，不能简单通过欧氏距离来计算相似度，使用动态时间规整算法(Dynamic Time Warping, DTW)来计算样本间的相似度。

### DTW算法
动态时间规整算法是一个计算时间序列之间距离的算法，常用于解决语音识别领域中不同语速发音情况下计算相似度的问题。相比于经典的欧氏距离，动态时间规整可以计算数据点个数不同的样本之间的距离，是因为该算法中的数据点可以一对多。DTW算法过程如下：
+ 给定两个序列，序列$X=(x_1,...,x_n)$，序列$Y=(y_1,...,y_n)$，给定序列中点到点的距离函数$d(i,j)=f(x_i,y_j)\geq0$（一般为欧氏距离）
+ 计算$X$和$Y$的距离矩阵$M$，即计算序列中数据点相对于另一个序列中每一个数据点的距离
+ 根据$M$计算累积距离矩阵$M_c$，计算过程：第一行第一列的元素为$M$矩阵的第一行第一列元素，其他位置的元素$M_c(i,j)=Min(M_c(i-1,j-1),M_c(i-1,j),M_c(i,j-1))+M(i,j)$，思想是动态规划求最短距离
+ $M_c$矩阵的最后一行最后一列元素就是序列$X$和$Y$的距离

## 算法实现
### 数据读取
数据集使用Matlab格式存储，使用scipy.io模块可以在Python中读取.mat格式的文件，通过Octave可以方便的查看数据集存放的结构。其中consts结构中的key键存放了样本标签的对应关系，consts结构中的charlabels键存放了每个2858个样本的标签，mixout结构存放了2858个样本，其中每个样本是一个3行n列的矩阵。实现如下：
``` python
def load_data(data_path) -> (list, list, list):
    # data_path: path of *.mat file
    
    assert os.path.isfile(data_path), 'wrong data path'
    data = loadmat(data_path)

    # read consts.key
    key = [data['consts'][0][0][3][0][i][0] for i in range(20)]

    # read consts.charlabels
    charlabels = [data['consts'][0][0][4][0][i] for i in range(2858)]

    # read mixout
    mixout = [data['mixout'][0][i] for i in range(2858)]

    return key, charlabels, mixout
```
### DTW算法
首先实现DTW算法中计算两个数据点的欧式距离：
``` python
def eucl_distance(vec1: np.ndarray, vec2: np.ndarray) -> float:
    return np.sqrt(np.sum(np.square(vec1 - vec2)))
```
然后通过构造累积距离矩阵的方式来计算两个序列之间的距离：
``` python
def dtw_distance(mat1: np.ndarray, mat2: np.ndarray) -> float:
    rows, cols = mat1.shape[1], mat2.shape[1]

    dp = np.zeros((rows, cols)).astype(np.float)
    for i in range(rows):
        for j in range(cols):
            dp_l = dp[i][j-1] if j != 0 else -1
            dp_t = dp[i-1][j] if i != 0 else -1
            dp_lt = dp[i-1][j-1] if i != 0 and j != 0 else -1

            dp[i][j] = min([item for item in [dp_l, dp_t, dp_lt] if item != -1], default=0) + eucl_distance(mat1[:, i], mat2[:, j])

    return dp[rows-1][cols-1]
```
由于DTW的计算复杂度高，使用Python计算非常耗时，所以使用C语言编写计算DTW距离的模块，在Python中使用ctypes来调用C模块。使用C编写的DTW算法的逻辑与使用Python实现的一致，所以就不贴出来。

### k-NN算法
首先计算测试样本与所有样本的距离，然后将得到的距离列表排序，取前k个元素即为k个最近邻，然后通过字典存放k个样本中每个类别的个数，最后根据字典的值排序得到个数最多的类别，即为测试样本的类别
``` python
def knn(sample_set: list, test_data: np.ndarray, k: int) -> str:
    dist = []
    for i in range(len(sample_set)):
        d = dtw_distance_c(sample_set[i]['data'], test_data)
        dist.append((d, sample_set[i]['label']))
        print('\r{:.3f}% ,sample {}, distance {:.3f}'.format(i / len(sample_set) * 100, sample_set[i]['label'], d), end='')
        
    dist = sorted(dist, key=lambda x: x[0])

    category = {}
    for i in range(k):
        category[dist[i][1]] = category.get(dist[i][1], 0) + 1
    category = sorted(list(category.items()), key=lambda x: x[1], reverse=True)

    return category[0][0]
```

### 主过程
实验的主要流程如下：
+ 读取数据，将数据组合成包含数据和标签的字典的列表，然后讲列表乱序，以便划分测试集出来
+ 读取输入的测试集数量和k值，然后划分测试集
+ 对测试集中的每个样本进行预测，最后计算准确率

``` python
if __name__ == '__main__':
    data_path = 'character-trajectories/mixoutALL_shifted.mat'
    key, charlabels, mixout = load_data(data_path)
    data = [{'data': mixout[i], 'label': key[charlabels[i] - 1]} for i in range(2858)]
    random.shuffle(data)

    test_set_num, k = int(input('number of test samples: ')), int(input('k: '))
    test_data, sample_data = data[:test_set_num], data[test_set_num:]
    
    correct_num = 0
    start = time.perf_counter()
    for idx, item in enumerate(test_data):
        print(f"{idx}/{test_set_num}, label: {item['label']}")
        pred = knn(sample_data, item['data'], k)
        print(f"\nlabel {item['label']}, prediction {pred}")
        correct_num += 1 if item['label'] == pred else 0
    
    print(f'{test_set_num} test samples, {correct_num} correct, correct rate {correct_num / test_set_num}, time {time.perf_counter() - start} s')
```
## 实验结果
本实验在Ubuntu的Python 3.7环境下进行。k取值为1，测试集数量300时，准确率为99%。实验结果如下
![](res/result.png)
不同k的取值情况下的实验结果如下表:

| K值 | 准确率 |
|------|------|
| 1 | 99% |
| 3 | 98.3% |
| 5 | 98.3% |
| 7 | 98.3% |




