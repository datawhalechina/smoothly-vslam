# **11.终识庐山真面目：建图**

学到很多东西的诀窍，就是一下子不要学很多。——洛克

---

> **本章主要内容** \
> 1.地图的类型 \
> 2.稠密重建 \
> 3.点云地图 \
> 4.TSDF地图 \
> 5.实时三维重建

建图的目的是获得满足任务需求的地图。地图的种类十分多，不同类型任务需要不同类型的地图。一般来说，地图可以分为度量地图与拓扑地图，而对于度量地图，又可以继续细分为**栅格地图、特征地图、点云地图等。**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685861896661-958d6d8f-b02b-4a3b-9efc-d40fa33b68b5.png#averageHue=%23988c7a&clientId=u4d9282ea-6449-4&from=paste&height=374&id=uf8b824d2&originHeight=588&originWidth=1012&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=545664&status=done&style=none&taskId=u940a1d5b-96a6-47a2-b439-e9ffce62079&title=&width=643)<br />不同类型地图满足不同任务需求：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685805136691-d1cc0485-8ba2-46bd-8cf8-9a946939a9d4.png#averageHue=%23d0cac0&clientId=uc14277d2-25da-4&from=paste&height=384&id=qwQug&originHeight=681&originWidth=1031&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=383126&status=done&style=none&taskId=ub0628d42-67de-4bc7-abd8-0ac199e702f&title=&width=581)<br />总结来说，地图分类如下：<br />![](https://cdn.nlark.com/yuque/0/2023/jpeg/1782698/1685863282746-e859afe3-c9d7-4777-b9fd-f15c34e61aa7.jpeg)
<a name="LVu44"></a>
## 11.1 度量地图（Metric Map）与拓扑地图（Topological Map）
**度量地图**强调精确地表示地图中物体的空间位置，通常用 稀疏（Sparse） 和 稠密（Dense） 对它们进行分类。

- 稀疏地图：进行了一定程度的抽象，并不需要表达所有的物体。例如，选择场景中一部分具有代表意义的东西，称之为路标(Landmark)，那么一张稀疏地图就是由路标组成的地图，而不是路标的部分就可以忽略。**定位时用稀疏路标地图就足够了**。
- 稠密地图：稠密地图着重于建模所有看到的东西。通常按照某种分辨率，由许多小块组成。二维度量地图是许多小格子（Grid，二维栅格地图），三维中则是许多小方块（Voxel，体素地图）。一个小块一般含有占据、空闲、未知三种状态，以表达该格内是否有物体。当查询某个空间位置时，地图能够给出该位置是否可以通过的信息。这样的地图可以用于各种导航算法，如A*、D*等。

**拓扑地图**则更强调地图元素之间的关系。拓扑地图是一个图（Graph），由节点和边组成，只考虑节点间的连通性。它放松了地图对精确位置的需要，去掉地图的细节问题，是一种更为紧凑的表达方式。<br />这里说一下个人对导航和避障的理解，二者是全局路径规划与局部路径规划的关系。全局路径规划需要知道全局静态障碍物的分布，比如墙体，台阶，桌椅这种不会移动的物体的位置，然后在两个点之间基于全局可通行区域搜索出一条路径出来，所以导航注重静态障碍物。避障需要实时感知周围障碍物信息，对突然出现的动态障碍物及时更改路线，因此避障注重动态障碍物，强调实时感知。<br />对于重建，个人感觉更偏向配准，及如何把点云拼起来，得到一个更还原的三维模型。在这个意义上说，SLAM强调的是通过估计位姿，并通过估计的位姿还原环境，而其地图，更强调拿来做定位。其强调定位准确，而不是得到一个更逼真的三维模型，这是和三维重建的区别所在。所以大多SLAM算法生成的，都是稀疏路标点地图。但通过SLAM估计的位姿及传感器数据，也能获得用于导航的稠密地图。<br />基于特征点的VSLAM算法建立的地图为稀疏路标点地图，可直接保存描述子信息，使用基于词袋模型的回环检测算法来进行全局定位，这里就不在赘述，接下来着重介绍生成稠密地图。
<a name="jxKat"></a>
## 11.2 稠密建图
由于相机，被认为是只有角度的传感器(Bearing only)。单幅图像中的像素，只能提供物体与相机成像平面的角度及物体采集到的亮度，而无法提供物体的距离(Range)。而在稠密重建中，我们需要知道每一个像素点（或大部分像素点）的距离，对此大致上有如下解决方案：

- 1.使用单目相机，估计相机运动，并且三角化计算像素的距离。
- 2.使用双目相机，利用左右目的视差计算像素的距离（多目原理相同）。
- 3.使用RGB-D相机直接获得像素距离。

