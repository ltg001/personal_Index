# 面试项目相关备忘录

因为一度面到自闭 把做的东西梳理一下

## [车道线识别](https://zhuanlan.zhihu.com/p/35134563)

### 基础步骤

1. 镜头畸变 & 失真系数 ===> 矫正畸变
    镜头畸变种类：
    1. 枕型 边缘放大率大于中心放大率（正向畸变）
    2. 桶型 中心放大率高于边缘放大率（反向畸变）
    3. 线性 垂平面不正交
2. 得到车道线的二进制图
    边缘：灰度值变换较大的像素点的集合
    提取车道线 ===> 边缘检测
        sobel 算子
        canny 算子 ===> 计算x轴 y轴颜色梯度变化导数 进行阈值过滤 *得到二值图像*
    效果较差：使用颜色空间进行滤波 如hls空间的s通道
3. 透视变换得到鸟瞰图 ===> 看说明
4. 提取边界
    拟合车道线（滑动窗多项式拟合） 从图像底部开始 迭代扫描到顶部 将检测到的像素添加到列表里 若在一个窗口里检测到足够数量的像素 下一个窗口讲义他们的平均位置为中心
    得到每个车道线的像素后拟合一个多项式 产生平滑曲线来近似
5. 计算曲率
    曲率公式 $[1 + f'^{2}]^{3/2}/f''$ 带入之前拟合的多项式求解即可

### 相关说明

1. Sobel 算子

卷积核为 3x3 通过横向和纵向对图像进行卷积 得出横向和纵向亮度的差分近似值 // 在横向和纵向上不同 将其中的 2 改为 sqrt(2) 即可得到各向同性的的卷积核（也是分横向和纵向的 更加精确）
每个像素的梯度大小为两个的欧式距离 方向为 arctan(Gy/Gx)

```text
Gx = [[-1 0 1],
      [-2 0 2],
      [-1 0 1]]

Gy = [[-1 -2 -1],
      [ 0  0  0],
      [ 1  2  1]]

```

2. Canny 算子
     1. 高斯滤波：用一个高斯函数去乘一个像素及其邻域 得到加权的平均值作为这个点的灰度值
     2. 计算梯度和方向：通过 Sobel算子（一阶偏导的有限差）等方式得出x轴和y轴方向的灰度梯度 由上述的方式进行每个点的梯度方向计算 并将角度map到差为45°的角度中 ([0~22.5] & [157.5~180] => 0)
     3. 过滤非最大值（非极大抑制）：使边缘的宽度更短 检测梯度方向上这个像素点是否是梯度最大的点 若是 则为边缘 否则将其灰度置为0
     4. 使用上下阈值进行检测：有两个阈值(threshold) 若大于大阈值的认为是边界 小于小阈值的认为不是 介于中间的若和确定为边缘的像素邻接则认为是边缘 否则认为非边缘

3. 透视变换 == 三维坐标变换
    保持 “直线性” 之前是直线的变换后还是直线 ===> 通过车道线等宽选点
    基于四个点计算变换矩阵 将原始图像和变换矩阵相乘即可得到变换后的图像

## NLP

### TransE

基础 将entity和relation都抽象为k维的单位向量 然后通过衡量cos距离进行判断合适程度

$loss = sigma(max(0, gama + distance(head + relation, tail) - distance(head' + relation, tail))) $ 其中distance为cos距离

输入：知识 前件 后件 关系
输出：对应embedding出来的向量
步骤：

1. 随机初始化 $[uniform(-6/sqrt(k), 6/sqrt(k)]*k$
2. sgd 后 单位化
3. 根据验证集上的效果停止迭代

优点：开创了实体embedding到低维向量的方法
缺点：在单一的向量平面上进行embedding 造成本不应该特别相关的向量特别近 仅适用于一对一的知识

### TransH

输入：知识 前件 后件 关系
输出：对应embedding出来的向量

### TransR