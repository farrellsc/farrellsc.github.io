---
layout:     post
title:      "Detection"
subtitle:   ""
date:       2018-06-25 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - AI
    - CV
    - Detection
---

> Old notes

# CV

### Detection

#### Object Detection（物体检测）
- 问题定义：
  - 给定图片，输出给定物体（比如车、人脸）的Location，Location通常用Bounding Box（包围矩形框）表示
  - 通常还要输出Detection Confidence（置信度），因为下游模块需要根据Confidence来设定阈值
  - 通俗地说Detection可认为是两个任务Classification（给出Object Confidence）和Regression（给出Bounding Box）的组合
    - 注：如果是用scanning window来做detection，那么就不包含Regression任务
- 如何评判Detection的性能：
  - 先说明IoU(Intersection over Union，简译为"交除并")：指两个Object的重合度，一般认为Inference与GT的IoU>0.5算是做对了
  - 评价Detection常用以下2个指标：
    - mAP，是Paper上常用的指标：mAP指mean Average Precision, 其中mean指多个类的平均值，Average指在多个IoU阈值上的平均值（传统上使用0.5~0.95间的10个值），参见：http://mscoco.org/dataset/#detections-eval
    - Recall/FA，是实际任务中常用的指标：Recall指做对的正例数量/全部正例的数量，FA指的是误报数量/图片数
- 相关的下游模块，以及如何设定IoU阈值：
  - Detection有非常多的下游模块，最常见的是Alignment、Tracking以及Recognition
  - 如何设定Evaluation时的IoU阈值
    - 如果是作为比较通用的算法模块（即下游的可能性有很多种），那么可以选定个经典值，如0.5IoU描述性能
    - 如果是作为特定模块的输入，那么就按照下游模块的需求设定IoU

#### 基于CNN的Detection方法
基于CNN的Object Detection常用的有2种方式，Two-Stage或One-Stage：
- Two-Stage
  - 算法显式分成2个网络和步骤：提取Region Proposal的RPN网络，以及基于Region Proposal进行分类和定位的网络
  - 相关Paper的工作有：Faster R-CNN系列（R-CNN，Fast R-CNN，Faster R-CNN，RBG大牛的三连发），R-FCN
- One-Stage（或称之为One-Shot）
  - 算法用端到端1个网络解决所有问题
  - 相关Paper地工作有：YOLO系列（YOLO，YOLOv2)，SSD系列（SSD，DSSD等）
- 二者的网络结构可以见以下附图，摘自Google 16年的Paper：Speed/accuracy trade-offs for modern convolutional object detectors
![](/img/in-post/2018-06-25-Detection/Detection-Fig1.png)

- 关于两类方法的优劣比较
  - 谨慎地说，相同水平的选手在做到同样速度时，Two-Stage的性能可以做的更好，同时Two-Stage的实现也更为复杂
  - 再表述一遍，上述观点是非常谨慎的，因为Detection的Trick很多，实现的好坏差距会很大
  - Google 16年的Paper对几种方法做了比较详尽的比较：[Speed/accuracy trade-offs for modern convolutional object detectors](https://arxiv.org/abs/1611.10012)
- 除了以上2种方式的划分，还有些横向的Ropic，主要是网络结构/LossFunction等方面：
  - 基础的网络结构：如果想性能做的更好，用ResNet；如果想做的更快，用MobileNet；等
  - 网络结构的改进：Kaiming.He做了几个对网络改进的工作，例如FPN，在Two-Stage/One-Stage上都能使用

#### SSD Detection方法
本篇Tutorial主讲SSD方法，具体参考Paper，对SSD方法的Summary如下：
SSD抛弃了Faster R-CNN为代表的传统Region Proposal的方式，转变成Single Shot(YOLO), End-to-End Training & Inference
SSD网络由Base Network, 和每个Feature Map上的卷积Detector组成
除了Single Shot外，SSD的Key Feature是：
相对于YOLO仅使用单个Layer的FeatureMap，SSD在不同深度的多个Layer进行检测，较好地处理了Scale的问题
与Faster R-CNN的Anchor类似，使用了Default Box，把问题空间离散化
在原始的SSD Paper上，性能与速度概要如下（以下比较其实是有bias的，更客观的对比可以看上面的Google Paper）
在PASCAL VOC上，SSD比Faster RCNN的mAP高了1-5个点，速度从7FPS提高到20-50FPS，且输入分辨率低六倍
在PASCAL VOC上，SSD和YOLO速度相近，mAP从58提高到75，接近20个点
SSD包括了SSD300和SSD512两个版本. SSD300快三倍，但mAP低约2个点
SSD方法的若干缺陷
性能上，小物体的检测性能较差
原始Paper在Training上，有大量的参数/细节，包括两个较难理解Data Augmentation和Hard Negative Mining部分
SSD Detection方法-Model
对于SSD方法，主要理解下面3张图：
1. Multi-Layer Prediction
如下图，在Base VGG网络上，在6层Layer上进行detection，detection的结果一起做次NMS，作为输出。
每层上的detection均采用3x3的卷积核，channels数量为#default_box * (#class + 4), 常数4代表location的四个变量xmin, ymin, xmax, ymax

2. Default Box
在Feature map cell上设置1个或多个Default Box，对于1个GT box，可能会有多个Default Box与其mapping，如下图，蓝色GT框所表示的猫，与2个不同Shape Ratio的Default Box mapping，这2个Default Box所对应分类器的预测目标都是该GT box
3. Loss Function
Loss Function由2部分组成，第1部分是confidence loss(softmax loss)，第2部分是localization loss(smooth L1 loss)



SSD Detection方法-Inference
Inference方法并无特殊之处：过一遍网络，拿到所有layer上的predication结果，做次NMS，即是输出
SSD Detection方法-Training
Training中有几个要素注意：
Default Box的相关策略
最重要是Shape Ratio&Scale，这2个参数决定了能否拿到基本能看的结果；Paper中采用了3种Shape Ratio(1:1, 1:2以及1:3)，Scale方面则是使用了以原图比例为0.2～0.9为等差数列的Scale
其次是与GT Box的mapping关系，这里不详述，作为一个问题
Data Augmentation：
首先Data Augmentation是非常重要的（想下为什么？）
Paper种做了各种花式的Data Augmentation，做与不做在性能上有8.8%的差异
对小物体的专门处理，有3%的性能差异
Hard Negative Mining：
在每张图上，只选用分数最高的K个负例计算loss，保持正例和负例的比例为1:3（想下为什么？）
你知道吗？
“你知道吗？”中的内容不影响正文的阅读，更多是参考资料，需要自行查阅
Detection是个经典问题，同时有庞杂的相关工作，这里将稍微展开做讨论




#### Reference:
- [R_CNN](https://arxiv.org/pdf/1311.2524.pdf)
- [Fast R-CNN](https://www.cv-foundation.org/openaccess/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
- [Faster R-CNN](https://arxiv.org/abs/1506.01497)
- [YOLO](https://arxiv.org/abs/1506.02640)
- [YOLO v2](https://arxiv.org/abs/1612.08242)
- [SSD](https://arxiv.org/abs/1512.02325)
- [FPN](https://arxiv.org/abs/1612.03144)
- [Focal Loss](https://arxiv.org/abs/1708.02002)
- [Speed/accuracy trade-offs for modern convolutional object detectors](https://arxiv.org/abs/1611.10012)