前两种方式称为立体视觉(Stereo Vision)，其中移动单目相机的又称为移动视角的立体视觉(Moving View Stereo,MVS)。相比于RGB-D直接测量的深度，使用单目和双目的方式对深度获取往往是“费力不讨好”的一计算量巨大，最后得到一些不怎么可靠的深度估计。当然，RGB-D也有一些量程、应用范围和光照的限制，不过相比于单目和双目的结果，**使用RGB-D进行稠密重建往往是更常见的选择**。**而单目、双目的好处是，在目前RGB-D还无法被很好地应用的室外、大场景场合中，仍能通过立体视觉估计深度信息。**<br />我们从最简单的情况讲起：假设通过SLAM，我们获得了每幅图像对应的相机轨迹，通过这些图像的位姿，估计某幅图像每个像素的深度，这就是最简单的建图问题。<br />首先，请回忆在特征点部分我们是如何完成该过程的：

- 1.对图像提取特征，并根据描述子计算特征之间的匹配。换言之，通过特征，我们对某一个空间点进行了跟踪，知道了它在各个图像之间的位置。
- 2.由于无法仅用一幅图像确定特征点的位置，所以必须通过不同视角下的观测估计它的深度，原理即前面讲过的三角测量。

但在稠密深度图估计中，不同之处在于，**我们无法把每个像素都当作特征点计算描述子**。因此，稠密深度估计问题中，匹配就成为很重要的一环：如何确定第一幅图的某像素出现在其他图里的位置呢？这需要用到极线搜索和块匹配技术2)。当我们知道了某个像素在各个图中的位置，就能像特征点那样，利用三角测量法确定它的深度。不过不同的是，在这里要使用很多次三角测量法让深度估计收敛，而不仅使用一次。我们希望深度估计能够随着测量的增加从一个非常不确定的量，逐渐收敛到一个稳定值。这就是深度滤波器技术。所以，下面的内容将主要围绕这个主题展开。
<a name="kUuBw"></a>
### 11.2.1 立体视觉稠密重建
<a name="OjfJg"></a>
#### 11.2.1.1 立体视觉稠密重建
接上一节，以下两种称为立体视觉(Stereo Vision)，其中移动单目相机的又称为移动视角的立体视觉(Moving View Stereo,MVS)。

- 1.使用单目相机，估计相机运动，并且三角化计算像素的距离。
- 2.使用双目相机，利用左右目的视差计算像素的距离（多目原理相同）。

在单目情况下，通过不同时刻相机位姿进行像素深度估计，而对于双目，则通过左右目图像进行像素深度估计。需要使用极线搜索与块匹配来获取深度分布，然后使用深度滤波器更新深度直到收敛。<br />第一步，极线计算。由相机之间位姿，计算出该图像上每个像素，在另外一幅图像上的极线。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685867856657-bdc92fd7-9917-4e16-a913-57e2c3240bf5.png#averageHue=%23fdfdfd&clientId=u4d9282ea-6449-4&from=paste&height=295&id=u12e1ee52&originHeight=495&originWidth=666&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=66279&status=done&style=none&taskId=u543a70e5-cf17-4a70-a282-75cde2b8b9c&title=&width=396.79998779296875)<br />第二步，沿着极线，计算该像素周围纹理，与待匹配点周围纹理的匹配程度。这一步称为块匹配算法。假设我们要计算下图中绿框块与白框块的相似度，可以这么做：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685868680772-d0ec90c1-62fa-4681-b71c-cd93a1ae5c37.png#averageHue=%236b9954&clientId=u4d9282ea-6449-4&from=paste&height=214&id=u3dc8ebdb&originHeight=361&originWidth=889&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=560627&status=done&style=none&taskId=u83740d90-22c3-4a4b-aaf5-eb61b1f2daf&title=&width=527.2000122070312)<br />先转为灰度图，然后计算每个像素的灰度差的绝对值之和：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685868631759-2b33e9c4-99ec-4323-9b9f-07f3a3aa03c7.png#averageHue=%23f6f6f6&clientId=u4d9282ea-6449-4&from=paste&height=299&id=uf4440814&originHeight=374&originWidth=996&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=98996&status=done&style=none&taskId=uaec095a4-afee-440e-a756-d325f0d41c5&title=&width=796.8)<br />当然，这里的指标有很多，也可以计算距离平方和（Sum of Squared Distance: SSD）或者归一化相关（Normalized Cross Correlation: NCC)。为了去除光照影响，这一步也可以先减去每个小块的均值，称为去均值的SSD，去均值的NCC等。为了将相机视角对图像的影响考虑进去，也会对匹配的图像块加入仿射变换后，再计算相似度分数。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685867954485-18ab1f5c-d0a0-4ebe-a28b-2cd828044bb7.png#averageHue=%23fcfcfb&clientId=u4d9282ea-6449-4&from=paste&height=319&id=u249d9c06&originHeight=399&originWidth=499&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=74880&status=done&style=none&taskId=u10ec6667-d2d1-47d4-ab7c-da2c043d84b&title=&width=399.2)<br />第三步，获得最佳匹配点，计算不确定度。获得极线上每个像素点相似度分数可能如下，取最高得分，作为最佳匹配点。这时需要考虑当前匹配点_P__2_下，左右如果误匹配一个像素点，会造成多大的误差，即下图中的P与P'之差，作为方差σobs，而μobs则是这次估计的深度P。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685868024740-be8a0b33-c8cc-4d73-8b1a-ba7007cf6bd9.png#averageHue=%23fdfdfd&clientId=u4d9282ea-6449-4&from=paste&height=330&id=u65bebc7b&originHeight=412&originWidth=590&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=69355&status=done&style=none&taskId=ubbd963a6-a268-469c-a6a8-fbe455eef5e&title=&width=472)<br />假设深度分布服从高斯分布，则该点深度分布服从：<br />$P(d_{obs}) = N(\mu _{obs},\delta ^{2}_{obs})$<br />第四步，更新深度分布。在_P__2_与其他图像的产生了匹配后，按照以上1到3步获得了新的深度分布，则将两个高斯分布进行相乘，得到融合后的深度分布。<br />$\mu_{f u s e}=\frac{\sigma_{o b s}^{2} \mu+\sigma^{2} \mu_{o b s}}{\sigma^{2}+\sigma_{o b s}^{2}}, \quad \sigma_{f u s e}^{2}=\frac{\sigma^{2} \sigma_{o b s}^{2}}{\sigma^{2}+\sigma_{o b s}^{2}}$<br />其中σ，而μ是原深度方差及均值。这样不断更新，直到深度不确定度小于阈值，停止更新。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685871375493-05f687ba-40be-4a1f-a937-58a009e5061e.png#averageHue=%23646464&clientId=u4d9282ea-6449-4&from=paste&height=428&id=ubc21395d&originHeight=838&originWidth=1036&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=673146&status=done&style=none&taskId=udb00e5a1-d568-4eea-88c7-0f06349f07a&title=&width=529)<br />迭代次数越多，深度越稳定。上图是迭代10次后与30次后的效果。
<a name="uXqb2"></a>
#### 11.2.1.2 稠密重建影响因数分析

