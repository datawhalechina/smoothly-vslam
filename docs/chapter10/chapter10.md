# **10.又回到曾经的地点：回环检测**

学习这件事不在乎有没有人教你，最重要的是在于你自己有没有觉悟和恒心。——法布尔

---

> **本章主要内容** \
> 1.回环检测的作用 \
> 2.视觉常用回环检测方法 \
> 3.使用词袋模型进行回环检测

前面几个章节介绍了VSLAM系统中前端和后端，这里介绍下一个重要模块-回环检测。回环检测作为VSLAM中相对独立又重要的一个模块，在消除累积误差，重定位等方面有着重要作用，更有甚者，把系统是否有重定位模块，来作为判断一个系统是SLAM还是里程计的依据，那么接下来我们看下到底什么是回环模块，其为什么会有这么重要的作用。
<a name="CGTVb"></a>
## 10.1 回环检测
回环检测顾名思义，就是检测当前是否有回环。如下面所示，在场景A中，我们从1点出发，沿着某条路线走，如果没有碰到之前的场景，我们的路线一直处于开环状态。当到达地点3时，属于之前走过的地方，路线闭合，形成闭环，这个闭合过程称为发生了回环。如果我们继续沿着之前的路线走（下图中白色虚线框内路径），则会不断发生新的回环，继续产生多个回环点。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686450269185-5402bb19-b1e3-41f2-a6c4-ba6629da887e.png#averageHue=%23515642&clientId=ud4f25423-17a1-4&from=paste&height=393&id=uf9786643&originHeight=674&originWidth=987&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=1060710&status=done&style=none&taskId=uf9f6454d-c770-4257-8453-1e0e83f2b85&title=&width=576)<br />为什么会有回环检测呢，或者说回环检测的作用是什么。这一切都和里程计的累计误差有关系。在上述场景，我们理想中的轨迹是这样的：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686451887713-d9cdfd3b-e916-4bf4-b836-38867a008a0e.png#averageHue=%234f5541&clientId=ud4f25423-17a1-4&from=paste&height=312&id=u75b944cb&originHeight=498&originWidth=713&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=578752&status=done&style=none&taskId=u864d934e-804f-4b99-ad27-80a867d0c3a&title=&width=446.4000244140625)<br />由于里程计的累计误差，走的越远越不准，最后实际估计的路线可能是下图这样，可以看到，从左侧开始，轨迹就逐渐开始偏掉了：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686452217459-bbb476c9-e224-477a-a45d-a9ef437572c9.png#averageHue=%234f5440&clientId=ud4f25423-17a1-4&from=paste&height=308&id=uc25a5108&originHeight=498&originWidth=708&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=581055&status=done&style=none&taskId=u76c20418-775d-4a30-bf81-41d5d0670eb&title=&width=438.4000244140625)<br />导致最后我们回到交叉路口时，人是回来了，估计的轨迹没有回来（下图左），整个轨迹与实际相比，出现了很大的偏差。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686453615749-821054ef-eef7-40d9-93a0-4bcf69f2c664.png#averageHue=%23535743&clientId=ud4f25423-17a1-4&from=paste&height=401&id=ue02c3143&originHeight=501&originWidth=1278&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=520958&status=done&style=none&taskId=u77cb3306-3bb9-4544-b837-ce951a76b90&title=&width=1022.4)<br />如果能通过算法检测到当前发生了回环，即上图中的两个绿点，其实是同一个地方，就可以通过匹配算法计算两个绿点之间的相对位置关系（黄线），通过这个相对位置关系，可以得到3点真实的位姿。把这个相对约束加到优化中，就可以把轨迹拉到正确的位置上，得到一个全局一致的姿态估计。<br />所以我们说，回环检测是消除里程计累计误差的好帮手。想在长时间，大范围的姿态估计中获得好的结果，回环检测是不可或缺的。更极端一点的是，我们会把只有前端和局部优化的算法叫做里程计，而有回环检测和全局优化的算法，称为SLAM算法。<br />另外也可以发现，回环检测算法也可以用在重定位之中。检测到回环后，可以通过匹配算法计算回环位置与当前位置之间的相对位置关系，然后得到当前的真实位姿。因此在定位阶段，回环检测的原理也能发挥重要的作用，回环检测算法真是一个应用很广泛的算法。
<a name="EjrbN"></a>
## 10.2 回环检测的方法
现在回过头来，如何做回环检测呢？对于视觉SLAM，最简单的方法是对任意两幅图像都做一遍特征匹配，根据正确匹配的数量确定哪两幅图像存在关联。这种方法朴素但是有效，缺点是任意两幅图像做特征匹配，计算量实在太大，因此不实用。退而求其次的话，随机抽取历史数据进行回环检测，比如在帧中随机抽5帧与当前帧比较。时间效率提高很多，但是抽到回环几率不高，也就是检测效率不高。<br />另外也有如下的一些可能方式：

