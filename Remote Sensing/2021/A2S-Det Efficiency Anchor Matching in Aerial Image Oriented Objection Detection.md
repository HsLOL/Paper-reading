#### A2S-Det: Efficiency Anchor Matching in Aerial Image Oriented Objection Detection(2021)

* 文章生成多个固定角度、尺度和宽高比的锚框来召回真值标注，自己感觉该方法过于暴力，生成 的锚框太多，正负样本比例不均，用了一个动态设置阈值的模块，来分配正负样本。
* 变换了回归锚框的坐标系，由平面直角坐标系变换成了与锚框宽、高平行的坐标系。

#### 一、实验数据集  
1. 从DOTA数据集裁剪600*600的子图，然后缩放到800 *800作为输入，重叠部分为200；没有目标的子图被直接丢弃；训练图像有30250张。  
2. 在线数据增强：以50%的概率进行随机旋转（从0-360°中以15°为间隔）、随机翻转。  
#### 二、网络结构  
1. 主干网络：ResNet网络生成C3（1/8）、C4（1/16）、C5（1/32）特征图。

2. 特征融合网络：特征金字塔产生P3 - P7特征图，步长为(8，16，32，64，128)。

3. Anchor的生成

   宽高比：

   ![image-20210515140311781](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140311781.png)

   尺度大小：

   ![image-20210515140328149](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140328149.png)

   角度设置：

   ![image-20210515140351654](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140351654.png)

4. 回归网络：见论文提出的方法，`Focal loss损失函数`。

5. 分类网络：正常网络输出通道为len(anchor)×len(class)，`Smooth L1损失`。

![image-20210515140521255](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140521255.png)

6. 总损失函数：

![image-20210515140456560](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140456560.png)

 #### 三、论文提出的方法
1. **提出一种正负样本的分配策略（所有Anchor均参与正负样本的分配)**。

  （1）先计算所有的旋转Anchor与真值标注的`旋转IoU`，然后计`max(旋转IoU)`找到每个Anchor对应的最大真值标注，实现每个旋转Anchor都被分配给一个真值标注，Anchor与真值是多对一的关系。
  （2）计算给每个真值标注分配的Anchor与它的`水平交并比(HIoU)`，将大于`设定的HIoU阈值=0.6`的Anchor定义为候选框。

​       （3）将上步得到的候选框利用下面的公式，动态的设定阈值，获取正负样本，其中`Std`是标准差，用来反映集合内部的集中程度，通过让每个集合内部的标准差最小来获取相应的动态阈值。

![image-20210515131433788](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515131433788.png)

（4）最后根据生成的动态阈值，将Anchor与真值标注计算出来的旋转IoU大于阈值的设为正类，剩余的设为负类。

2. **旋转边界的回归方式**

   提出一种新的旋转框横纵坐标的定义及回归方式。原来采用的是与坐标轴平行的坐标系（b），现在采用与Anchor宽高平行的坐标轴（c）。

   ![image-20210515135450573](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515135450573.png)

   （1）坐标回归的表示方式
   $$
   \delta_x=(x-x_a)cos(\theta_a)-(y-y_a)sin(\theta_a)\\
   t_x'=\theta_x/w_a\\
   \theta_y=(y-y_a)cos(\theta_a)+(x-x_a)sin(\theta_a)\\
   t_y'=\theta_y/h_a
   $$

​       （2）角度的回归方式

![image-20210515140112653](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515140112653.png)

#### 四、实验结果

![image-20210515153120987](C:\Users\fzh\AppData\Roaming\Typora\typora-user-images\image-20210515153120987.png)