- **像素梯度**

对深度图像进行观察，会发现一件明显的事实。块匹配的正确与否依赖于图像块是否具有区分度，即像素梯度变化分布情况。纯色的色块没有灰度梯度，无法得到有效的匹配。产生的深度信息多半是不正确的。如图中打印机表面出现了明显不该有的条纹状深度估计（而根据我们的直观想象，打印机表面肯定是光滑的）。相反，像素梯度比较明显的地方，我们得到的深度信息也相对准确，例如桌面上的杂志、电话等具有明显纹理的物体。这表明了立体视觉中一个非常常见的问题：对物体纹理的依赖性。该问题在双目视觉中也极其常见，体现了立体视觉的重建质量十分依赖于环境纹理。<br />从某种角度来说，该问题是无法在现有的算法流程上加以改进并解决的。如果我们依然只关心某个像素周围的邻域（小块）的话。进一步讨论像素梯度问题，还会发现像素梯度和极线之间的联系。我们举两种比较极端的情况，一个是像素灰度沿着极线变化（左），一个是像素灰度垂直于极线变化（右）。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685872082581-b113abd2-c621-47ed-bdbe-176e5f30a538.png#averageHue=%23cacaca&clientId=u4d9282ea-6449-4&from=paste&height=364&id=uf0eeeed1&originHeight=757&originWidth=955&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=282433&status=done&style=none&taskId=u3632b015-6d48-4fb5-acc0-02d0efc17f8&title=&width=459)<br />像素梯度平行于极线方向（像素灰度沿着极线变化，左图），我们能够精确地、确定匹配度最高点出现在何处。而在垂直的情况，即使小块有明显梯度，当我们沿着极线做块匹配时，会发现匹配程度都是一样的，因此得不到有效的匹配。<br />而实际中，梯度与极线的情况很可能介于二者之间：既不是完全垂直也不是完全平行。这时，我们说，当像素梯度与极线夹角较大时，极线匹配的不确定性大：而当夹角较小时，匹配的不确定性变小。而在演示程序中，我们把这些情况都当成一个像素的误差，实际是不够精细的。考虑到极线与像素梯度的关系，应该使用更精确的不确定性模型。

- **深度分布假设**

从另一个角度看，我们不妨问：把像素深度假设成高斯分布是否合适呢？这里关系到一个参<br />数化的问题。<br />逆深度(Inverse depth)是近年来SLAM研究中出现的一种广泛使用的参数化技巧。简单来说，就是深度并不太服从一个高斯分布：