- **1.基于里程计几何关系的方法：**

1.大概思路是当我们发现当前相机运动到之前的某个位置附近时，检测他们是否存在回环<br />缺点：由于累积误差，很难确定运动到了之前某个位置

- **2.基于外观的方法**

1.仅仅依靠两幅图像的相似性确定回环<br />其核心问题是如何计算图像的相似性。<br />目前视觉SLAM中主要是基于外观的方法，细分下的话，分为传统的方法与基于深度学习的方法，传统的方法有词袋模型与随机蕨法，基于深度学习的方法有CALC算法。
<a name="Xgiiw"></a>
### **10.2.1词袋模型（Bag of words）**
Bag of words模型最初被用在文本分类中，将文档表示成特征矢量。它的基本思想是假定对于一个文本，忽略其词序和语法、句法，仅仅将其看做是一些词汇的集合，而文本中的每个词汇都是独立的。如果文档中猪、马、牛、羊、山谷、土地、拖拉机这样的词汇多些，而银行、大厦、汽车、公园这样的词汇少些，我们就倾向于判断它是一篇描绘乡村的文档，而不是描述城镇的。
> 举例说明
> 文档一：Bob likes to play basketball, Jim likes too.
> 文档二：Bob also likes to play football games.
> 基于这两个文本文档，构造一个词典：Dictionary = {1:”Bob”, 2. “like”, 3. “to”, 4. “play”, 5. “basketball”, 6. “also”, 7. “football”，8. “games”, 9. “Jim”, 10. “too”}。
> 基于上述的词典可以构造出一个两个直方图向量
> 1：[1, 2, 1, 1, 1, 0, 0, 0, 1, 1]
> 2：[1, 1, 1, 1 ,0, 1, 1, 1, 0, 0]
> 这两个向量共包含10个元素, 其中第i个元素表示字典中第i个单词在句子中出现的次数. 因此BoW模型可认为是一种统计直方图 (histogram)

在视觉SLAM当中，每个词换为了特征点的描述子，而一幅图像可以理解为由许多描述子形成的文本，然后使用词袋向量来表示这幅图像。在检测是否发生回环时，通过计算每个关键帧与当前关键帧词袋向量的相似度来看是否发生回环。<br />DBoW系类库是开源的词袋模型库，许多有代表性的VSLAM算法都使用了DBoW做为回环检测算法，如ORB-SLAM系列，VINS-Mono，RTAB-Map。另外，对于激光SLAM，目前也有使用LinK3D描述子的BoW3D词袋库。
<a name="J58cs"></a>
### **10.2.2 随机蕨法（Random ferns）**
开始看到这个名字时，我第一时间想到的是随机森林，但看了算法后，感觉这个和随机森林好像没啥关系。。。<br />先来看下蕨<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686475423405-10647e9e-2103-40d2-814e-5d0a89bbcec4.png#averageHue=%23568f47&clientId=ud4f25423-17a1-4&from=paste&height=385&id=ub6d1896a&originHeight=702&originWidth=542&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=821142&status=done&style=none&taskId=uff33e8c8-fac2-403a-a5dc-b37c43192ba&title=&width=297.6000061035156)<br />哦不，先看下随机蕨算法。。。<br />随机蕨算法是一种图像压缩编码的算法，它首先通过随机算法，对整幅图像随机生成N个采样点，然后对每个采样点，再随机生成一个阈值。然后对每个通道，对比阈值与对应灰度值的大小，生成1或者0的编码，这样每个位置，会生成一个四维的向量。整个图像，会生成一个4N维的向量作为编码，一个随机蕨的生成过程如下：<br />将F=|fi|，i∈{R,G,B,D}定义为一个随机蕨<br />$f_{i}=\left\{\begin{array}{l}
1, I_{i}(x) \geqslant \tau_{i} \\
0, I_{i}(x)<\tau_{i}
\end{array}\right.$<br />式中Ti，的值通过随机函数产生（TR,TG,TB∈[0,255]，TD∈[800,4000])。将随机蕨F中所有的fi按顺序排列，<br />得到一个二进制编码块bF<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686477100719-63fdd905-225a-46c7-ad35-f482ebc4134a.png#averageHue=%23adc7a9&clientId=ud4f25423-17a1-4&from=paste&height=142&id=ue881f540&originHeight=178&originWidth=410&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=83826&status=done&style=none&taskId=u00c3fe19-11ec-4680-9f84-27ff3db74f5&title=&width=328)<br />在计算相似度时，使用基于块的汉明距离(block-wise hamming distance: BlockHD)，即只要一个块里有一位不一样，则整个块的距离为1，反之都一样则为0。两幅图像的 BlockHD越小，说明两者越相似，如果 BlockHD值越大，说明差异性越大。使用随机蕨算法来进行相似度计算的有一些RGBD-SLAM算法，如kinect fusion，elastic fusion等

