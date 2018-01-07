---
layout: post
title: "RCNN,SPPNET,Fast-RCNN,Faster-RCNN"
tags:
---
RCNN是目前最准确的图像目标检测方法之一，一直是[Imagenet][1]获奖的热门算法。

最早的[RCNN][2]是Ross Girshick提出的一种通用的目标检测方法，将深度网络引入到目标检测任务中，使得Pascal VOC challenge的mAP一下子提高了20%多。虽然RCNN算法已经有了许多改进，但是重读RCNN论文还是可以很多收益，特别是和当时其它将CNN引入目标识别算法比较，为什么RCNN可以将进度提高这么多？ 该论文有两大创造:
第一点是新的基于Region的目标识别算法RCNN。RCNN的算法示意图如下![R-CNN Algorithm Diagram](../rcnn.png)。RCNN先由Region Proposal算法（具体算法可有多种选择，比如selective search，CPMC）识别存在潜在目标的区域，再由CNN生成前面得到区域的特征，最后由目标类数目个SVM分类器识别建议区域中有无物体，或者是什么物体。为了提高目标识别的mAP，RCNN还使用了回归算法进一步提高获取目标Bounding Box的位置精度，因为mAP的计算还要检测目标和标准结果的重合程度。
第二点论文还介绍了可以利用辅助数据集进行预训练CNN，来解决训练数据较少的问题。因为Pascal VOC的训练数据较少，而ILSVRC 2012数据集的样本就很多，先在较多的样本上进行预训练，可以防止出现过拟合的情况。也说明CNN网络具有很好的抽象能力，能够表示与具体数据集无关的语义。

RCNN发表之后的一年微软亚洲研究院的He Kaiming等就发表了SPPNET。SPPNET在CNN中添加了Spatial Pyramid Pooling Layer![SPP](../spp.png)，CNN可以不限制输入图像的尺寸(AlexNet等网络因为全连接层限制，只能处理固定长宽的图像)。由于RCNN中Region Proposal是在输入图像上生成的，有可能生成2000多个Region Proposal，RCNN计算非常耗时，即使使用GPU加速RCNN一次检测需要十几秒钟时间。其实许多Region Proposal是互相重叠的，有一些CNN计算是重复的。He Kaiming的SPPNET就可以解决这个问题，他们将RCNN的Region Proposal从CNN的输入挪到了最后一个卷积特征上，在每一个Region Proposal上使用SPPNET输出固定长度的特征![RCNN+SPP](../rcnn-spp.png)，再和RCNN一样使用SVM识别是哪一类物体。这个新方法可以在1s内检测一张图片，而且保持和RCNN一样的准确性。

作为对SPPNET的回应，Ross很快发表了Fast-RCNN，提出了更快，更准的算法。该算法相对于RCNN有如下改进：
1. 和SPPNET一样对整个图像只做一遍CNN计算。Fast-RCNN的做法是将输入图像上的Region Proposal区域映射到CNN的输出特征上，在输出特征上叠加了Roi Pooling Layer，无论Region的大小输出长度固定的特征。Roi Pooling Layer有点像只包含一个尺度的Spatial Pyramid Pooling。
2. RCNN的类别识别和物体定位是两个分开的步骤，怎么样训练类别识别SVM和物体位置回归网络就是一个需要考虑的问题，而且需要更长的训练时间。Fast-RCNN使用一个合一的代价函数将两个网络的训练合到了一起，可以将物体识别SVM和定位回归网络一起训练到最佳状态。

虽然Fast-RCNN已经够好了，但是Ross和Ren Shaoqing，He Kaiming等人仍然找到了提高的地方，那就是Fast-RCNN使用的Region Proposal算法成了计算瓶颈。Faster-RCNN提出了Region Proposal Network