- 1.我们实际想表达的是：这个场景深度大概是5~10米，可能有一些更远的点，但近处肯定不会小于相机焦距(或认为深度不会小于0)。这个分布并不是像高斯分布那样，形成一个对称的形状。它的尾部可能稍长，而负数区域则为零。
- 2.在一些室外应用中，可能存在距离非常远，乃至无穷远处的点。我们的初始值中难以涵盖这些点，并且用高斯分布描述它们会有一些数值计算上的困难。

逆深度应运而生。人们在仿真中发现，假设深度的倒数，也就是逆深度为高斯分布。是比较有效的。随后，在实际应用中，逆深度也具有更好的数值稳定性，从而逐渐成为一种通用的技巧。

- **平滑及外点假设**

1.现在各像素完全是独立计算的，可能存在这个像素深度很小，边上一个又很大的情况。我们可以假设深度图中相邻的深度变化不会太大，从而给深度估计加上了空间正则项。这种做法会使得到的深度图更加平滑。<br />2.我们没有显式地处理外点(Outlier)的情况。事实上，由于遮挡、光照、运动模糊等各种因素的影响，不可能对每个像素都保持成功匹配。而演示程序的做法中，只要NCC大于一定值，就认为出现了成功的匹配，没有考虑到错误匹配的情况。<br />3.处理错误匹配亦有若干种方式。例如均匀一高斯混合分布下的深度滤波器，显式地将内点与外点进行区别并进行概率建模，能够较好地处理外点数据。<br />从上面的讨论可以看出，存在许多可能的改进方案。如果我们细致地改进每一步的做法，最后是有希望得到一·个良好的稠密建图的方案的。然而，正如我们所讨论的，有一些问题存在理论上的困难，例如对环境纹理的依赖，又如像素梯度与极线方向的关联（以及平行的情况）。这些问题很难通过调整代码实现来解决。所以，直到目前为止，虽然双目和移动单目相机能够建立稠密的地图，但是我们通常认为它们过于依赖环境纹理和光照，不够可靠。
<a name="yp0Np"></a>
### 11.2.2 RGBD稠密重建
除了使用单目和双目相机进行稠密重建，在适用范围内，RGB-D相机是一种更好的选择。对于深度估计问题，在RGB-D相机中可以完全通过传感器中硬件的测量得到，无须消耗大量的计算资源来估计。并且，RGB-D的结构光或飞时原理，保证了深度数据对纹理的无关性。即使面对纯色的物体，只要它能够反射光，我们就能测量到它的深度。这也是RGB-D传感器的一大优势。<br />利用RGBD进行稠密建图是相对容易的。不过，根据地图形式不同，也存在着若干种不同的主流建图方式。最直观、最简单的方法就是根据估算的相机位姿，将RGB-D数据转化为点云，然后进行拼接，最后得到一个由离散的点组成的点云地图(Point Cloud Map)。在此基础上，如果我们对外观有进一步的要求，希望估计物体的表面，则可以使用三角网格(Mesh)、面片(Surfel)进行建图。另外，如果希望知道地图的障碍物信息并在地图上导航，也可通过体素(Voxl)建立占据网格地图(Occupancy Map)。<br />使用RGBD相机生成稠密地图时，需要注意如下一些点：

- 1.在生成每帧点云时，去掉深度值无效的点。这主要是考虑到RGBD相机会有一个有效量程，超过量程之后的深度值会有较大误差或返回一个零。
- 2.利用统计滤波器方法去除孤立点。该滤波器统计每个点与距离它最近的N个点的距离值的分布，去除距离均值过大的点。这样，就保留了那些“粘在一起”的点，去掉了孤立的噪声点。
- 3.利用体素网络滤波器进行降采样。由于多个视角存在视野重叠，在重叠区域会存在大量的位置十分相近的点。这会占用许多内存空间。体素滤波保证了在某个一定大小的立方体（或称体素）内仅有一个点，相当于对三维空间进行了降采样，从而节省了很多存储空间。

<a name="q7HXO"></a>
## 11.3 点云地图
经过以上稠密重建，我们可以获得如下稠密的点云地图。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685874280581-02e42c09-8c95-4b27-9cf2-fe58523b4dd2.png#averageHue=%232e2d2d&clientId=u4d9282ea-6449-4&from=paste&height=295&id=u8eeb5fc3&originHeight=369&originWidth=564&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=239521&status=done&style=none&taskId=ud28be325-abe3-4206-b12e-512e29abc3d&title=&width=451.2)<br />点云地图提供了比较基本的可视化地图，让我们能够大致了解环境的样子。它以三维方式存储，使我们能够快速地浏览场景的各个角落，乃至在场景中进行漫游。不过，使用点云表达地图仍然是十分初级的，我们不妨按照之前提的对地图的需求，看看点云地图是否能满足这些需求。