<a name="S5YEP"></a>
### **10.2.3 基于深度学习的方法： CALC**
CALC是用于回环检测的卷积自动编码器。它该代码分为两个模块。TrainAndTest用于训练和测试模型，DeepLCD是一个用于在线回环检测或图像检索的C ++库。
> 在大型实时SLAM中采用无监督深度神经网络的方法检测回环可以提升检测效果。该方法创建了一个自动编码结构，可以有效的解决边界定位错误。对于一个位置进行拍摄，在不同时间时，由于视角变化、光照、气候、动态目标变化等因素，会导致定位不准。卷积神经网络可以有效地进行基于视觉的分类任务。在场景识别中，将CNN嵌入到系统可以有效的识别出相似图片。但是传统的基于CNN的方法有时会产生低特征提取，查询过慢，需要训练的数据过大等缺点。而CALC是一种轻量级地实时快速深度学习架构，它只需要很少的参数，可以用于SLAM回环检测或任何其他场所识别任务，即使对于资源有限地系统也可以很好地运行。
> 这个模型将高维的原始数据映射到有旋转不变性的低维的描述子空间。在训练之前，图片序列中的每一个图片进行随机投影变换，重新缩放成120×160产生图像对，为了捕捉运动过程中的视角的极端变化。然后随机选择一些图片计算HOG算子，采用固定长度的HOG描述子可以帮助网络更好地学习场景的几何。将训练图片的每一个块的HOG存储到堆栈里，定义为X2，其维度为NxD，其中N是块的大小，D是每一个HOG算子的维度。网络有两个带池化层的卷积层，一个纯卷积层，和三个全连接层，同时用ReLU做卷积层的激活单元。在该体系结构中，将图片进行投影变换，提取HOG描述子的操作仅针对整个训练数据集计算一次，然后将结果写入数据库以用于训练。在训练时，批量大小N设置为1，并且仅使用boxed区域中的层。
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686478526173-d3e45f5c-dfd7-4ec6-a261-c4790894dc73.png#averageHue=%23dededd&clientId=ud4f25423-17a1-4&from=paste&height=242&id=u42d14ec6&originHeight=302&originWidth=1028&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=250754&status=done&style=none&taskId=u7f436cdd-1fae-4fde-9a40-20174138826&title=&width=822.4)


<a name="luzp7"></a>
### **10.2.4 基于缩略图**
PTAM的回环检测方法和random ferns很像。PTAM是在构建关键帧时将每一帧图像缩小并高斯模糊生成一个缩略图，作为整张图像的描述子。在进行图像检索时，通过这个缩略图来计算当前帧和关键帧的相似度。这种方法的主要缺点是当视角发生变化时，结果会发生较大的偏差，鲁棒性不如基于不变量特征的方法。

