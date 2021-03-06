# DeepSafe
2022年计算机设计大赛人工智能赛道作品部署源码

我们使用到的环境有如下: 

python 3.7 

Anaconda3 

Pycharm 

cuda10.2cudnn-10.2-windows10-x64-v7.6.5.32 pytorch1.5.1-gpu

我们运用到的工具包以及其建议版本如下:

Keras==> 2.2.4 

opencv-python==>4.2.0.32 

numpy==> 1.16.2
1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
tensorflow==> 1.14.0 

sklearn==> 0.20.3 

scipy==> 1.2.1 

我们使用 Anaconda3 来安装所有必要的软件环境：

安装 opencv： conda install opencv 

安装 mingw libpython： conda install mingw libpython 

安装 theano： conda install theano 

安装 tensorflow-gpu： conda install tensorflow-gpu 

安装 sklearn： conda install scikit-learn 

安装 matplotlib： conda install matplotlib 

安装 keras： condainstall keras 

### [1.1.1 YOLOv5s -]()目标检测

捕捉速度快，运算量较小，能实现多目标多样本的同时捕捉。行人检测与行人重识别如何更好的进行结合。reid使得行人检测可以存在多一些的误检，要求recall比较高，因为一些背景可以通过reid进行相似度比较从而排除掉。但是如过置信度阈值设的太低，会有很多的proposal要进入reid模型，降低模型的速度。因此，需要进行合理的设置，本作品就设置了conf-thres=0.1，nms-thres=0.4，最后，对行人重识别模型进行剪枝、蒸馏，加快推理速度。

本作品以无人机为载体，视频造成模糊的原因有：①快速移动 ②面部失焦 ③视频噪声，这会大大影响检测的精度，所以我们对数据集进行运动模糊（模仿快速运动）、平均模糊（模仿失焦），高斯模糊（模拟随机噪声）来增强处理，更好的适应实际应用，如图：

**                              **

图 5 初始图像及三种模糊图像****

### [1.1.2 DeepSORT -]()多目标追踪

目标跟踪速度快，精准度高，可绘制物体轨迹，便于追踪个体。无人机能进行播报警告，当存在感染风险时，无人机将发出提醒或警报，并回传数据，本作品的追踪流程如下图所示：

 

图 6 多目标追踪流程图

结合口罩佩戴情况与社交距离分别在安全或有感染风险时标记绿色或橙色边界框：

 

图 7 效果展示

### [1.1.3 DBSCAN -]()人群聚类

DBSCAN算法 是一种基于密度的聚类算法：

• 聚类的时候不需要预先指定簇的个数。

• 最终的簇的个数不定。

 

 

### [1.1.4 社交距离计算]()

在正式应用社交距离之前，需要两种类型的距离校准，以实现像素距离到现实距离的转换。

**1.****相机视图到顶视图的转换**

通过机载摄像头的拍摄角度计算变换矩阵将摄像机视图转换为俯视图图像。

** **

图 8 视图转换****

2**.**** ****比例因子的估计**

人的身高约为 5.6 英尺（头部和脚部位置的像素点间距）, 通过算法估计比例因子转换以英尺为单位的像素距离。

 

图 9 人的身高

通过距离校准实现像素距离到现实距离的转换，评估感染风险(试验场景：本校学生活动中心)。

  

图 10 距离的转换

## [1.2 作品特色]()

**1. ****防疫工作覆盖面广**

为YOLO检测与Deepsort多目标追踪算法提供了极佳的鸟瞰图视角，保证AI视觉的有效执行，实现“应检尽检、不漏一人”。

 

图 11 无人机展示

**2. ****立体式作业**

根据像素距离计算社交距离，评估感染风险并以高空自动播报的方式执行宣传任务，突破道路限制，迅速穿梭在城市的每个角落，强机动性使宣传效果和人员安全达到最大化。

 

图 12 风险评估流程

详细需求流程设计如下：

主要分为两部分：行人检测聚类与社交距离计算，大体流程如下：

\1. 使用YOLOv5模型检测输入视频或视频流中的所有行人，得到行人坐标，并根据此计算位置信息和质心位置；

\2. 根据第1步的信息，计算所有检测到的行人的人质心之间的相互距离；

\3. 根据上一步计算得到的社交距离，进行人群聚类分析，对比预先设置的安全距离，从而计算每个人之间的距离对，结合口罩佩戴情况，检测两个人之间的距离是否小于N个像素，小于则处于安全距离，反之则不处于安全距离。

 

图 13 详细需求设计流程

# [第2章 系统实现]()

我们的作品是基于openbayes与pytorch实现的，首先在openbayes上完成数据集的标注与发布，使用AI市场算法进行训练，训练资源可选Ascend 910或GPU；然后在openbayes (Beta)中转换模型，调试代码，实现行人检测与跟踪以及简单人数统计，整体开发在OpenBayes上完成，从零开始，最终进行Serving模型部署上，独立运行。

## [2.1 数据的加工和扩增]()

因为有些数据背景的复杂性，相关目标需要标注，网上有可以自动标注的服务但是由于需要付费，于是我们最终花了几天时间手动标注了数据，并生成了一些负样本，防止训练的过拟合。 

因为实验前期的数据量不太可观，担心数据的数量不能很好的使得网络达到良好的训练效果，尤其是遇到小目标时，于是我们主要采用了几种数据扩增的方式： 

（1） Mosaic 

Mosaic 是把四张训练图片缩放拼成一张图，Mosaic 有利于提升小目标的检测，这是因为一般在数据集中小目标在图片中分布不均匀，这导致在常规的训练中小目标的学习总是不太充分。使用 mosaic 数据增强后，在遍历每个张图片包含了四张图片具有小目标的可能性就很大了，同时，每张图都有不同程度的缩小，即使没有小目标，通过缩小，原来的目标尺寸也更接近小目标的大小，这对模型学习小目标很有利。 

（2） Cutout 

Cutout 是随机选择一个固定大小的正方形区域，然后采用全 0 填充，同时 为了避免填充 0 值对训练的影响，对数据进行中心归一化操作，norm 到0。 

（3） 矩形训练 

通常 yolo 算法解决输入图像大小比率不同的方法是直接缩放和填充到固定的大小，但是这样将导致有很多冗余的信息，会让网络产生很多无意义的候选框。矩形训练就是将图像 resize 到变成可以被步长整除并且最接近需要输入的大小，从而实现最小填充。 

## [2.2 数据管理]() 

将处理好的数据先分成多个目标类文件，再生成对应的正样本和负样本样例，在训练时将正样本和负样本组合打乱。