- 1.定位需求：取决于前端视觉里程计的处理方式。如果是基于特征点的视觉里程计，由于点云中没有存储特征点信息，则无法用于基于特征点的定位方法。如果前端是点云的ICP,那么可以考虑将局部点云对全局点云进行ICP以估计位姿。然而，这要求全局点云具有较好的精度。我们处理点云的方式并没有对点云本身进行优化，所以是不够的。
- 2.导航与避障的需求：无法直接用于导航和避障。纯粹的点云无法表示“是否有障碍物”的信息，我们也无法在点云中做“任意空间点是否被占据”这样的查询，而这是导航和避障的基本需要。不过，可以在点云基础上进行加工，得到更适合导航与避障的地图形式。
- 3.可视化和交互：具有基本的可视化与交互能力。我们能够看到场景的外观，也能在场景里漫游。从可视化角度来说，由于点云只含有离散的点，没有物体表面信息（例如法线），所以不太符合人们的可视化习惯。例如，从正面和背面看点云地图的物体是一样的，而且还能透过物体看到它背后的东西：这些都不太符合我们日常的经验。

综上所述，我们说点云地图是“基础的”或“初级的”，是指它更接近传感器读取的原始数据。它具有一些基本的功能，但通常用于调试和基本的显示，不便直接用于应用程序。

<a name="KOwgv"></a>
### 11.3.1 由点云地图构建网格地图
如果我们希望地图有更高级的功能，那么点云地图是一个不错的出发点。例如，针对导航功能，可以从点云出发，构建占据网格(Occupancy Grid)地图，以供导航算法查询某点是否可以通过。再如，SfM中常用的泊松重建方法，就能通过基本的点云重建物体网格地图，得到物体的表面信息。除了泊松重建，Surfel也是一种表达物体表面的方式，以面元作为地图的基本单位，能够建立漂亮的可视化地图。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685874470287-6d5e1bf6-6f28-488d-ab3a-cec4a9a00f86.png#averageHue=%23bababa&clientId=u4d9282ea-6449-4&from=paste&height=279&id=udfa49d8a&originHeight=349&originWidth=1090&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=240343&status=done&style=none&taskId=ubc3374b9-6f6b-48a0-8d96-5519658ad61&title=&width=872)<br />点云重建网格：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685874540934-33191df8-3735-4221-ba87-e4b674063148.png#averageHue=%232e2e2e&clientId=u4d9282ea-6449-4&from=paste&height=311&id=u7417c3b3&originHeight=523&originWidth=898&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=480288&status=done&style=none&taskId=u6ee4d869-11f2-444a-98f1-6931aa2bed9&title=&width=533.4000244140625)
<a name="FKp60"></a>
### 11.3.2 由点云地图构建八叉树地图
在一个立方体过中心的位置，在上，前，左方向各沿平行于两面的方向切一刀，可以将一个立方体分割为八个小块，然后不断对每个小方块进行分割，每个小块会继续分为八个小块。这相当于每个方块节点展开都会有八个子节点，因此称之为八叉树。每个小立方体是空间的一个元素，称为体素。体素可以表示占据状态，如果该方格内有点云时，该网站占据状态为1，没有占据的话，占据状态为0，还有一种是未知的占据状态。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685881628592-f857a10f-37ff-4965-b92f-e386b1be271e.png#averageHue=%23e9e9e9&clientId=u4d9282ea-6449-4&from=paste&height=222&id=u80e2ec9c&originHeight=455&originWidth=1034&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=245718&status=done&style=none&taskId=u7d75ddf1-2f7e-4abc-b82e-e6507418762&title=&width=505)<br />这里展开的层数可以设定，层数越多，每个叶子层表示的体素越小，地图“分辨”率越高。看起来会越精细，反之则会越来越粗糙。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685884053753-da379929-3720-4490-aaf6-dc2ab06ad53f.png#averageHue=%23fbfefa&clientId=u4d9282ea-6449-4&from=paste&height=230&id=uf7660aab&originHeight=288&originWidth=683&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=182816&status=done&style=none&taskId=u2d4f7ac0-857b-4604-a714-705d298a8fa&title=&width=546.4)<br />相比于直接使用点云地图，八叉树地图有如下好处：

- 1.减少存储空间。点云文件通常都比较大，相比之下，八叉树存储同样的点云信息则更高效。因为在某一大块立方体内，每个小块都是同一状态时。该大块节点就可以不展开。
- 2.点云地图不能表示直接表示某个空间是否被占据，不能用于导航，而**八叉树地图可以表示某个空间状态，可直接用于导航。**
- 3.可以方便地更新空间状态。对于八叉树，我们会选择用概率形式表达某节点是否被占据的事情。例如，用一个浮点数x∈0,1来表达。这个x一开始取0.5。如果不断观测到它被占据，那么让这个值不断增加；反之，如果不断观测到它是空白，那就让它不断减小即可。通过这种方式，我们动态地建模了地图中的障碍物信息。

