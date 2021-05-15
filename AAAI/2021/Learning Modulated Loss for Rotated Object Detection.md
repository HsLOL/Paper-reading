#### Learning Modulated Loss for Rotated Object Detection

* 将旋转目标检测过程中的5参数回归过程中由于角度的周期性和宽高突然交换造成的损失不连续（间断）的问题定义为`旋转敏感损失`，并提出了调制旋转损失来解决旋转过程中的边界约束问题，使训练过程中的训练曲线更加的平滑。

#### 导致损失函数不连续的第一个问题--->角度问题

##### 问题说明：

> 下图说明了当一个框位于边界位置时，角度仅顺时针或者逆时针变化1°，不仅角度变化超级多，而且宽高的定义也发生了巨大的变化，造成回归损失相差很大。

![image-20210515181250012](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515181250012.png)

>  自己对这个理解是，假如真值标注是红色框，现在有一个绿色框，原本只要角度参数稍微更新一点就可以，但是由于回归损失变化很大使得导数变化也很大，最终导致参数更新的过大而造成了不匹配。

##### 解决方式：

对5参数回归过程提出调制损失

![image-20210515202121201](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515202121201.png)

![image-20210515202130983](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515202130983.png)

##### 8参数的调制损失函数

##### 存在的问题

![image-20210515203932377](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515203932377.png)

理论上根据坐标点的定义，蓝a->绿a，蓝b->绿b,...，但实际上比较好的回归方式是蓝a->绿b，蓝b->绿c，这种情况会导致损失不连续。

##### 解决方案

（1）先将预测框的四个顶点顺时针移动一个位置

（2）保持预测框的坐标点顺序不改变

（3）将预测框的四个顶点逆时针移动个位置

（4）获取上述三种情况下的最小值

#### 导致损失函数不连续的第二个问题--->5个回归参数有不同的测量单位

>由于不同的参数与交并比的关系曲线并不一致，对不同参数的损失进行简单的加和再对损失函数求导的方法更新参数完成回归会存在问题。
>
>下图说明了不同的参数与IoU曲线间的关系，其中宽度/高度与IoU是先正比后反比的关系，图（a）；中心坐标与IoU的关系是对称关系，图（b）；角度与IoU的关系为多项式的关系。

![image-20210515194523965](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515194523965.png)

#### 本文提出的方法

1. 发现5参数检测器范式存在角度变化不一致以及宽高突变问题造成的损失变化不一致，使用8参数回归范式，又指出8参数回归范式也存在着固有的损失变化不一致问题。



#### 实验数据集(DOTA)

将图像从原始数据集的原图上以600 * 600进行裁剪，重叠率为150个像素，然后缩放到800 * 800的大小进行训练。

数据增强策略包括：随机水平翻转、随机垂直翻转、随机图像灰度化和随机旋转。

针对DOTA数据集中类别不平衡问题，对某一（几）类数据进行复制。

#### 网络结构：采用RetinaNet网络结构 