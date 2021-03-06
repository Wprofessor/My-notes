# pbject detection



#### 一、object detection技术的演进：RCNN->SppNET->Fast-RCNN->Faster-RCNN



![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112530304-2103953078.jpg)

#### 二、CNN已经比较好地解决了识别的问题

#### 那么检测问题我们可以大致转化成什么问题呢？

* #### 回归问题：需要预测出(x,y,w,h)四维坐标，得出方框的位子

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112638429-1953242676.png)

​	步骤：

​		步骤1、一个cnn		![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112659914-1900232742.jpg)

​	步骤2、将cnn尾部展开，加两个“头”（分类头和回归头） 

​	将原本的classification模式改成：classification+regression模式

​	![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112723757-880743532.png)

​	步骤3、Regression部分使用欧式距离计算损失，然后结合梯度法进行优化

​	步骤4、预测阶段将2个头拼接上来完成不同的功能

​	那么regression部分应该加在哪儿？

​	加在最后一个卷积层后或者是最后一个FC层后

regression的训练参数收敛的时间要长得多，所以上面的网络采取了用classification的网络来计算出网络共同部分的连接权值。

* #### 取图像窗口

  还是刚才的classification+regression思路

  取不同大小的框

  让框出现在不同位子，得出这个框打判定得分

  取出得分最高的框



![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112903273-1900432759.jpg)

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112919320-1728574836.jpg)

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112933164-1200242604.jpg)

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504112949320-428298146.jpg)

如果有两个框得分相同，就取两个框的交集部分

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113014179-105680354.jpg)

但是这个方法太耗时了，所以需要优化。

原本是：

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113030523-41422116.jpg)

优化后：

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113039304-1103823779.jpg)

#### 物体检测

当我们需要“划出”的东西很多，而且种类也很多的时候怎么办？

任务就变为：多识别+多物体定位

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113057601-300431852.jpg)



所以不能看成暴力枚举分类问题：因为“框”实在太多了

所以就改为：找出可能有物体的框（候选框：如1000个），这些框之间是可以互相重叠或者包含的

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113129211-135695982.jpg)

下面列出一些常见的选框方式：

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113209164-1717059035.jpg)



基于上面的描述——RCNN就出来了

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113229570-69371857.png)

第一步：训练一个分类模型（拿AlexNet举例）

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113247742-406451407.jpg)

第二步：对该模型做fine-tuning处理（将ALexnet分类数从1000改为20，去掉最后一个全连接层）

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113302195-1032285194.png)

第三步：特征提取：

提取出图像的所有候选框（选择性搜索）

对于每一个区域：修正区域大小以适合CNN输入，做一次向前运算，将第五个池化层的输出（就是对候选框提取到的特征）存到硬盘

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113326195-1862868537.png)

步骤四：训练一个SVM分类器（二分类）来判断这个候选框的物体的类别

每个类别对应一个SVM，判断是不是属于这个类别

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113349351-169304797.png)

步骤五：使用回归器修正候选框的位子：训练一个线性回归模型去判定这个框是不是最好的框

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113412351-143526289.png)

### SPP Net

spp:spatial pyramid pooling（空间金字塔池化）

结合空间金字塔的方法实现CNNs的对尺度输入

一般CNN后都接全连接层或者分类器，他们都需要固定的输入尺寸，因此不得不对输入数据进行crop或者warp，但是这样处理会造成数据的丢失或者几何失真

那么我们就加入spp layer。此时网络的输入可以是任意尺度的，在SPP Layer中每个filter会根据输入调整大小，但是输出尺度是固定的

![](https://images2015.cnblogs.com/blog/1093303/201705/1093303-20170504113433539-94801265.jpg)