每一个体素的概率，一般使用对数几率来表示：<br />$x=\operatorname{logit}^{-1}(y)=\frac{\exp (y)}{\exp (y)+1}$<br />其中y为当前体素的观测状态，如果该体素不断被观测到为“占据”状态时，则给y加上一个数，反之，则给y减去一个数。当查询概率时，则使用上式，将y转为概率即可。使用公式来说，就是t+1时刻的，对于节点n来说，其概率为开始到t-1时刻的状态，加上t时刻的观测。<br />$L\left(n \mid z_{1: t+1}\right)=L\left(n \mid z_{1: t-1}\right)+L\left(n \mid z_{t}\right)$<br />如果要写为概率形式，则比较复杂，这里放在附录中，感兴趣的小伙伴可以自行查看。
<a name="DuaVo"></a>
## 11.4 TSDF地图
TSDF是Truncated Signed Distance Function的缩写，与八叉树相似，TSDF地图也是一种网格式（或者说方块式）的地图，先选定要建模的三维空间，按照一定分辨率将这个空间分成许多小块，存储每个小块内部的信息。不同的是，TSDF地图整个存储在显存而不是内存中。利用GPU的并行特性，我们可以并行地对每个体素进行计算和更新，而不像CPU遍历内存区域那样不得不串行。<br />每个TSDF体素内存储了该小块与距其最近的物体表面的距离。如果小块在该物体表面的前方，则它有一个正值：反之，如果该小块位于表面之后，那么就为负值。TSDF为0的地方就是表面本身或者，由于数值误差的存在，TSDF由负号变成正号的地方就是表面本身。如下图，我们看到一个类似人脸的表面出现在TSDF改变符号的地方。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685884784553-13716e6b-1409-4a02-9c19-48f9e3cb927d.png#averageHue=%23a2a39d&clientId=u4d9282ea-6449-4&from=paste&height=267&id=GnqQR&originHeight=334&originWidth=391&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=123724&status=done&style=none&taskId=u38549b4d-ba92-4bbd-abf5-961e16bd710&title=&width=312.8)<br />TSDF也有“定位”与“建图”两个问题，与SLAM非常相似，不过具体的形式与本书前面几讲介绍的稍有不同。在这里，定位问题主要指如何把当前的RGB-D图像与GPU中的TSDF地图进行比较，估计相机位姿。而建图问题就是如何根据估计的相机位姿，对TSDF地图进行更新。传统做法中，我们还会对RGB-D图像进行一次双边贝叶斯滤波，以去除深度图中的噪声。<br />TSDF的定位类似于前面介绍的ICP,不过由于GPU的并行化，我们可以对整张深度图和TSDF地图进行ICP计算，而不必像传统视觉里程计那样必须先计算特征点。同时，由于TSDF没有颜色信息，意味着我们**可以只使用深度图，不使用彩色图就完成位姿估计**，这在一定程度上摆脱了视觉里程计算法对光照和纹理的依赖性，使得RGB-D重建更加稳健。另外，建图部分也是一种并行地对TSDF中的数值进行更新的过程，使得所估计的表面更加平滑可靠。由于我们并不过多介绍GPU相关的内容，所以具体的方法就不展开了，请感兴趣的读者参阅相关文献。
<a name="GOgtP"></a>
## 11.5 实时三维重建
实时三维重建是一个与SLAM非常相似但又有稍许不同的研究方向。在前面的地图模型中，以定位为主体。地图的拼接是作为后续加工步骤放在SLAM框架中的。这种框架成为主流的原因是定位算法可以满足实时性的需求，而地图的加工可以在关键帧处进行处理，无须实时响应。定位通常是轻量级的，特别是当使用稀疏特征或稀疏直接法时：相应地，地图的表达与存储则是重量级的。它们的规模和计算需求较大，不利于实时处理。特别是稠密地图，往往只能在关键帧层面进行计算。<br />但是，现有做法中并没有对稠密地图进行优化。例如，当两幅图像都观察到同一把椅子时，我们只根据两幅图像的位姿把两处的点云进行叠加，生成了地图。由于位姿估计通常是带有误差的，这种直接拼接往往不够准确，比如同一把椅子的点云无法完美地叠加在一起。这时，地图中会出现这把椅子的两个重影一一这种现象被形象地称为“鬼影”。<br />这种现象显然不是我们想要的，我们希望重建结果是光滑的、完整的，是符合我们对地图的认识的。在这种思想下，出现了一种以“建图”为主体，定位居于次要地位的做法，也就是本节要介绍的实时三维重建。由于三维重建把重建准确地图作为主要目标，所以需要利用GPU进行加速，甚至需要非常高级的GPU或多个GPU进行并行加速，通常需要较重的计算设备。与之相反，**SLAM是朝轻量级、小型化方向发展的**，有些方案甚至放弃了建图和回环检测部分，只保留了视觉里程计。而**实时重建则正在朝大规模、大型动态场景的重建方向发展**。<br />自从RGB-D传感器出现以来，利用RGB-D图像进行实时重建成为了一个重要的发展方向，陆续产生了Kinect Fusion、Dynamic Fusion、Elastic Fusion、Fusion4D、Volumn Deform等成果。其中，KinectFusion完成了基本的模型重建，但仅限于小型场景；后续的工作则是将它向大型的、运动的，甚至变形场景下拓展。由于篇幅有限，这部分算法在这里就不做展开介绍了，感兴趣的小伙伴可以自行查找了解。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685883833506-009a569d-ecb6-4f5a-ae25-cf7f5c69a9ff.png#averageHue=%235c82b4&clientId=u4d9282ea-6449-4&from=paste&height=270&id=u11c2196b&originHeight=546&originWidth=1056&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=1220820&status=done&style=none&taskId=u3c2cb377-39ae-40ab-ba67-b744288ca7f&title=&width=522)<br />Kinetic Fusion建图效果<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685883909451-a0033db6-7868-4529-93fc-1f6c3ecba7eb.png#averageHue=%23949494&clientId=u4d9282ea-6449-4&from=paste&height=570&id=udff770ee&originHeight=970&originWidth=786&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=472026&status=done&style=none&taskId=u7f5dc6b9-bf1b-4447-ab48-aa24ae45532&title=&width=461.79998779296875)
<a name="sVBjs"></a>
## 本章小结
本节介绍SLAM最后一个模块，建图部分。主要介绍了一些常见的地图类型，着重介绍了稠密地图的建图形式与立体视觉如何进行稠密重建。也比较了与SLAM十分类似的一个分支-实时三维重建的特点。