目前最广泛使用并且最有效的方法是基于外观的一种方法，使用词袋模型进行回环检测
<a name="pgXPM"></a>
## 10.3 词袋模型
词袋是什么，当然是装词的袋子啦。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686573531748-d229ae8f-3b23-4a4e-ba6d-2e5ff32547ba.png#averageHue=%23171f11&clientId=ubabe5fcc-6f45-4&from=paste&height=362&id=udfe237d3&originHeight=453&originWidth=1068&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=300853&status=done&style=none&taskId=u6d612b01-fc00-4d0d-95c5-15bb9393e55&title=&width=854.4)<br />我们一般只关心袋子里物品的种类和数量，对它们的顺序并不关心。现在我们想问，这袋子容量是多少，能装的物品种类是多少，词袋和回环检测有什么关系？别着急，咱们一个个回答。<br />**首先是词袋和回环检测的关系：**词袋模型用于回环检测的基本假设是，如果两幅图像中出现的特征种类和数量差不多，那么两幅图像大概率是同一个地方拍摄的。这就是基于外观的回环检测原理。<br />想象你去商业街购物，在同一家商店里购买东西，出门时袋子里的东西基本是十分相似的，如果去另外一家商店购买，则出门时购物袋里的东西就大概率不一样，当然，也要小心类似于麦当劳和肯德基这种卖同种商品的店铺。因此，我们可以说，词袋相似是回环的必要但不充分条件。<br />**袋子的容量：**取决于每幅图像提取的特征点数量。而如果袋子能装的东西越多，那包含不同内容的袋子的数量就越多。<br />**而袋子能装的物品的种类：**取决于我们词库里单词的数量。单词数量越多，每个袋子的差异性就可以越大，不同场景的区分性越好。比如上文的麦当劳和肯德基，如果我们把物品只区分到汉堡这个层面，那我们就无法区分这个袋子是麦当劳的购物袋还是肯德基的购物袋，但如果我们对汉堡进一步分类，香辣鸡腿堡是肯德基的，麦辣鸡腿堡是麦当劳的，炫辣鸡腿堡是汉堡王的，那么我们就可以区分这个购物袋的来源了。<br />如何表征每个袋子呢，用它里面装的东西就可以了。上图右边两个袋子可以表示为：<br />**美食街的袋子= 2*青苹果+1*螃蟹+1*汉堡套餐**<br />**动物园的袋子= 1*大象+1*蝴蝶+2*猫+1*腕龙 **<br />一看到这两个表示，我们基本可以断定，这肯定不是一个类型的东西，如果看到一个：<br />**神秘的袋子 = 1*大象+1*蝴蝶+2*猫+1*腕龙 + 1*熊猫**<br />我们有很大把握说，这个袋子和动物园的袋子来着同一个商店。<br />那么接下来的问题就是：<br />1.构建袋子中的这些类别<br />2.用这些类别物品，表征这个袋子<br />3.计算两个袋子的相似度<br />在接下来的几个小节中我们来对每个问题逐一进行讲解。

<a name="iwKqr"></a>
### 10.3.1 词的生成
就像生活中常见的做法一样，按照某些特征的不同，我们会把生活中不同物品不断细分为不同类别。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686576641326-11337f61-29fd-4f15-a12f-ed428391a9fd.png#averageHue=%23e2e7f6&clientId=ubabe5fcc-6f45-4&from=paste&height=494&id=ue23ec5ab&originHeight=734&originWidth=617&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=396757&status=done&style=none&taskId=uf00c8853-3541-4770-a0fc-a984a6a7481&title=&width=415.6000061035156)<br />  然后按照这个分类模型，就可以对未知物品，进行分类了。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686576461391-7bcc2686-17a6-446a-8bd9-92a4ea50bfdb.png#averageHue=%23817361&clientId=ubabe5fcc-6f45-4&from=paste&height=268&id=u1cd596bc&originHeight=443&originWidth=617&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=347196&status=done&style=none&taskId=ucd2e69b2-24fe-4735-b0f2-22b98b48349&title=&width=373.6000061035156)<br />词的构建也是如此，我们先采集大量的图片，然后从中提取出大量描述子，然后对这些描述子，使用聚类的方法，构建出一个一个类出来，每个类，就是一个单词。聚类问题在无监督机器学习（Unsupervised ML）中特别常见，用于让机器自行寻找数据中的规律。BoW的字典生成问题也属于其中之一。首先，假设我们对大量的图像提取了特征点，例如有N个。现在，我们想找一个有k个单词的字典，每个单词可以看作局部相邻特征点的集合，这可以用经典的K-means（K均值）算法解决。
> K-means是一个非常简单有效的方法，因此在无监督学习中广为使用，下面对其原理稍做介绍。简单的说，当有N个数据，想要归成k个类，那么用K-means来做主要包括如下步骤：
> 1.随机选取k个中心点：c1，... ，ck。
> 2.对每一个样本，计算它与每个中心点之间的距离，取最小的作为它的归类。
> 3.重新计算每个类的中心点。
> 4.如果每个中心点都变化很小，则算法收敛，退出：否则返回第2步。

但实际训练单词时，我们使用的是和动物学分类中类似的树状形式，进一步说，是K叉树形式。
> 1.在根节点，用K-means把所有样本聚成k类（实际中为保证聚类均匀性会使用K-means++)。这样就得到了第一层。
> 2.对第一层的每个节点，把属于该节点的样本再聚成k类，得到下一层。
> 3.依此类推，最后得到叶子层。叶子层即为所谓的wods。
> 4.计算每个单词的DF权重：idf(i)=logN/n


