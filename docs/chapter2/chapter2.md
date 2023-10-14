# **2.差异与统一：坐标系变换与外参标定**

学习这件事不在乎有没有人教你，最重要的是在于你自己有没有觉悟和恒心。——法布尔

---

> **本章主要内容** \
> 1.坐标系变换 \
> 2.相机外参标定

上一章我们了解了相机内参的概念，内参主要解决三维世界与二维图像之间的映射关系。有了内参我们可以一定程度上还原相机看到了什么（但缺乏尺度）。但相机看到的数据只是处于相机坐标系，为局部观测，我们需要将局部观测转换到全局观测上，这就涉及坐标系间转换。将传感器坐标系观测转换到载体坐标系需要通过外参。本章将介绍坐标系转换及相机外参这两部分内容。
<a name="wH8Kr"></a>
## **2.1 坐标系转换**
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986314191-4c2666d6-d0cc-48d5-a49c-f247f1eb7760.png#averageHue=%23f5f4f4&from=url&height=356&id=rKEIc&originHeight=451&originWidth=847&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=668)<br />如图6，对同一物体，在不同坐标系下便有不同坐标。那么如果知道A与B的相对位置关系，且知道树在A坐标系下的坐标，我们如何求出树在B坐标系下的坐标呢，这就涉及到坐标系转换的内容。<br />首先我们有一个人为设定的世界坐标系，任何物体在这个坐标系下有其固定的坐标，这个是不变的。而这个物体被相机观测到之后，在相机坐标系下会有一个观测坐标，如图6中的A,B,C,但这个观测是相对观测，只对当前的相机坐标系有意义，对其他观测者是没有意义的，因为它无法把这个信息转换为对自己有用的信息，只有当坐标能转化到世界坐标系这个公共坐标系下，其他观测者才能利用这次观测，从而达到对环境的“有效建模”。<br />那么如何转换呢，需要使用转换矩阵来完成。接下来我们会从最简单的二维坐标系变换开始，一步一步推导出三维坐标系之间坐标转换的计算方式。
<a name="LYI3C"></a>
### **2.1.1 二维纯旋转变换**
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986314591-1331f5eb-a9c0-44fb-a8e5-0ee850a394ad.png#averageHue=%23fcfafa&from=url&height=307&id=J4U8V&originHeight=425&originWidth=535&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=386.98931884765625)<br />假设XOY为固定的世界坐标系，X'OY'为活动坐标系，在XOY坐标系下有一点P(x,y),假设X'OY'相对于XOY旋转了θ度，则根据三角函数，可以得到在X'OY'下点P的坐标P'为：<br />$\mathbf{x}^{*}=\mathbf{O D}+\mathrm{DF}=\mathbf{x} \times \cos (\theta)+\mathbf{y} \times \sin (\theta) \\
\mathbf{y}^{*}=\mathbf{P C}-\mathrm{FC}=y \times \cos (\theta)-\mathbf{x} \times \sin (\theta) \\$<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986314948-23724978-7359-4f82-bb71-2ba13cfa17b4.png#averageHue=%23fcfcfb&from=url&height=397&id=vngjH&originHeight=483&originWidth=473&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=388.9967956542969)<br />根据上述计算关系，我们也可以把P看做首先在X'OY'中测得的坐标，而XOY相对于X'OY'反向旋转了θ度（-θ），可以得到在活动坐标系下点P(x,y)，在固定世界坐标系下坐标为<br />$\mathbf{x}^{*}=\mathbf{O D}+\mathrm{DF}=\mathbf{x} \times \cos (-\theta)+\mathbf{y} \times \sin (-\theta)=\mathbf{x} \times \cos (\theta)-\mathbf{y} \times \sin (\theta) \\
\mathbf{y}^{*}=\mathbf{P C}-\mathbf{F C}=y \times \cos (-\theta)-\mathbf{x} \times \sin (-\theta)=y \times \cos (\theta)+\mathbf{x} \times \sin (\theta) \\$<br />写为矩阵形式为<br />$\left(\begin{array}{l}
x^{*} \\
y^{*}
\end{array}\right)=\left(\begin{array}{cc}
\cos \theta & -\sin \theta \\
\sin \theta & \cos \theta
\end{array}\right)\left(\begin{array}{l}
x \\
y
\end{array}\right)=R\left(\begin{array}{l}
x \\
y
\end{array}\right) \\$<br />其中R为X'OY'在XOY坐标系下的旋转矩阵
<a name="aHLlL"></a>
### **2.1.2二维纯平移变换**
两个坐标系只有平移变换，则坐标系转换很简单，如下<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986315252-1f31c1b4-49b3-4b0d-83c3-93c3034164e3.png#averageHue=%23fafafa&from=url&height=329&id=pLtT9&originHeight=466&originWidth=571&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=402.9754333496094)<br />$\mathbf{x}^{*}=\mathbf{x}+t_{x} \\
\mathbf{y}^{*}=\mathbf{y}+t_{y} \\$
<a name="nrHE1"></a>
### **2.1.3二维坐标系变换**
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986315567-78af370f-58a6-43c0-ad78-a0abc6bc07a9.png#averageHue=%23fdfcfc&from=url&height=385&id=CoZzY&originHeight=597&originWidth=717&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=462.9668884277344)<br />把上述旋转变换与平移变换结合在一起，就是二维坐标系变换<br />$\left(\begin{array}{l}
x^{*} \\
y^{*}
\end{array}\right)=\left(\begin{array}{cc}
\cos \theta & -\sin \theta \\
\sin \theta & \cos \theta
\end{array}\right)\left(\begin{array}{l}
x \\
y
\end{array}\right)+\left(\begin{array}{l}
t_{x} \\
t_{y}
\end{array}\right) \\$
<a name="Wf4TK"></a>
### **2.1.4 最终坐标系变换-扩展到三维**
二维的旋转，可以看做三维下只绕Z轴的旋转，其中Z坐标不变，旋转矩阵扩展为<br />$\left(\begin{array}{l}
x^{*} \\
y^{*} \\
z^{*}
\end{array}\right)=\left(\begin{array}{ccc}
\cos \theta & -\sin \theta & 0 \\
\sin \theta & \cos \theta & 0 \\
0 & 0 & 1
\end{array}\right)\left(\begin{array}{l}
x \\
y \\
z
\end{array}\right) \\$<br />相应的，对于不同旋转，目前只需要记住中间有一个3X3的旋转矩阵就行（该矩阵有一个特殊性质，就是属于SO3群），而平移则是从二维变为三维就行了，最后三维变换：<br />$\left(\begin{array}{l}
x^{*} \\
y^{*} \\
z^{*}
\end{array}\right)=\left(\begin{array}{ccc}
\cos \theta & -\sin \theta & 0 \\
\sin \theta & \cos \theta & 0 \\
0 & 0 & 1
\end{array}\right)\left(\begin{array}{l}
x \\
y \\
z
\end{array}\right)+\left(\begin{array}{l}
t_{x} \\
t_{y} \\
t_{z}
\end{array}\right)=R\left(\begin{array}{l}
x \\
y \\
z
\end{array}\right)+t \\$<br />但对于坐标系变换，常用齐次矩阵来表示，及把旋转变换与平移变换，放到一个矩阵里<br />$\left(\begin{array}{l}
x^{*} \\
y^{*} \\
z^{*} \\
1
\end{array}\right)=\left(\begin{array}{cccc}
\cos \theta & -\sin \theta & 0 & t_{x} \\
\sin \theta & \cos \theta & 0 & t_{y} \\
0 & 0 & 1 & t_{z} \\
0 & 0 & 0 & 1
\end{array}\right)\left(\begin{array}{l}
x \\
y \\
z \\
1
\end{array}\right) \\$<br />在原三维向量下，添加1后的坐标，称为齐次坐标，这个是几何变换中常用的数学技巧<br />$\widetilde{a^{*}}=\left(\begin{array}{cc}
R & t \\
0 & 1
\end{array}\right) \widetilde{a} \\$<br />于是可以简化到下面这种形式：<br />$\widetilde{a_{1}}=T_{2}^{1} \widetilde{a_{2}} \\$<br />其中T12称为变换矩阵，表示将坐标系2下的坐标，转换到1所需要变换。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986315890-867123ff-6466-4ee4-ad65-af6c568907a3.png#averageHue=%23fefdfd&from=url&height=330&id=JUMVx&originHeight=561&originWidth=1038&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=611.2622680664062)<br />使用坐标系变换，可以把相机坐标系（cam）下的坐标，转换到世界坐标系下。反之，也可以把世界坐标系下坐标，转换到相机坐标系下。
<a name="zqujT"></a>
## **2.2 相机外参**
<a name="gO3Kh"></a>
### **2.2.1 定义**
对于移动载体来说，相机只是载体的一个传感器，我们更感兴趣的是，通过相机观测到的物体，相对于载体而言，其坐标为什么。如果说相机坐标系为camera_link，那么载体坐标系为base_link，这二者之间存在一个变换矩阵，通常用camera_link到base_link的欧式转换表示，而这个变换矩阵，就是常说的相机外参。使用外参矩阵，可以把相机坐标系下看到的点转换到机器人坐标下。<br />通俗来讲，相机外参描述了相机安装在载体的什么地方，安装角度是怎样的。<br />如下图，相机0与相机1均不与飞行器本体坐标系重叠，假设相机0安装在本体坐标系的t处（平移），旋转矩阵为R，则其外参为（R,t）。通过相机外参，可以把相机观测转换到飞行器本体坐标系的观测中。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986316336-769c72ba-42e2-4ca6-b5ab-809cc85609e4.png#averageHue=%23b3b3b3&from=url&height=365&id=ljrCP&originHeight=726&originWidth=1132&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=569.2633666992188)
<a name="yGZVs"></a>
### **2.2.2 相机外参的标定**
如果知道相机相对于载体坐标系安装的位置与角度，是可以直接得到相机外参的，但是，由于机械安装的误差，或者运动过程的变形，这个外参是会改变的。为了获取此时的外参，常借助于一些外参标定工具，常见的外参标定工具有如下几种：<br />**OpenCV**<br />OpenCV除了提供相机内参标定外，也提供了相机外参标定的功能。通过拍摄已知的三维空间点的图像，并利用这些点在相机坐标系下的三维坐标，可以计算相机的位姿。<br />**Matlab**<br />Matlab也提供了相机外参标定的工具箱，其中包括了单目和立体相机的标定工具。通过拍摄已知的三维空间点的图像，可以计算相机的位姿。<br />**Kalibr**<br />Kalibr是ETH Zurich的一个开源相机标定工具，支持单目、双目和多目相机标定。Kalibr使用基于优化的方法来计算相机的位姿，同时也可以进行相机和IMU的联合标定。
<a name="orus0"></a>
### **2.2.3 外参标定的意义**
相机外参标定确定了相机在世界坐标系中的位置和朝向，这些信息可以用于将图像中的点转换为世界坐标系中的点，或者将世界坐标系中的点转换为图像中的点。因此，外参标定对于相机视觉算法的准确性和稳定性有着重要的影响。以下是外参标定对算法的几个方面的影响：

   1. 特征匹配：在进行特征匹配时，需要将图像中的特征点匹配到世界坐标系中的点。如果外参标定不准确，将导致匹配错误，从而影响算法的准确性。
   2. 三维重建：在进行三维重建时，需要将多张图像中的点匹配到世界坐标系中的点，并进行三维重建。如果外参标定不准确，将导致重建的几何形状不准确或者出现扭曲。
   3. 目标跟踪：在进行目标跟踪时，需要将跟踪目标在多张图像中的位置匹配到世界坐标系中的点，并进行跟踪。如果外参标定不准确，将导致跟踪不准确或者跟踪失败。
   4. 相机姿态估计：在进行相机姿态估计时，需要将相机在多张图像中的位置和朝向匹配到世界坐标系中的点，并进行姿态估计。
