# **4.也见森林：视觉SLAM简介**

构成我们学习最大障碍的是已知的东西，而不是未知的东西。——贝尔纳

---

| **本章主要内容**<br />1.什么是视觉SLAM<br />2.视觉SLAM的主流框架及分类 |
| --- |

前三章都在讲解相机，刚体运动相关内容，属于一个铺垫，那么什么是SLAM呢，在本章进行介绍。
<a name="vOt0A"></a>
## **4.1 SLAM**
SLAM为Simultaneous Localization and Mapping的缩写，意为同时定位与建图。这有啥用呢，想象一下到了一个陌生环境中，比如一下被扔到阿尔卑斯山某个山谷中，你需要找到出去的路，给自己打完气后，就出发了，往前面走啊走，走啊走，你突然发现目前这个场景好熟悉，是不是哪里见过。不太肯定，也没心思去管这个了，继续走啊走，走啊走，你才大呼一声，这特么不是我刚才踩断的那截树根么。你站在这个还很新的断面接口前方，双方仿佛大眼瞪小眼相互看着，有着些许的尴尬，这时候你想，要是能把走过的地方都记录下来，并从中提取出方向信息，确定自己的位置，那该多好啊。<br />是的，SLAM就是干这个事的。利用载体上的各种传感器，记录环境的同时估计载体的空间姿态，最后生成一张可用于地位的地图，就是SLAM的使命。<br />![](https://cdn.nlark.com/yuque/0/2023/gif/1782698/1683986340140-48dfd9b5-f1f1-4693-adcf-87e5ca65c933.gif#averageHue=%231f2636&id=qPEET&originHeight=240&originWidth=320&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />这里的载体，指能在空间中运动的载体，移动机器人，车辆，手持设备等都可以算是移动载体。而要达到对环境及自身运动的估计，则需要传感器。传感器目前主要分为两类，测量值依赖外界环境的，与测量值依赖自身运动情况的。依赖外部环境的传感器如相机，激光雷达。相机需要收集外界环境光线，而雷达需要接收外界物体表面反射的激光。依赖自身运动的则如一些编码器与imu，imu测量得到的加速度与载体运动情况有关。载体与传感器的关系有如身体与眼睛的关系一样，载体为传感器提供了搭载的平台，而传感器则是载体感知世界的来源，计算机模仿大脑处理传感器信息，经过一系列算法，把传感器采集的数据不断拼接为环境的模型，并估计出自身的位姿。
<a name="hKBdx"></a>
## **4.2 视觉SLAM**
视觉SLAM，也称为VSLAM，指利用视觉传感器来进行建图与定位的技术。常用的视觉传感器就是相机，而相机有很多种，如单目相机，双目相机，深度相机等等。这类相机统称为视觉传感器。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986340610-386f7fd1-8350-4ae0-a4f0-cf02626650d9.png#averageHue=%23bfbfbf&id=eLkwV&originHeight=399&originWidth=774&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />直白来说，视觉SLAM，就是要用图像这个信息，去认识世界，定位自己。
<a name="N1jcC"></a>
## **4.3 视觉SLAM主流框架**
<a name="QUN40"></a>
### **4.3.1 主流框架**
经过多年的发展，目前VSLAM框架已趋近成熟，总体说来，分为前端估计，后端优化，回环检测，地图构建这四个模块。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986341029-ab1a3a21-9be7-4df0-97a4-6b539fc24849.png#averageHue=%23caccc6&id=RA8aS&originHeight=449&originWidth=1114&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />首先接收传感器数据后，看数据情况会有一个预处理过程。之后前端会对传感器数据进行特征提取及特征追踪，这样可以得到前端的一个里程计估计及短中期的一个数据关联。前端信息经过一定筛选，留下一些有代表性的信息帧，传递给后端，同时前端特征也会与历史帧进行匹配，检测是否有回到之前到过的地方，如果有，也会把这个信息传递给后端，后端使用这个回环检测信息及全部关键帧信息进行全局的优化，得到最优的观测量及姿态值的估计，最后把姿态及特征保存为地图。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986341446-f17a4890-5a09-4d89-857a-75b75449b67e.png#averageHue=%23f3f4d8&id=ccZ6I&originHeight=413&originWidth=669&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />总结来说，需要一个模块处理原始传感器数据，然后数据给到前端模块进行粗加工，提取其中的特征来短时间估计当前移动情况及特征的追踪情况（前端），然后把所有这些信息给到后端，作为优化的初始值。同时前端特征会与历史观测进行匹配（回环检测），并把回环检测结果发送给后端。后端把所有信息进行最优化估计（最大后验估计），得到最优的地图，最后生成一张可用来定位或者导航的地图（建图）。
<a name="eq9qP"></a>
### **4.3.2 视觉SLAM分类**
在以上框架中，按照前端使用不同的特征，可以把当前视觉slam分为直接法，间接法，及联合使用这两类的混合法。直接法使用图像的灰度梯度信息，按照使用的图像范围，分为稠密，半稠密，稀疏。而间接法，则从图像中提取一些特征点，提取完特征点后便舍弃掉整幅图像，所以间接法也叫基于特征的方法。而接着按后端中使用的优化技术方案，又可以把上述几个大类接着细分为基于滤波的方法，基于非线性优化的方法。<br />同时，也有按照算法适配的相机类型，把VSLAM算法分为单目算法，双目算法及RGB-D算法的。下图是几个类别的VSLAM代表算法及出现的时间，可以看出最早的VSLAM算法是基于特征的单目算法，之后出现了混合算法，再最新出现了直接法。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986341855-c1ab9e71-3243-4cf5-a625-e3eb5fb30882.png#averageHue=%23b6b590&id=Nbxal&originHeight=429&originWidth=907&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />而对于后端优化，则是最先出现的滤波技术，然后出现了非线性优化技术。滤波计算量小，速度快，但结果精度差于非线性优化。非线性优化分为滑窗优化及全局优化，计算量大，使用的观测更多，速度慢一些，但目前取得了最高精度。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986342240-10d242e7-6971-4715-87c4-d3cde59d8b9f.png#averageHue=%23f8f1eb&id=PcPVx&originHeight=329&originWidth=589&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />目前直接法的代表算法，有LSD，和慕尼黑工业大学的DSO<br />间接法代表算法，有MonoSLAM，PTAM，ORB_SLAM<br />混合法代表算法，有SVO
<a name="rm6an"></a>
### **4.3.3 单目VSLAM的一个工作示例**
简单来说，单目VSLAM的工作流程为提取特征点->特征点追踪->系统初始化->初始位姿估计->关键帧判断->局部BA->回环检测->全局BA。这里我们以经典的ORB_SLAM为例，讲一个单目VSLAM是如何工作的。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1693616383576-afd6cccf-c7a1-4e4b-87fc-76b3ed0bf71d.png#averageHue=%23a6927a&clientId=u349385ce-be18-4&from=paste&height=438&id=u2e773e45&originHeight=579&originWidth=723&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=183964&status=done&style=none&taskId=u16e83959-b50f-4950-87f1-bd4d01c9c7e&title=&width=547.4000244140625)<br />上图是ORB_SLAM（ORB_SLAM1）的系统总览。整个系统分为追踪（Tracking）,局部建图（Local Mapping）和回环检测（Loop Closing）三大线程，并维护地图及视觉词典两大数据结构。<br />**1.特征追踪**<br />当接收到每一帧图像数据时，追踪线程首先在图像上提取角点特征（ORB特征）。在系统刚启动时，单目VSLAM系统由于缺乏尺度信息，需要先利用对极几何进行初始化。这个时候需要相机在空间中运动一小段距离以保证对极约束的有效性，ORB_SLAM在初始化中会开两个线程，对本质矩阵和单应矩阵同时进行估计，选择效果最好的模型进行初始化。<br />初始化完成后，系统就获得了带尺度信息的初始地图。对于后面新接收到每一帧图像，再利用特征点的匹配关系，就可以计算出当前帧的位姿。在获得当前帧位姿之后，ORB_SLAM会在局部地图中将具有共视关系帧上的特征点也投影到当前帧上，以寻找更多的匹配点，并利用新找到的匹配点再次对相机位姿进行优化，这一步称为局部地图追踪。在完成这些步骤之后，会根据当前帧相对于上一帧运动的距离及匹配到的特征点数量，判断当前帧是否有条件成为一个关键帧，如果是关键帧，则会进行局部建图，如果不是，则继续处理下一帧。<br />**2.局部建图**<br />在新生成一个关键帧之后，便会进入局部建图线程。新的关键帧的插入会使ORB_SLAM更新其“生成树”（spanning Tree）及共视图（Covisibility Graph）及本质图（Essential Graph）。这些数据结构维护关键帧之间的相对位移关系及共视关系。更新完这些数据结构后，就完成了关键帧的插入。<br />接下来是对不可靠地图点的剔除，即能观测到该地图点的关键帧数量少于一个阈值，这个地图点作用就不大了，此时会对其进行剔除。剔除完之后就是新地图点的生成，对于新检测到的特征点，如果和之前未匹配上的地图点产生了匹配，这个特征点就可以恢复出深度，并产生新的3D地图点。之后对新的关键帧及与其有共视关系的局部地图帧，及其它们能看到的地图点，会进行局部BA，通过新的约束关系提高局部地图精度。局部追踪和局部BA利用了中期数据关联，是ORB_SLAM精度高的一大原因。在完成局部BA后，ORB_SLAM会对冗余的关键帧（一帧上的大部分的地图点，能被其他关键帧看到）进行剔除。这种关键帧的剔除机制，在ORB_SLAM在称为适者生存机制，是其能在一个地方长期运行，而关键帧数量不至于无限制增长的关键设计。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1693617287889-9cb75199-cdec-4f2f-ba20-3c23d8b2b0d3.png#averageHue=%23faf6f6&clientId=u349385ce-be18-4&from=paste&height=452&id=u6b8f5b7b&originHeight=706&originWidth=661&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=249892&status=done&style=none&taskId=uab014820-c249-4032-822d-3298e6166b9&title=&width=422.79998779296875)<br />**3.回环检测**<br />对于每一个关键帧，在进行完局部建图后，会进入到回环检测阶段。首先ORB_SLAM会使用DBoW库检索可能的回环帧，对检测结果会进行几何一致性检验。通过几何一致性检验后，认为发生了回环，此时会计算当前帧与回环帧之间的Sim3关系，即平移，旋转加尺度。利用这个信息，建立当前帧及其共视图，与回环帧及其共视图之间的约束关系，这一步就称为回环融合。最后，对带回环约束关系的本质图进行优化，完成对整个地图的调整。如果没有发生回环，则会把当前帧的特征描述子信息加入到视觉词典中，以与后面新的关键帧做匹配。<br />在这整个系统中，特征追踪部分称为前端，而局部建图，回环检测部分，则称之为后端。前端主要职责是检测特征，并寻找尽可能多的数据关联。后端则是基于前端给的初始估计及数据关联进行优化，消除观测误差及累计误差，生成精度尽可能高的地图。
<a name="Tx2Oj"></a>
## 本章小结
介绍了什么是SLAM及其主流框架，VSLAM的主要分类。
<a name="QurXg"></a>
## 本章思考
1.VSALM中间接法与直接法的主要区别在什么地方，其各自的优势是什么？<br />2.SLAM中前端与后端的关系是什么？
<a name="Vosvu"></a>
## **参考**
[1.视觉/视觉惯性SLAM最新综述：领域进展、方法分类与实验对比](https://zhuanlan.zhihu.com/p/393381346)<br />[2.SLAM技术综述](https://zhuanlan.zhihu.com/p/501102444)<br />[3.视觉SLAM:模型介绍、算法框架及应用场景](https://zhuanlan.zhihu.com/p/623111690?utm_campaign=shareopn&utm_medium=social&utm_oi=662363645720268800&utm_psn=1632301428428447744&utm_source=wechat_session)