![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684076464012-3be62e86-088e-4e75-90e2-831b14753b5d.png#averageHue=%23b79b4d&clientId=ucac1b5d4-6cce-4&from=paste&height=335&id=ujQGc&originHeight=419&originWidth=364&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=102881&status=done&style=none&taskId=u1e1356cc-c2db-49da-9107-fe7541e1e10&title=&width=291.2)<br />实际上，最终我们仍在叶子层构建了单词，而树结构中的中间节点仅供快速查找时使用。这样一个k分支、深度为d的树，可以容纳kd个单词。另外，在查找某个给定特征对应的单词时，只需将它与每个中间节点的聚类中心比较(一共d次)，即可找到最后的单词，保证了对数级别的查找效率。而为每一个单词计算一个IDF权重，主要用来给每个词增加区分度，在数据集中出现得越少的单词，其可辨认度就越高。<br />我们把生成的这个词汇树，称为字典。训练词汇树的过程就是构建词典的过程。
<a name="DFTXH"></a>
### 10.3.2 使用字典表征图像
有了字典之后，给定任意特征，只要在字典树中逐层查找，最后都能找到与之对应的单词wj:当字典足够大时。我们可以认为fi和wj来自同一类物体。那么，假设从一幅图像中提取了N个特征，找到这N个特征对应的单词之后，就相当于拥有了该图像在单词列表中的分布，或者直方图。<br />另外我们还需要知道每个词的权重。TF-IDF(Term Frequency-Inverse Document Frequency)，或译频率-逆文档频率，是文本检索中常用的一种加权方式，也用于BoW模型中。T部分的思想是，某单词在一幅图像中经常出现，它的区分度就高。另外，IDF的思想是，某单词在字典中出现的频率越低，分类图像时区分度越高。<br />IDF是在构建字典时就进行的。假设所有特征数量为n，wj的数量为ni，那么该单词的IDF为:<br />$\mathrm{IDF}_{i}=\log \frac{n}{n_{i}}$<br />TF部分则是在获得每幅图像时计算的。假设图像A中单词wi出现了ni次，而该幅图像中共出现的单词次数为n，那么TF为<br />$\mathrm{TF}_{i}=\frac{n_{i}}{n}$<br />于是，wi的权重等于TF乘IDF之积：<br />$\eta_{i}=\mathrm{TF}_{i} \times \mathrm{IDF}_{i}$<br />考虑权重以后，对于某幅图像A，把它的特征点匹配到对应单词之后，组成它的BoW:<br />$A=\left\{\left(w_{1}, \eta_{1}\right),\left(w_{2}, \eta_{2}\right), \ldots,\left(w_{N}, \eta_{N}\right)\right\} \stackrel{\text { def }}{=} \boldsymbol{v}_{A} \\$<br />由于相似的特征可能落到同一个类中，因此实际的VA中会存在大量的零。无论如何，通过词袋我们用单个向量）VA描述了一幅图像A。这个向量vA是一个稀疏的向量，它的非零部分指示了图像A中含有哪些单词，而这些部分的值为TF-IDF的值。
<a name="wWjbP"></a>
### 10.3.3 相似度计算
接下来的问题是：给定vA和vB,如何计算它们的差异呢？这个问题和范数定义的方式一样，存在若干种解决方式，比如常用的L1范数形式：<br />$s\left(\boldsymbol{v}_{A}-\boldsymbol{v}_{B}\right)=2 \sum_{i=1}^{N}\left|\boldsymbol{v}_{A i}\right|+\left|\boldsymbol{v}_{B i}\right|-\left|\boldsymbol{v}_{A i}-\boldsymbol{v}_{B i}\right| \\$
<a name="psXIr"></a>
### 10.3.4 回环检测流程
**数据库查询：**<br />假设通过上节的算法，我们得到了当前图像的词袋（Bow）向量v，现在要在词库中检索最为相似的图像。如何实现这一步，需要先介绍一下DBoW词袋库的特点，以DBoW2词袋库为例：<br />DBoW2库的结构如下，即保存了图像到词汇（根节点）的正向索引，也保存了词汇到图像的逆向索引。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684076686229-f885b581-e66c-47e2-90fe-16057c77486d.png#averageHue=%23e8cf6f&clientId=ucac1b5d4-6cce-4&from=paste&height=309&id=LNAF0&originHeight=485&originWidth=967&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=170699&status=done&style=none&taskId=u3486b2f4-e398-4025-ab4a-7ca3c7a06fe&title=&width=616)<br />每当有新图像进来，DBoW2库就会根据描述子的情况，不断更新这个正向索引和逆索引。<br />在检索最相似的图像时，使用了DBoW2库的逆索引，来加速索引速度，而不是使用暴力匹配的方式：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686582074133-9ac26564-756c-4fb9-b4fc-5d2ee07e3a2b.png#averageHue=%23f0eeec&clientId=ubabe5fcc-6f45-4&from=paste&height=470&id=u07769b3c&originHeight=587&originWidth=577&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=211034&status=done&style=none&taskId=u610a1418-b0f5-45a2-bd96-3aec068107f&title=&width=461.6)<br />上图的含义，就是对图像Q中每个特征点，通过逆索引找到出现的图像，然后给该图像投一票，到最后哪个图像得票多，哪个就是最相似的图像。<br />在查询到相似图像后，其得分计算公式如下：<br />$\eta\left(\mathbf{v}_{t}, \mathbf{v}_{t_{j}}\right)=\frac{s\left(\mathbf{v}_{t}, \mathbf{v}_{t_{j}}\right)}{s\left(\mathbf{v}_{t}, \mathbf{v}_{t-\Delta t}\right)} \\$<br />其中分母是当前关键帧上一帧，与当前的帧的相似度（最优图像得分），用以对查询结果分数做归一化。但需要注意是在转弯等部分场景，上一帧的相似度与当前帧会很小。为防止分母很小导致得分异常得大，需要对分母做判断，跳过那些得分差的帧，往前找直到得到一个分数合理的关键帧。<br />**组匹配：**<br />在进行回环匹配时，会将待匹配的图像按时间划分为独立的组，每一组的相似得分是组内所有图像相似得分之和。这是因为回环帧前后的帧，基本也是相似的，但我们只需要一帧，因此需要进行类似非极大值抑制。<br />$H\left(\mathrm{v}_{t}, V_{T_{i}}\right)=\sum_{j=n_{i}}^{m_{i}} \eta\left(\mathrm{v}_{t}, \mathrm{v}_{t_{j}}\right)$<br />  1、防止连续图像在数据库查询时存在的竞争关系，但是不会考虑同一地点，不同时间的关键帧。<br />	2、防止误匹配<br />**时间一致性判断：**<br />在经过组匹配之后，得到了最佳组匹配，此时需要检验该组匹配的时间一致性。这时做法是，对于当前组匹配vt，与前k帧的最优组匹配，应该有较大的重叠。简单来说，就是在第一次检测到相似帧后，并不会立即认为已发生回环，而是会继续等待接下来k次匹配，在这k次最佳匹配组之间，均有较大的重叠区，这个时候，才会认为真正发生了回环。<br />在当前经过时间一致性校验之后的匹配组，取组内得分最高的图像，作为匹配图像，进入结构一致性校验。<br />**结构一致性判断：**<br />该部分通过特征点匹配，通过RANSAC计算基本矩阵，再通过基本矩阵，对两幅图像的特征点进行投影匹配。这个部分用到了各个特征点在空间中的位置是唯一不变的这一假设。<br />在匹配时，使用了词袋库维护的正向索引（direct image index），加速了特征点的匹配速度。搜索的时候需要注意，同时为了获取足够数量的特征点，不能直接选取words作为匹配索引。同时凸显特征之间的区分度，也不能采用采用较高层数的节点（词袋树形结构），即需要在合适的某一层进行索引匹配。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686582876028-292b5178-97ac-4972-adee-07d7349c65bb.png#averageHue=%231c1d1c&clientId=ubabe5fcc-6f45-4&from=paste&height=191&id=uc68b4655&originHeight=239&originWidth=878&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=31278&status=done&style=none&taskId=ub7efa937-a1e3-4748-a376-6ffa89e3fdd&title=&width=702.4)<br />如果使用基础矩阵投影匹配的点数（内点数）大于12个，认为发生了真正的回环。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686662918479-320d4b10-8430-4509-8b06-f31cff4f329e.png#averageHue=%23a8837b&clientId=ud97a95f5-6d2d-4&from=paste&height=246&id=ueaa9c37c&originHeight=307&originWidth=703&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=261493&status=done&style=none&taskId=uac177455-e573-4aba-8b5d-5aefd9d3001&title=&width=562.4)

<a name="vXefC"></a>
## 10.4 回环检测算法性能评估
对于回环检测算法的评估，常用的标准就是精度—召回曲线，如下<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1686582941537-ae533689-19c5-4a8e-a416-01c917782825.png#averageHue=%23f9f8f7&clientId=ubabe5fcc-6f45-4&from=paste&height=380&id=uc6e47164&originHeight=475&originWidth=905&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=88263&status=done&style=none&taskId=ue2ecacdc-6750-4528-bf7d-cbae5bb8d86&title=&width=724)<br />评价标准<br />	精度：正确回环检测占总检测回环数目的比率<br />	召回：正确回环检测占测试样本总回环的比率<br />$\operatorname{Precision}=\frac{\mathrm{TP}}{\mathrm{TP}+\mathrm{FP}} \\
\text { Recall }=\frac{\mathrm{TP}}{\mathrm{TP}+\mathrm{FN}} \\$<br />一般地，如果某个回环检测算法曲线，被另一个算法的PR曲线包裹在下方，则认为被包裹算法性能弱于包裹其的算法。

<a name="YKIdf"></a>
## 10.5 DBoW系列库
**1）DBoW**<br />DBoW库是一个开源的C++库，用来把图像转换为bag-of-word的表示（词袋模型）。它使用特征提取算法（可选取）提取图像的特征描述子，然后根据一定的策略将这些特征描述子聚类为“单词”，并使用树的形式组织，得到一个vocabulary tree。之后将利用vocabulary tree将图像转换为{单词，权值}的向量表示，通过计算向量间距离的方式计算图像之间的相似性。<br />源码地址：[https://github.com/dorian3d/DBow](https://github.com/dorian3d/DBow)

**2）DBOW2**<br />DBoW2是DBow库的改进版本，DBoW2实现了具有正序和逆序指向索引图片的的图像数据库，可以实现快速查询和特征比较。与以前的DBow库的主要区别是：

- DBoW2类是模板化的，因此它可以与任何类型的描述符一起使用。
- DBoW2可直接使用ORB或BRIEF描述符。
- DBoW2将直接文件添加到图像数据库以进行快速功能比较，由DLoopDetector实现。
- DBoW2不再使用二进制格式。另一方面，它使用OpenCV存储系统来保存词汇表和数据库。这意味着这些文件可以以YAML格式存储为纯文本，更具有兼容性，或以gunzip格式（.gz）压缩以减少磁盘使用。
- 已经重写了一些代码以优化速度。DBoW2的界面已经简化。
- 出于性能原因，DBoW2不支持停止词。

DBoW2需要OpenCV和 Boost::dynamic_bitset类才能使用BRIEF版本。<br />DBoW2和DLoopDetector已经在几个真实数据集上进行了测试，执行了3毫秒，可以将图像的简要特征转换为词袋向量量，在5毫秒可以在数据库中查找图像匹配超过19000张图片。<br />源码地址：[https://github.com/dorian3d/DBoW2](https://github.com/dorian3d/DBoW2)

**3）DBoW3**<br />DBoW3是DBow2库的改进版本，与以前的DBow2库的主要区别是：

- DBoW3只需要OpenCV。DLIB的DBoW2依赖性已被删除。
- DBoW3能够适用二进制和浮点描述符。无需为任何描述符重新实现任何类。
- DBoW3在linux和windows中编译。
- 已经重写了一些代码以优化速度。DBoW3的界面已经简化。
- 使用二进制文件。二进制文件加载/保存比yml快4-5倍。而且，它们可以被压缩。
- 兼容DBoW2的yml文件

源码地址：[https://github.com/rmsalinas/DBow3](https://github.com/rmsalinas/DBow3)

**4）FBOW**<br />FBOW（Fast Bag of Words）是DBow2 / DBow3库的极端优化版本。该库经过高度优化，可以使用AVX，SSE和MMX指令加速Bag of Words创建。在加载词汇表时，fbow比DBOW2快约80倍。在使用具有AVX指令的机器上将图像转换为词袋时，它的速度提高了约6.4倍。<br />源码地址：[https://github.com/rmsalinas/fbow](https://github.com/rmsalinas/fbow)
<a name="KGDsk"></a>
## 本章小结
本章主要介绍了回环检测在SLAM中的意义，即消除里程计的累计误差，获得一张全局一致的地图。并介绍了目前视觉中最为流行的基于外观的回环检测算法，词袋模型。并给出了DBOW3的练习。<br />下一节我们会介绍VSLAM算法最后一个模块，建图模块，也是与需求端连接最为紧密的一个模块。地图作为一个纽带，连接着建图与定位。
<a name="e48hR"></a>
## 本节思考
验证回环检测算法，需要有人工标记回环的数据集。然而人工标记回环是很不方便的，我们会考虑根据标准轨迹计算回环。即，如果轨迹中有两个帧的位姿非常相近，就认为它们是回环。请根据TUM数据集给出的标准轨迹，计算出一个数据集中的回环。这些回环的图像真的相似吗？

<a name="uXGZH"></a>
## 本章练习
[回环检测-词袋模型代码练习](https://github.com/datawhalechina/smoothly-vslam/tree/main/ch10)<br />[DBoW3仓库](https:/github.com/rmsalinas/DBow3)
<a name="xQU7k"></a>
## 参考
0.《视觉SLAM十四讲》<br />[1.回环检测中词袋模型DBoW2的原理分析](https://blog.csdn.net/weixin_37835423/article/details/112890634)<br />[2.ORB-SLAM3知识点(一)：词袋模型BoW](https://zhuanlan.zhihu.com/p/354616831)<br />[3.[ORB-SLAM2] 回环&DBoW视觉词袋](https://zhuanlan.zhihu.com/p/61850005)<br />[4.ORBSLAM2学习（三）：DBoW2理论知识学习](https://blog.csdn.net/lwx309025167/article/details/80524020)<br />[5.视觉slam中的回环检测概述](https://blog.csdn.net/qq_42518956/article/details/107546479#:~:text=1%20%E6%9C%80%E7%AE%80%E5%8D%95%E7%9A%84%E6%96%B9%E6%B3%95%E6%98%AF%E5%AF%B9%E4%BB%BB%E6%84%8F%E4%B8%A4%E5%B9%85%E5%9B%BE%E5%83%8F%E9%83%BD%E5%81%9A%E4%B8%80%E9%81%8D%E7%89%B9%E5%BE%81%E5%8C%B9%E9%85%8D%EF%BC%8C%E6%A0%B9%E6%8D%AE%E6%AD%A3%E7%A1%AE%E5%8C%B9%E9%85%8D%E7%9A%84%E6%95%B0%E9%87%8F%E7%A1%AE%E5%AE%9A%E5%93%AA%E4%B8%A4%E5%B9%85%E5%9B%BE%E5%83%8F%E5%AD%98%E5%9C%A8%E5%85%B3%E8%81%94%E3%80%82%20%E8%BF%99%E7%A7%8D%E6%96%B9%E6%B3%95%E6%9C%B4%E7%B4%A0%E4%BD%86%E6%98%AF%E6%9C%89%E6%95%88%EF%BC%8C%E7%BC%BA%E7%82%B9%E6%98%AF%E4%BB%BB%E6%84%8F%E4%B8%A4%E5%B9%85%E5%9B%BE%E5%83%8F%E5%81%9A%E7%89%B9%E5%BE%81%E5%8C%B9%E9%85%8D%EF%BC%8C%20%E8%AE%A1%E7%AE%97%E9%87%8F%E5%AE%9E%E5%9C%A8%E5%A4%AA%E5%A4%A7%EF%BC%8C%E5%9B%A0%E6%AD%A4%E4%B8%8D%E5%AE%9E%E7%94%A8%20%E3%80%82,2%20%E9%9A%8F%E6%9C%BA%E6%8A%BD%E5%8F%96%E5%8E%86%E5%8F%B2%E6%95%B0%E6%8D%AE%E8%BF%9B%E8%A1%8C%E5%9B%9E%E7%8E%AF%E6%A3%80%E6%B5%8B%EF%BC%8C%E6%AF%94%E5%A6%82%E5%9C%A8nn%E5%B8%A7%E4%B8%AD%E9%9A%8F%E6%9C%BA%E6%8A%BD5%E5%B8%A7%E4%B8%8E%E5%BD%93%E5%89%8D%E5%B8%A7%E6%AF%94%E8%BE%83%E3%80%82%20%E6%97%B6%E9%97%B4%E6%95%88%E7%8E%87%E6%8F%90%E9%AB%98%E5%BE%88%E5%A4%9A%EF%BC%8C%E4%BD%86%E6%98%AF%E6%8A%BD%E5%88%B0%E5%9B%9E%E7%8E%AF%E5%87%A0%E7%8E%87%E4%B8%8D%E9%AB%98%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%20%E6%A3%80%E6%B5%8B%E6%95%88%E7%8E%87%E4%B8%8D%E9%AB%98%20%E3%80%82)<br />[6.实时词袋模型BoW3D | 用于3D激光雷达SLAM回环检测（开源）](https://zhuanlan.zhihu.com/p/598151357)<br />[7.浅谈回环检测中的词袋模型（bag of words）](https://blog.csdn.net/qq_24893115/article/details/52629248)<br />[8.猫科の完全指南①--猫科动物概论](https://zhuanlan.zhihu.com/p/33842443)