<a name="vlK5q"></a>
### **2.2.4 相机标定类型**
相机标定是指确定相机内部参数和外部参数的过程，以便在图像中恢复真实世界中的几何信息。常用的相机标定类型包括以下几种：

   1. **内部标定：**用于确定相机的内部参数，包括焦距、主点位置、畸变系数等。内部标定通常使用标定板等已知几何形状的物体，并采用校正方法进行计算。
   2. **外部标定：**用于确定相机在世界坐标系中的位置和方向。外部标定通常使用已知位置的标定板或者其他几何形状的物体，并采用三维重建方法进行计算。
   3. **传感器到传感器标定：**用于确定多个相机之间的相对位置和方向，以便进行双目或多目视觉处理。传感器到传感器标定通常使用标定板或者球形标定物等，采用立体匹配算法进行计算。
   4. **传感器到车体标定：**用于确定相机在车体坐标系中的位置和方向，以便进行车载视觉处理。传感器到车体标定通常使用车体固定的标定板或者其他几何形状的物体，采用三维重建方法进行计算。
   5. **姿态标定：**用于确定相机在平面或者空间中的朝向，以便进行视觉导航或者机器人控制。姿态标定通常使用旋转平台或者陀螺仪等设备，采用角度解算方法进行计算。
   6. **相机-激光雷达标定：**用于确定相机和激光雷达之间的相对位置和方向，以便进行三维点云重建或者障碍物检测。相机-激光雷达标定通常使用已知几何形状的标定板或者球形标定物，采用多视角几何约束方法进行计算。
   7. **相机-IMU标定：**用于确定相机和惯性测量单元（IMU）之间的相对位置和方向，以便进行惯性辅助导航或者姿态估计。相机-IMU标定通常使用运动平台或者旋转平台等设备，采用卡尔曼滤波或者优化方法进行计算。

