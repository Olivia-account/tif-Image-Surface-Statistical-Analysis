# tif-Image-Surface-Statistical-Analysis
# 摘要
"图像表面统计分析" 是一个用于深入了解图像表面特性的工具，特别关注于统计分析和表面粗糙度的评估。该项目提供了一套功能强大的函数，用于计算图像表面的自相关函数、均方根高度以及相关长度。
# 项目介绍
"图像表面统计分析" 是一个用于深入了解图像表面特性的工具，特别关注于统计分析和表面粗糙度的评估。该项目提供了一套功能强大的函数，用于计算图像表面的自相关函数、均方根高度以及相关长度。
# 项目思路
1. 均方根高度计算： 通过计算图像表面的均方根高度，了解表面高度的整体波动程度。
2. 自相关函数分析： 利用自相关函数，揭示表面高度在不同滞后下的相关性，帮助确定表面的特征长度。
3. 相关长度测定： 通过寻找自相关函数下降至指定阈值的滞后值，计算相关长度，提供表面高度相关性的信息。
4. 图像数据处理： 通过去除图像中的零值，提高数据质量，确保统计分析的准确性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/4f75a3f1968a4cd98858dde09ec075d7.png)

# 具体代码
## 均方根高度计算
- data 是输入的图像表面数据，表示为一个二维数组。
- N 是数据的总长度，即图像中的像素总数。
- mean_height 是数据的平均高度，通过 np.mean(data) 计算得到。
- squared_diff 表示每个数据点与平均高度的差的平方的总和，通过 np.sum(np.square(data)) - N * np.square(mean_height) 计算得到。
- rms_height 是均方根高度，通过对 squared_diff 进行平方根运算得到。
整个计算过程遵循均方根高度的定义，即对每个数据点与平均高度的差的平方进行平均并取平方根。最终，rms_height 表示图像表面的整体波动程度。
在项目的上下文中，这个函数用于分析图像表面的高度分布，从而提供了关于表面整体波动情况的信息。

```python
def calculate_rms_height(data):
    N = len(data)
    mean_height = np.mean(data)
    squared_diff = np.sum(np.square(data)) - N * np.square(mean_height)
    rms_height = np.sqrt(squared_diff / (N - 1))
    return rms_height
```
## 自相关函数分析
- data 是输入的图像表面数据，表示为一个一维数组。
- j 是滞后值，表示自相关函数的滞后步数。
自相关函数的计算涉及两个步骤：
1. 计算自相关的分子：
  - numerator 使用 numpy 的数组切片操作，计算了数据在不同滞后步数下的自相关分子部分。
2. 计算自相关的分母：
  - denominator 计算了数据的平方和，即自相关函数的分母部分。
3. 计算自相关函数值：
  - rho 通过将分子除以分母得到自相关函数值。
整个函数的返回值 rho 表示在给定的滞后步数 j 下的自相关函数值。在项目中，该函数用于分析图像表面高度在不同滞后步数下的相关性，帮助确定表面的特征长度。

```python
def normalized_autocorrelation(data, j):
    N = len(data)
    
    numerator = np.sum(data[:N+1-j] * data[j-1:])
    denominator = np.sum(np.square(data))
    
    rho = numerator / denominator
    
    return rho
```
## 相关长度测定
- data 是输入的图像表面数据，表示为一个一维数组。
该函数的主要任务是通过迭代找到自相关函数下降至指定阈值（这里选择了1/e，为了归一化）的滞后值 j，然后计算相关长度 L。具体步骤如下：
1. 初始化参数：
  - j 初始化为 1，表示从滞后 1 开始搜索。
  - target_rho 设置为 1/e，为了确定自相关函数下降的阈值。
2. 迭代搜索：
  - 使用 normalized_autocorrelation 函数计算当前滞后值 j 下的自相关函数值 rho。
  - 如果 rho 小于等于 target_rho，则跳出循环。
3. 计算相关长度：
  - L 通过将滞后值 j 减去 1 并乘以 0.005 得到，以得到表面的相关长度。
最终，函数返回找到的滞后值 j 和对应的相关长度 L。在项目中，这个函数用于确定表面高度的相关长度，为了了解表面的特征尺度提供了关键的信息。

```python
def find_correlation_length(data):
    j = 1
    target_rho = 1/np.e  # 为了归一化，选择1/e
    while True:
        rho = normalized_autocorrelation(data, j)
        if rho <= target_rho:
            break
        j += 1
    L = (j - 1) * 0.005
    return j, L
```
## 图像数据处理
- data 是输入的图像表面数据，表示为一个二维数组。
该函数的目的是去除图像中的零值，以提高数据质量，确保后续的统计分析的准确性。具体步骤如下：
1. 创建数据副本：
  - data_copy 通过 data.copy() 创建输入数据的副本，以避免对原始数据的读取问题。
2. 提取非零值：
  - non_zero_values 使用布尔索引，提取数据中非零的数值。
3. 计算非零值的平均值：
  - mean_non_zero 通过 np.mean(non_zero_values) 计算非零值的平均值。
4. 用平均非零值替换零值：
  - data_copy[data_copy == 0] 选择所有零值，并将它们替换为平均非零值。
最终，函数返回一个经过零值处理的新数据副本 data_copy。在项目中，这个函数确保了图像数据中的零值不会影响统计分析的准确性。

```python
def remove_zeros(data):
    data_copy = data.copy()  # Make a copy to avoid read-only issues
    non_zero_values = data_copy[data_copy != 0]
    mean_non_zero = np.mean(non_zero_values)
    data_copy[data_copy == 0] = mean_non_zero
    return data_copy
```
# 项目链接
CSDN：https://blog.csdn.net/m0_46573428/article/details/136069515
# 后记
如果觉得有帮助的话，求 关注、收藏、点赞、星星 哦！