<a name="BDCNs"></a>
## 本章思考
1.是否可以只关注梯度显著的区域，进行立体视觉的半稠密重建。<br />2.如何在八叉树地图中进行导航或路径规划。<br />3.了解均匀-高斯混合滤波器的原理与实现。
<a name="ikDhR"></a>
## 附录
<a name="ys5mR"></a>
### 1.八叉树体素网格概率更新公式:
$P\left(n \mid z_{1: T}\right)=\left[1+\frac{1-P\left(n \mid z_{T}\right)}{P\left(n \mid z_{T}\right)} \frac{1-P\left(n \mid z_{1: T-1}\right)}{P\left(n \mid z_{1: T-1}\right)} \frac{P(n)}{1-P(n)}\right]^{-1}$

<a name="XgdFr"></a>
## 本节代码练习
[smoothly-vslam:单目稠密重建](https://github.com/datawhalechina/smoothly-vslam/tree/main/ch11)
<a name="nc63i"></a>
## 参考
1.《视觉SLAM十四讲》<br />[2.占据栅格地图构建（Occupancy Grid Map）](https://www.guyuehome.com/14968)<br />[3.基于图像的三维模型重建——稠密点云重建](https://zhuanlan.zhihu.com/p/131590433)<br />[4.SLAM14讲学习笔记（九）单目稠密重建、深度滤波器详解与补充](https://blog.csdn.net/zkk9527/article/details/89095967#:~:text=%E6%9E%81%E7%BA%BF%E6%90%9C%E7%B4%A2%E5%92%8C%E5%9D%97%E5%8C%B9%E9%85%8D%E7%9A%84%E7%9B%AE%E7%9A%84%E5%B0%B1%E6%98%AF%E6%89%BE%E5%88%B0%E7%89%B9%E5%AE%9A%E7%9A%84%E7%82%B9%E5%9C%A8%E4%B8%8B%E4%B8%80%E5%B8%A7%E4%B8%AD%E7%9A%84%E4%BD%8D%E7%BD%AE%E3%80%82%20%E8%BF%99%E4%B8%AA%E7%9B%AE%E7%9A%84%E5%B0%B1%E6%98%AF%E8%BF%BD%E8%B8%AA%E5%83%8F%E7%B4%A0%E7%82%B9%EF%BC%8C%E5%92%8C%E5%85%89%E6%B5%81%E6%B3%95%E5%92%8C%E7%9B%B4%E6%8E%A5%E6%B3%95%E7%9A%84%E7%9B%AE%E7%9A%84%E6%98%AF%E4%B8%80%E6%A0%B7%E7%9A%84%E3%80%82%20%E8%BF%99%E7%A7%8D%E6%96%B9%E5%BC%8F%E5%B0%B1%E6%98%AF%E5%9F%BA%E4%BA%8E%E6%95%B0%E5%AD%A6%E7%9A%84%E5%87%A0%E4%BD%95%E6%96%B9%E5%BC%8F%EF%BC%9A%20O1p%E8%BF%99%E6%9D%A1%E5%B0%84%E7%BA%BF%E5%9C%A8O2%E5%AF%B9%E5%BA%94%E7%9A%84%E9%82%A3%E4%B8%AA%E6%88%90%E5%83%8F%E5%B9%B3%E9%9D%A2%E4%BC%9A%E5%BD%A2%E6%88%90%E4%B8%80%E4%B8%AA%E6%8A%95%E5%BD%B1%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E6%89%80%E8%B0%93%E2%80%9C%E6%9E%81%E7%BA%BF%E2%80%9D%EF%BC%8C%E7%BA%A2%E8%89%B2%E7%9A%84%E9%82%A3%E6%9D%A1%E5%AE%9E%E7%BA%BF%E3%80%82%20%E7%89%A9%E4%BD%93%E5%8F%AF%E8%83%BD%E5%9C%A8O1p1%E8%BF%99%E6%9D%A1%E5%B0%84%E7%BA%BF%E4%B8%8A%E7%9A%84%E4%BB%BB%E4%BD%95%E4%BD%8D%E7%BD%AE%EF%BC%8C%E8%BF%99%E4%B8%AA%E4%BD%8D%E7%BD%AE%E6%98%AF%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84%E3%80%82,%E4%BD%86%E6%98%AF%E7%89%A9%E4%BD%93%E6%98%AF%E5%9C%A8%E5%B0%84%E7%BA%BF%E4%B8%8A%E7%9A%84%EF%BC%8C%E5%B0%84%E7%BA%BF%E5%9C%A8%E5%8F%B3%E8%BE%B9%E6%88%90%E5%83%8F%E5%B9%B3%E9%9D%A2%E7%9A%84%E6%8A%95%E5%BD%B1%E5%8F%88%E6%98%AF%E6%9E%81%E7%BA%BF%EF%BC%8C%E9%82%A3%E4%B9%88%E7%89%A9%E4%BD%93%E5%9C%A8%E5%8F%B3%E8%BE%B9%E7%9A%84%E6%88%90%E5%83%8F%E5%B9%B3%E9%9D%A2%E4%B8%8A%E7%9A%84%E6%8A%95%E5%BD%B1%E4%B8%80%E5%AE%9A%E5%9C%A8%E6%9E%81%E7%BA%BF%E4%B8%8A%E3%80%82%20%E4%B8%80%E6%97%A6%E7%A1%AE%E5%AE%9A%E7%89%A9%E4%BD%93%E7%9A%84%E6%8A%95%E5%BD%B1%E5%9C%A8%E6%9E%81%E7%BA%BF%E4%B8%8A%E7%9A%84%E4%BD%8D%E7%BD%AE%EF%BC%8C%E8%BF%9E%E7%BA%BF%E5%B0%B1%E8%83%BD%E7%A1%AE%E5%AE%9A%E7%89%A9%E4%BD%93%E7%9A%84%E7%A9%BA%E9%97%B4%E4%BD%8D%E7%BD%AE%E3%80%82%20%E2%80%9D%E6%9E%81%E7%BA%BF%E6%90%9C%E7%B4%A2%E2%80%9C%E7%9A%84%E5%90%AB%E4%B9%89%E5%B0%B1%E6%98%AF%E5%9C%A8%E6%9E%81%E7%BA%BF%E4%B8%8A%E6%90%9C%E7%B4%A2%EF%BC%8C%E6%89%BE%E5%88%B0%E5%AF%B9%E5%BA%94%E7%9A%84%E6%8A%95%E5%BD%B1%E7%82%B9%E3%80%82%20%E2%80%9D%E5%9D%97%E5%8C%B9%E9%85%8D%E2%80%9C%E7%9A%84%E5%90%AB%E4%B9%89%E5%B0%B1%E6%98%AF%EF%BC%8C%E4%B8%8D%E5%8D%95%E7%8B%AC%E5%8C%B9%E9%85%8D%E7%82%B9%EF%BC%8C%E8%80%8C%E6%98%AF%E5%8C%B9%E9%85%8D%E7%82%B9%E6%89%80%E5%9C%A8%E7%9A%84%E5%B0%8F%E5%9D%97%E7%9A%84%E7%9B%B8%E4%BC%BC%E7%A8%8B%E5%BA%A6%E3%80%82%20%E5%9B%A0%E6%AD%A4%E8%BF%99%E9%87%8C%E5%B0%B1%E5%8F%88%E6%98%AF%E5%9F%BA%E4%BA%8E%E7%81%B0%E5%BA%A6%E4%B8%8D%E5%8F%98%E6%80%A7%EF%BC%8C%E8%B7%9F%E5%85%89%E6%B5%81%E5%92%8C%E7%9B%B4%E6%8E%A5%E6%B3%95%E7%9A%84%E5%81%87%E8%AE%BE%E6%98%AF%E7%9B%B8%E5%90%8C%E7%9A%84%E3%80%82)<br />[5.三角化---深度滤波器---单目稠密重建（高翔slam---十三讲）](https://www.cnblogs.com/Jessica-jie/p/7730731.html)