<a name="o4ZEg"></a>
## 本章小结
本章主要介绍如何将相机坐标系下观测，转换到其他坐标系（如载体坐标系），紧接着介绍了相机外参。相机外参主要作用是将相机的局部观测转到载体坐标系下。载体在世界坐标系中不断运动，如何描述载体在世界坐标系下姿态，见下一章。
<a name="q5yYf"></a>
## 本章思考
1.相机外参的作用是什么？<br />2.一个载体搭载着摄像头，在平面上运动，相机外参【t(x,y,z)与R(roll,pitch,yaw)】哪些量不可以被标定出来？
<a name="TuQOq"></a>
## **附录**
<a name="NAI20"></a>
### **1. 摄像机外参标定**
求解相机的内、外参数矩阵，描述了三维世界到二维像素的映射关系<br />![image-20230618182716202.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687086475049-cd4eef91-abaa-464a-9ae8-6de4dba6dc6d.png#averageHue=%23f7eeee&clientId=u71ec186e-6a17-4&from=ui&height=206&id=x9mdn&originHeight=343&originWidth=828&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=26603&status=done&style=none&taskId=uae023d40-1a86-4d11-98e8-fdb5739cdb4&title=&width=498.2553405761719)<br />其中的$\theta$为单个像素两条边的夹角，一般我们认为这个角度为90°，早期对于一些加工不好的传感器，像素可能会是平行四边形。<br />M矩阵有11个未知量，求解投影矩阵需要最少六对点对应，但实操时一般使用多于六对点获得更鲁棒的效果<br />**径向畸变**：图像像素点以畸变中心为中心点,沿着径向产生的位置偏差,从而导致图像中所成的像发生形变<br />**产生原因**：光线在远离透镜中心的地方比靠近中心的地方更加弯曲<br />图像放大率随距光轴距离的增加而减小<br />![image-20230618184958384.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687086523239-1b925afa-793c-48a6-b2ff-ff6852f18f0c.png#averageHue=%23e4e2e2&clientId=u71ec186e-6a17-4&from=ui&height=211&id=bgmvz&originHeight=272&originWidth=314&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=41856&status=done&style=none&taskId=u974980f4-b7a4-4da9-8b7d-6e305a86e49&title=&width=243.9957275390625)<br />![image-20230618190527055.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687086536321-2688ffe1-89d9-4f41-b6e0-8cf45409f642.png#averageHue=%23f4efef&clientId=u71ec186e-6a17-4&from=ui&height=178&id=gS21G&originHeight=398&originWidth=665&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=14764&status=done&style=none&taskId=u23300135-aec4-49f9-93b0-ea855d099d4&title=&width=296.9786376953125)<br />![image-20230618190543351.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687086549357-93f8a9be-dc3f-46c1-86b9-e950f81a4e19.png#averageHue=%23f2eded&clientId=u71ec186e-6a17-4&from=ui&height=132&id=BXqcf&originHeight=241&originWidth=748&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=23543&status=done&style=none&taskId=u877ebc0a-b417-4573-b5de-2becf9b7f35&title=&width=409.2339782714844)
<a name="A9rVe"></a>
### **2 提取摄像机参数**
将投影矩阵M拆分开，参数分为内参数与外参数。<br />![image-20230618182716202.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687086475049-cd4eef91-abaa-464a-9ae8-6de4dba6dc6d.png#averageHue=%23f7eeee&clientId=u71ec186e-6a17-4&from=ui&height=147&id=IC3ye&originHeight=343&originWidth=828&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=26603&status=done&style=none&taskId=uae023d40-1a86-4d11-98e8-fdb5739cdb4&title=&width=355)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687676964325-f3bc3aec-baa5-49d9-bbf5-7dc8e737de5d.png#averageHue=%23f2f1f1&clientId=u04f2e5be-db77-4&from=paste&height=99&id=ue5d7241a&originHeight=225&originWidth=1306&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=35034&status=done&style=none&taskId=u8d0874bc-c0ca-46a3-b300-31fe85ea799&title=&width=573.2625122070312)<br />对于这里的$\rho$，是一个标量，因为我们求得的解，其实会与实际的解相差一个系数，该系数由$\rho$表示。
<a name="CxQlT"></a>
#### **2.1 提取摄像机内参数u、v与**θ
![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677759430-8a1f8808-a8dd-421b-a2c5-1037c7a8abdd.png#averageHue=%23eee3e3&clientId=u04f2e5be-db77-4&from=paste&height=76&id=u8517ccee&originHeight=145&originWidth=721&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=12803&status=done&style=none&taskId=u06843a5d-2646-4eda-9ce1-095b862deea&title=&width=379.2104797363281)<br />其中A与b分别为<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677796721-87527164-50f8-4f44-af83-aa9fbf0a3b1d.png#averageHue=%23ebebeb&clientId=u04f2e5be-db77-4&from=paste&height=100&id=u5c476f54&originHeight=117&originWidth=280&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=5667&status=done&style=none&taskId=u94e8e4a7-e06b-4bef-8793-025d24038a0&title=&width=239.31624809430699)<br />带入其中得<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677822442-9bfb0f99-555a-4399-b302-9f312f2c09c9.png#averageHue=%23f4c291&clientId=u04f2e5be-db77-4&from=paste&height=128&id=uc8b03685&originHeight=198&originWidth=809&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=22926&status=done&style=none&taskId=u9259c8ed-baf5-49d5-972e-78e5e513518&title=&width=522.9930419921875)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677871259-0ca39e11-01ce-42eb-8011-a234b2b9a3b4.png#averageHue=%23f1f1f1&clientId=u04f2e5be-db77-4&from=paste&height=113&id=ud3109a73&originHeight=169&originWidth=925&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=20719&status=done&style=none&taskId=u57d9e961-27fa-42f3-a1f8-b94d18c8ce9&title=&width=620.2708129882812)<br />其中K、R、T分别为<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677909559-ffdb9fc3-d862-4cac-825e-e05faa14af27.png#averageHue=%23f1f1f1&clientId=u04f2e5be-db77-4&from=paste&height=215&id=ud06cc5c8&originHeight=251&originWidth=284&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=10681&status=done&style=none&taskId=u6ad92aed-d703-4beb-8640-d7175bb0f5f&title=&width=242.73505163851138)<br />计算得<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687677951401-0426a555-ce42-4e4b-b7b0-bbdc64cebb39.png#averageHue=%23ececec&clientId=u04f2e5be-db77-4&from=paste&height=81&id=u0922c0f9&originHeight=142&originWidth=560&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=11577&status=done&style=none&taskId=u75c6838d-a01f-43f8-934d-7c6753919e6&title=&width=319.5908203125)<br />利用正交性质，将两行对应相乘<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687678321270-2765e855-55ad-45e6-9050-f31d6ca647eb.png#averageHue=%23ededed&clientId=u04f2e5be-db77-4&from=paste&height=101&id=u85a1810a&originHeight=167&originWidth=561&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=14415&status=done&style=none&taskId=u497ef56f-2a49-4d8a-9c6f-0510d7b0647&title=&width=339.4620666503906)<br />求相应的模，可得<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687678463041-b9486499-1d71-450e-892b-3669e1621a5c.png#averageHue=%23efeaea&clientId=u04f2e5be-db77-4&from=paste&height=116&id=ud12ea3f9&originHeight=204&originWidth=366&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=10748&status=done&style=none&taskId=u07f52633-5635-4aae-87f5-52ffc6158b2&title=&width=208.8076934814453)<br />自己点乘自己，可得<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687678560299-b651ba5a-d8b5-459f-9cfc-f4a1e40ce646.png#averageHue=%23ffeded&clientId=u04f2e5be-db77-4&from=paste&height=105&id=u5cf59678&originHeight=151&originWidth=788&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=17364&status=done&style=none&taskId=u9126a13c-42d2-4e06-9b11-4d8e1c1568f&title=&width=547.4989624023438)<br />得到θ<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687678626948-278dfc28-1258-4c9d-aeeb-7ad35e166c3b.png#averageHue=%23e5e5e5&clientId=u04f2e5be-db77-4&from=paste&height=66&id=u364d71a0&originHeight=108&originWidth=556&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=8666&status=done&style=none&taskId=ua1c8b081-84b4-4551-928a-a8a03b2a11c&title=&width=341.1949768066406)<br />最后得到α与β<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687678784786-47a8e0d7-02b1-4f7d-8316-943ad6f38895.png#averageHue=%23e4e4e4&clientId=u04f2e5be-db77-4&from=paste&height=75&id=u07793813&originHeight=138&originWidth=444&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=10637&status=done&style=none&taskId=u9da8446f-1cbe-4722-bcf6-bcab101769c&title=&width=240.45086669921875)
<a name="VwuBA"></a>
#### **2.2 提取摄像机外参数r1、r2与**r3
![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687679100354-2365f32c-9303-44c5-8090-2f081f967ff4.png#averageHue=%23eeeeee&clientId=u04f2e5be-db77-4&from=paste&height=105&id=ue5c0ed2e&originHeight=197&originWidth=559&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=12550&status=done&style=none&taskId=u4edf7467-b424-4b05-bd9e-9aaebc8cc39&title=&width=296.72650146484375)<br />再求得外参数T<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687679171442-d89aff6e-6004-46c3-b082-2f749fa320ba.png#averageHue=%23e5e5e5&clientId=u04f2e5be-db77-4&from=paste&height=30&id=uc3479adf&originHeight=59&originWidth=256&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=2864&status=done&style=none&taskId=uc22e8968-178a-46ee-84a2-ad3e0dffeee&title=&width=131.7911376953125)
<a name="Jq34d"></a>
## **参考**
[1.坐标变换最通俗易懂的解释（推到+图解）](https://blog.csdn.net/weixin_45590473/article/details/122848202)<br />[2.计算机视觉之三维重建（深入浅出SfM与SLAM核心算法）——2.摄像机标定_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1DP41157dB?p=2&vd_source=6deef2eecab362ce87a170f121391888)
