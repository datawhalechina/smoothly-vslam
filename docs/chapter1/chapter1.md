# **1.一幅图像的诞生：相机模型及坐标系**

一个能思考的人，才真是一个力量无边的人。——巴尔扎克

---

> **本章主要内容：** \
> 本教程结构 \
> 1.相机模型 \
> 2.相机成像的几个坐标系 \
> 3.相机标定


本节主要介绍相机成像过程及相机模型， 相机模型是描述相机如何将三维世界投影到二维图像平面的数学模型，一般包括：\
**1.投影模型：** 描述从三维世界坐标到图像坐标的映射，常见的模型有针孔相机模型。 \
**2.畸变模型：** 描述实际物理相机镜头因为光学误差或设计缺陷所产生的畸变 ，常见的包括径切变rad-tan模型或者KB模型。
<div style="text-align: center;">
    <strong>相机模型</strong> = <strong>投影模型 (Projection Model)</strong> + <strong>畸变模型 (Distortion Model)</strong>
</div>

在常用的相机标定工具如kalibr中，用--models参数来指定相机模型，如： \
针孔径切畸变相机：pinhole-radtan \
针孔鱼眼畸变相机： pinhole-equi \
其中pinhole指投影模型为针孔，radtan与equi分别是普通相机和鱼眼相机的畸变模型。
<a name="CQnkQ"></a>
## **1.1 针孔投影模型**
针孔相机模型是目前大多数相机的成像模型，其成像原理为小孔成像，回顾一下我们小学二年级学过的内容，光线在同一介质中按直线传播，在穿过小孔之后，会在另一面形成倒立缩小的实像。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986390545-0b0cb59b-73de-4d12-bc91-bfe139dfd562.png#averageHue=%23e8e7e6&from=url&height=333&id=nimTZ&originHeight=420&originWidth=827&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=656)<br />如果不借助其他东西，这个成像过程会遵循如下原理：小孔越小，成像越清晰，但因为进光更少，所以图案会越暗。 \
如下图，小孔直径从2mm变到0.35mm过程，**图像越来越清晰，但也越来越暗。** <br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986390812-026c7e81-0397-4ec3-90b3-383c00d74fb1.png#averageHue=%236f6f6f&from=url&height=377&id=blUR7&originHeight=432&originWidth=584&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=508.9840087890625)<br />但我们总是既要又要，想要照片既清晰，同时又要有足够的亮度。从物理上来说，就是要让足够的光线进来，同时又能把光汇聚在一起。为了达到这个效果，透镜闪亮登场：<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986391243-dfc73522-763c-4381-8406-1e4729d0fbee.png#averageHue=%23f4f3f3&from=url&height=292&id=UkgnL&originHeight=362&originWidth=661&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=533.9866333007812) 

透镜是现代相机的一大杀手锏，解决了成像的清晰度与亮度问题，为了达到更好的成像效果，现代相机镜头都是由多种透镜组合而成：

![](https://cdn.nlark.com/yuque/0/2026/jpeg/1782698/1771601881457-ff3ad660-00ac-4bb5-8e87-21d11aaae275.jpeg)

简单来说，**相机 = 镜头 （Lens） +  图像传感器  （Image Sensor）**

![](https://cdn.nlark.com/yuque/0/2026/png/1782698/1771602696099-5fc0e13d-7738-4d07-9a34-745f6a0859af.png)

## **1.2 畸变模型**
光线在穿过众多镜片到达底片上时，与理想的小孔成像的光路会有所差异，比如下方的鱼眼镜头成像过程，为了模拟镜头对光线的折射效果，于是出现了畸变模型。\
![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687660584100-75eae03e-ee8e-4685-8c00-a43d957c77d5.png)

最常用的畸变模型有Radtan径切畸变模型(对于标准镜头)及Kannala-Brandt模型（对于广角及鱼眼镜头）

### **1.2.1径向畸变**
径向畸变的原因与相机镜头的设计和光学特性密切相关。特别是对于广角镜头，视角较宽，光线进入镜头时，靠近边缘的光线会受到镜头元素的非均匀折射，导致图像畸变。\

**径向畸变的类型：**

+ **桶形畸变（Barrel Distortion）**：这种畸变通常发生在广角镜头和超广角镜头中，图像中心的物体保持正常，而越靠近边缘的物体越容易被拉伸，造成图像边缘像一个桶一样膨胀。
+ **表现**：图像的边缘部分会被拉向外部，像桶的形状。
+ **枕形畸变（Pincushion Distortion）**：这种畸变常见于长焦镜头，图像的边缘部分被压缩，导致边缘向内弯曲，像枕头一样。
+ **表现**：图像的边缘部分会被压向中心，形成枕形的效果。

![](https://cdn.nlark.com/yuque/0/2026/png/1782698/1771653053445-2fb92245-a3b8-4ab5-9fa4-8ccf8f6b78ae.png) 
对于径向畸变，常用如下公式进行修正<br />$\begin{array}{l}
x_{\text {distortion }}=x\left(1+k_{1} r^{2}+k_{2} r^{4}+k_{3} r^{6}\right) \\
y_{\text {distortion }}=y\left(1+k_{1} r^{2}+k_{2} r^{4}+k_{3} r^{6}\right)
\end{array} \\
\text { 其中: } r^{2}=x^{2}+y^{2}$<br />k1,k2,k3称为径向畸变系数。

### **1.2.2切向畸变**

切向畸变来自于整个摄像机的组装过程。由于透镜制造上的缺陷使得透镜本身与图像平面不平行而产生的，如下图所示。\
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986394315-ff2ca78a-5437-4bc3-a92d-f37234818c77.png)
切向图像畸变使用如下公式进行修正
<br />$\begin{aligned}
x_{\text {distortion }} & =x+\left[2 p_{1} x y+p_{2}\left(r^{2}+2 x^{2}\right)\right] \\
y_{\text {distortion }} & =y+\left[p_{1}\left(r^{2}+2 y^{2}\right)+2 p_{2} x y\right]
\end{aligned}
\\
其中:  r^{2}=x^{2}+y^{2}$<br />其中 p1,p2为切向畸变系数。<br />将径向畸变与切向畸变校正，结合在一起，便是常见的畸变过程。<br />$\begin{array}{l}
x_{\text {distortion }}=x\left(1+k_{1} r^{2}+k_{2} r^{4}+k_{3} r^{6}\right)+\left[2 p_{1} x y+p_{2}\left(r^{2}+2 x^{2}\right)\right] \\
y_{\text {distortion }}=y\left(1+k_{1} r^{2}+k_{2} r^{4}+k_{3} r^{6}\right)+\left[p_{1}\left(r^{2}+2 y^{2}\right)+2 p_{2} x y\right]
\end{array}
\\
其中:  r^{2}=x^{2}+y^{2}$<br />
等式右边的(x, y)是理想小孔成像的照片上像素点坐标，没有畸变，将其带入等式右边，经过径向和切向变换后，得到左边的添加畸变后的实际点坐标(Xdistortion, Ydistortion)，就是我们实际获得的照片，如果我们将这个映射取逆，就可以对存在畸变的实际图像，进行去畸变，得到去畸变后的图像：
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986394648-56f1f7d0-7375-4bd0-a09a-066b219b5fdd.png)

### **1.2.3Kannala-Brandt模型**
Kannala-Brandt模型（简称KB模型）是处理广角和鱼眼相机时最主流的相机模型之一。它在2006年由Juho Kannala和Sami S. Brandt提出，其核心思想是使用一个高阶多项式来统一描述各种镜头的投影和畸变，尤其擅长处理超过180°视场的极端广角场景。简单来说，KB模型对入射光线入射角与投影位置进行近似模拟，最简单的就是认为投影位置与入射角θ成线性正比关系，即

$$
r_{d} = f \theta
$$

上面的公式就是等距模型，KB模型中一共有四种假设模型，通过这四个模型，发现投影位置是θ的奇函数，将这些式子按泰勒级数展开，畸变位置可以用θ的奇次多项式表示，即

$$ \theta_{d}=k_{0} \theta+k_{1} \theta^{3}+k_{2} \theta^{5}+\cdots $$

取前5项，给出了足够的自由度来很好的近似各种投影模型。

$$ \theta_{d}=k_{0} \theta+k_{1} \theta^{3}+k_{2} \theta^{5}+k_{3} \theta^{7}+k_{4} \theta^{9} $$

上式是根据四种鱼眼相机投影模型得出的一种通用鱼眼相机多项式模型。OpenCV中鱼眼相机标定方法就是用的这种KB模型。其中一般会假设k0为1，然后标定得到k1,k2,k3,k4四个畸变系数。

## **1.3 不同相机及其适用的畸变模型**
首先要注意的是，不同相机模型，畸变的大小是不同的，简单来说，FOV(视场角)越大的相机，畸变程度就越大。

![](https://cdn.nlark.com/yuque/0/2026/png/1782698/1771652733011-ed7a01bd-cce4-4077-bdd7-dae2b82a73c8.png)

在视觉SLAM中，不同FOV镜头适用的畸变模型如下：

| 镜头类型 | FoV范围 | 畸变模型 | 特点 |
| --- | --- | --- | --- |
| 长焦 | 20°~40° | radtan |  大多数长焦镜头会表现出一些**轻微的桶形畸变**，但一般不显著。由于其较长的焦距，图像的边缘处畸变通常较不明显   |
| 标准 | 40°~60° | radtan | 标准镜头通常表现出非常少的畸变，尤其是在相机的中心区域。大多数标准镜头的畸变在图像中心非常轻微，但在图像的边缘可能出现**轻微的桶形畸变** |
| 广角 | 60°~90° | 高阶radtan |  广角镜头有较大的视场角，因此畸变较为明显，尤其是在图像的边缘区域。广角镜头通常会出现**桶形畸变**（图像中心物体正常，边缘物体被拉伸或扭曲），并且会出现更多的**几何畸变**。   |
| 超广角 | 90°~180° | KB模型 |  超广角镜头有非常大的视场角，因此畸变会更加严重，尤其是在画面边缘。超广角镜头经常出现**明显的桶形畸变**，而且在一些极端情况下，还会出现**鱼眼效应**（图像中心正常，边缘极度弯曲）   |
| 鱼眼 | ≥180° | KB模型 |  鱼眼镜头的畸变非常强烈，几乎是**极端的桶形畸变**，并且呈现出明显的“球形”畸变，导致图像中心正常而边缘区域严重拉伸。鱼眼镜头的视场角可以达到180度，甚至更广，所有图像都被极度弯曲。   |
## **1.4 感光元件**
现代相机都是数码相机，使用电子感光元件进行图像记录。感光元件一般为CCD或者CMOS，其原理是将光转化为模拟电信号来记录。 \
![](https://cdn.nlark.com/yuque/0/2026/png/1782698/1771657003181-f9171815-f3c3-43a5-addc-b79c506c4631.png) 

> 电子感光元件也叫图像传感器（Image Sensor），分为两种：一种是广泛使用的 CCD(电荷耦合)元件，另一种是 CMOS(互补金属氧化物半导体)器件。其产生的模拟信号，首先经过模拟信号放大器进行信号放大，进而经过数模转换电路（DAC）变为数字图像，数字图像再经过 ISP（Image Signal Processor）图像处理器进行数字图像处理，最后数字图像经过压缩编码算法，存储到 SD 卡中成为一个照片文件。
>
> 1. CCD
>
> CCD 全称 Charge Coupled Device，它使用一种高感光度的半导体材料制成，由许多感光单位组成，通常以百万像素为单位。当 CCD 表面受到光线照射时，每个感光单位会将电荷反映在组件上，即把光转换为电荷，所有的感光单位所产生的信号加在一起，就构成了一幅完整的画面。
>
> 2. CMOS
>
> CMOS 全称 Complementary Metal-Oxide Semiconductor，它主要是利用硅和锗这两种元素所做成的半导体，使其在 CMOS 上共存着 N 极和 P 极的半导体，这两个互补效应所产生的电流即可被处理芯片记录为影像。
>
> 两者最主要的区别在于：CCD 传感器的图像质量优于 CMOS 传感器，而 CMOS 传感器在成像速度、功耗、价格等方面优于 CCD 传感器。
>

传感器生成的图像，是对连续世界的离散采样，即一个一个像素，我们所说的图像，也是由这一个一个像素构成，在计算机下面来看，就是一个二维数组。下图中的uv坐标系就是像素坐标系。
![](https://cdn.nlark.com/yuque/0/2026/png/1782698/1771657195302-da4a4bd4-e88d-45e4-89bb-45b1c9889eae.png)

<a name="kpuFm"></a>
## **2.2 相机成像的几个坐标系**
要把相机拍摄的照片与实际物体关联起来，就要建立三维世界到二维图像平面的映射关系，这个过程主要通过世界坐标系，相机坐标系，图像坐标系与像素坐标系这几个坐标系之间的转换来实现。
<a name="t69Jy"></a>
### **2.2.1成像坐标系之间的关系**
世界坐标系下物体的光线（世界坐标系），通过透镜（相机坐标系），投射到感光原件上（图像坐标系），最后计算机在像素坐标系（离散化过程）上做处理。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986391552-09e1f357-0aaf-491b-9360-9d48ccf83700.png#averageHue=%23c3ad94&from=url&height=289&id=j2KM4&originHeight=427&originWidth=1000&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=677.2740478515625)<br />**世界坐标系**：用于表示空间物体的绝对坐标，使用（Xw,Yw,Zw）表示，世界坐标系可通过旋转和平移得到相机坐标系。<br />**相机坐标系**：以相机的光心为坐标系原点，Xc.Yc轴平行于图像坐标系的x,y轴，相机的光轴为Zc轴，坐标系满足右手法则，相机的光心可理解为相机透镜的几何中心。<br />**图像物理坐标系**：坐标原点在CCD图像平面的中心x,y轴分别平行于图像像素坐标系的(u,v)轴，坐标用(x,y)表示。<br />**图像像素坐标系**：表示三维空间物体在图像平面上的投影，**像素是离散化的**，其坐标原点在CCD图像平面的左上角，u轴平行于CCD平面水平向右，v轴垂直于u轴向下，坐标使用(u,v)来表示。图像宽度W，高度H。
<a name="SYpV2"></a>
### **2.2.2 坐标计算**
![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986391785-d7c5d2c5-1a31-4690-839d-ea1187575387.png#averageHue=%23fcfbfa&from=url&id=qBc3w&originHeight=504&originWidth=883&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=)<br />三维坐标投影到成像平面的坐标（**完成三维到二维点的映射**），可以通过相似三角形得出，对于相机坐标系下的P(X,Y,Z)，成像坐标为：<br />$\frac{f}{Z}=-\frac{X^{\prime}}{X}=-\frac{Y^{\prime}}{Y}$<br />为了方便运算，取对称的镜像进行计算，效果等价。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986392278-85592406-7069-4440-bbad-03d8252d1c05.png#averageHue=%23fafafa&from=url&id=vxoSW&originHeight=346&originWidth=997&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=)<br />$\frac{f}{Z}=\frac{X^{\prime}}{X}=\frac{Y^{\prime}}{Y}$<br />可得：<br />$X^{\prime}=f \frac{X}{Z}$<br />$Y^{\prime}=f \frac{Y}{Z}$<br />计算机中的图像，是一个个像素构成，且其并不是图像中心为坐标原点，而是一般把左上角规定为坐标原点，要在计算机中处理图像信息，需要在得到成像平面上的坐标后，把成像坐标，转为像素坐标。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986392829-dc8f8311-ec2d-4920-9494-f7a42b1aa0ca.png#averageHue=%23f4f3f3&from=url&height=347&id=cUKgL&originHeight=530&originWidth=558&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=364.9952087402344)<br />其中O1是投影后的坐标系原点，位于图像中心，而在计算机图像处理库中（如OpenCV）,则是定义的左上角O0为图像坐标系原点，横坐标轴为u轴，向右，纵坐标轴为v轴，向下。<br />相机内感光原件（如cmos）是一个一个小格子拼接而成的，可以认为是离散的，这个小格子可能不是正方形，要将投影坐标P'转化为像素坐标，需要经历如下过程：<br />计算P'到图像中心的像素距离<br />$u^{\prime}=\frac{X^{\prime}}{\alpha_{x}}=\frac{f}{\alpha_{x}} \frac{X}{Z}=f_{x} \frac{X}{Z}$<br />$v^{\prime}=\frac{Y^{\prime}}{\alpha_{y}}=\frac{f}{\alpha_{y}} \frac{Y}{Z}=f_{y} \frac{Y}{Z}$<br />其中f为成像焦距，αx与αy为u,v方向像素的长度。<br />最后转为O0下的坐标<br />实现摄像机下三维世界的点到像素平面二维图像平面的点的映射，f为单位米转化为像素的数量，**非线性变换**如下<br />$u=u^{\prime}+c_{x}=f_{x} \frac{X}{Z}+c_{x}$<br />$v=v^{\prime}+c_{y}=f_{y} \frac{Y}{Z}+c_{y}$<br />引入齐次坐标，改为线性变换，以方便运算<br />$\left(\begin{array}{ccc}
u \\
v \\
1
\end{array}\right)=\frac{1}{Z}\left(\begin{array}{ccc}
f_{x} & 0 & c_{x} \\
0 & f_{y} & c_{y} \\
0 & 0 & 1
\end{array}\right)\left(\begin{array}{c}
X \\
Y \\
Z
\end{array}\right)=\frac{1}{Z} K P \\$<br />其中<br />$K=\left(\begin{array}{ccc}
f_{x} & 0 & c_{x} \\
0 & f_{y} & c_{y} \\
0 & 0 & 1
\end{array}\right)$<br />就是通常所说的相机内参矩阵<br />通过内参矩阵K，就可以将相机坐标系的下三维的坐标点转换为图像上的二维像素坐标。而对于世界坐标系上的坐标，转换到相机坐标系中则涉及的是不同坐标系中坐标转换，这个会在第二章中进行讲解。



## **本章小结**
本节以针孔相机模型作为基本模型，简要介绍了相机成像原理，相机模型中的几个坐标系及相机的畸变及相机标定的方法。最后鱼眼相机模型作为扩展介绍。<br />通过相机标定，可以获得相机内参及畸变系数，这解决了相机坐标系，图像坐标系，像素坐标系之间的转换过程，属于投影过程。而世界坐标系到相机坐标系之间的转换，则涉及到不同坐标系之间的空间转换，这部分涉及的参数为外参，这部分内容在下节进行讲解。
<a name="ynZLk"></a>
## 本章思考
1.叙述相机内参的物理意义。如果一幅图像长宽缩小变为原来的一半而其他不变，那么它的内参将如何变化？<br />2.调研全局快门(global shutter)相机和卷帘快门(rolling shutter)相机的异同。它们在SLAM中有何优缺点？
<a name="CHe9l"></a>

## 扩展
**a.透镜的聚焦和失焦**
>关于透镜，有两个概念：聚焦与失焦 \
聚焦：从物体不同部分射出的光线，通过镜头之后，聚焦在底片的一个点上，使影像具有清晰的轮廓与真实的质感，这个过程称为聚焦。 \
失焦：即接收的点的信息未聚焦到一起会导致成像模糊。
注意：物体“聚焦”有特定距离（景深），在景深内可清晰成像，景深外成像模糊。 \ 

加入透镜之后，成像规律会有一点变化，此时当物体离透镜不同距离时，会形成不同的像。当物体处于凸透镜的 2 倍焦距之外，会形成倒立的、缩小的实像。一倍焦距到二倍焦距之间，则会形成倒立的、放大的实像。成像物体则在一倍焦距内，成正立的、放大的虚像。

一般地，相机成像时，**物体在透镜的二倍焦距之外**。而对于投影仪，则会把成像物体放在一倍焦距到二倍焦距之间。对于放大镜，成像物体则在一倍焦距内。

![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1693316432667-185ba964-b59e-4982-826f-d54b474d156b.png) 

**b.鱼眼相机模型** \
鱼眼镜头一般是由十几个不同的透镜组合而成的，在成像的过程中，入射光线经过不同程度的折射，投影到尺寸有限的成像平面上，使得鱼眼镜头与普通镜头相比起来拥有了更大的视野范围。下图表示出了鱼眼相机的一般组成结构。最前面的两个镜头发生折射，使入射角减小，其余的镜头相当于一个成像镜头，这种多元件的构造结构使对鱼眼相机的折射关系的分析变得相当复杂。<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687660584100-75eae03e-ee8e-4685-8c00-a43d957c77d5.png#averageHue=%23f7f7f6&clientId=u066aa5a8-e0e4-4&from=paste&id=oC0TR&originHeight=230&originWidth=537&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=u7bdac733-b9b6-4cd5-9f51-dddefe3dd74&title=) 

**b.鱼眼相机成像模型** \
研究表明鱼眼相机成像时遵循的模型可以近似为单位球面投影模型。可以将鱼眼相机的成像过程分解成两步：第一步，三维空间点线性地投影到一个球面上，它是一个虚拟的单位球面，它的球心与相机坐标系的原点重合；第二步，单位球面上的点投影到图像平面上，这个过程是非线性的。下图表示出了鱼眼相机的成像过程。<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687660618939-3f5d7d9f-a374-4645-9821-9964e3251aba.png#averageHue=%23f7f7f7&clientId=u066aa5a8-e0e4-4&from=paste&id=OkCWP&originHeight=294&originWidth=233&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=u84b7e864-b123-4e4d-8c2e-6a0e5bfb9d1&title=)<br />我们知道，普通相机成像遵循的是针孔相机模型，在成像过程中实际场景中的直线仍被投影为图像平面上的直线。但是鱼眼相机如果按照针孔相机模型成像的话，投影图像会变得非常大，当相机视场角达到180°时，图像甚至会变为无穷大。所以，鱼眼相机的投影模型为了将尽可能大的场景投影到有限的图像平面内，允许了相机畸变的存在。并且由于鱼眼相机的径向畸变非常严重，所以鱼眼相机主要的是考虑径向畸变，而忽略其余类型的畸变。
<a name="QwJF7"></a>
### **1* 鱼眼相机投影函数**
为了将尽可能大的场景投影到有限的图像平面内，鱼眼相机会按照一定的投影函数来设计。根据投影函数的不同，鱼眼相机的设计模型大致能被分为四种：等距投影模型、等立体角投影模型、正交投影模型和体视投影模型。下面的四种鱼眼相机的投影模型反映出了空间中的一点P是如何投影到球面上，然后到图像平面上成像的。<br />1、等距投影模型<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687660957922-a6ebbb79-3eb5-437d-a05e-7cb70d4a7c4d.png#averageHue=%23f8f8f8&clientId=u066aa5a8-e0e4-4&from=paste&height=242&id=bpNAU&originHeight=242&originWidth=225&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=ubdd671d3-fca4-4ada-bdd9-868907e2709&title=&width=225)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687660983508-d1ade030-a453-4955-829c-53f445b881af.png#averageHue=%23faf8f7&clientId=u066aa5a8-e0e4-4&from=paste&height=32&id=MGzoP&originHeight=38&originWidth=89&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=1409&status=done&style=none&taskId=uefcc7702-ec3e-4751-beea-4495cc14922&title=&width=76.06837885854758)<br />上述式子中，rd表示鱼眼图像中的点到畸变中心的距离，是鱼眼相机的焦距，是入射光线与鱼眼相机光轴之间的夹角，即入射角。 <br />2、等立体角投影模型<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661008609-8b68d151-b609-4e05-acd0-81e5bad7b128.png#averageHue=%23f7f7f7&clientId=u066aa5a8-e0e4-4&from=paste&height=242&id=KH8Te&originHeight=242&originWidth=225&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=uc456cad4-ec70-40fa-b9cd-d5a4beeac09&title=&width=225)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661038593-b1e1d476-df94-4e89-8c07-052c59ed5e77.png#averageHue=%23faf8f7&clientId=u066aa5a8-e0e4-4&from=paste&height=48&id=MEUqn&originHeight=56&originWidth=160&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=3382&status=done&style=none&taskId=ua001d2e3-8ebe-4714-9b65-bb400ca71e4&title=&width=136.75214176817542)<br />3、正交投影模型<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661076454-c7088e4b-ed7b-4145-9802-8fb0a91cc83b.png#averageHue=%23f8f8f7&clientId=u066aa5a8-e0e4-4&from=paste&height=242&id=Dkw6g&originHeight=242&originWidth=228&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=ue59659ae-63ca-4f9e-be3f-b68fd9a79fb&title=&width=228)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661087815-acc9a0ac-6908-45bc-a370-93fffbb79a7d.png#averageHue=%23f8f6f3&clientId=u066aa5a8-e0e4-4&from=paste&height=30&id=bfiqC&originHeight=35&originWidth=132&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=2623&status=done&style=none&taskId=uda9dde4c-f949-4d70-a464-ec3d612d839&title=&width=112.82051695874472)<br />4、体视投影模型<br />![](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661111947-799bd123-f1a7-474b-9deb-4d495f9bec21.png#averageHue=%23f7f7f7&clientId=u066aa5a8-e0e4-4&from=paste&height=242&id=XjnKO&originHeight=242&originWidth=228&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=ubf455b81-c61d-4b66-932c-fdf87de26fb&title=&width=228)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/36012760/1687661122635-f556dd0c-5302-4b9a-965b-74a0de31c375.png#averageHue=%23faf9f8&clientId=u066aa5a8-e0e4-4&from=paste&height=57&id=NHBuD&originHeight=67&originWidth=157&originalType=binary&ratio=1.1699999570846558&rotation=0&showTitle=false&size=3493&status=done&style=none&taskId=uf357e6d4-10fc-4c49-8509-5abc178dc6f&title=&width=134.18803911002212)
<a name="qVgNV"></a>

## 附录
<a name="WRl86"></a>
## **1.1相机内参标定简介**
到目前为止，相机成像有两大转换过程，相机投影及畸变消除，主要由：<br />$K=\left(\begin{array}{ccc}
f_{x} & 0 & c_{x} \\
0 & f_{y} & c_{y} \\
0 & 0 & 1
\end{array}\right)$<br />内参矩阵以及畸变参数：Distortioncoefficients=(k1，k2，k3, p1，p2)，这两种参数来决定。而求出这两类参数的过程，就是相机标定，目前相机常用的方法是借助棋盘格标定板，在相机前面拿着标定板上下左右前后移动，然后借助标定算法来求出以上参数，常用的标定算法有张正友标定法。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683986395167-73c8a8c5-1143-47b8-8ab1-bb2c1492f913.png#averageHue=%23b9bab2&from=url&height=417&id=JEqcg&originHeight=587&originWidth=807&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=&width=573.2617797851562)
<a name="oc7IY"></a>
## 1.2 相机内参标定工具
<a name="NDPEE"></a>
### **1.2.1 OpenCV**
OpenCV是一款广泛使用的计算机视觉库，其中包含相机内参标定的相关函数。在OpenCV中，使用calibrateCamera函数进行相机内参标定，该函数使用棋盘格等标定板，通过对标定板拍摄的多幅图像进行处理，得出相机的内参参数。OpenCV还提供了相关的可视化工具，如drawChessboardCorners函数，用于显示标定板的角点，以及projectPoints函数，用于将3D点投影到2D图像平面上。

1. 循环读取图片
2. 使用findChessboardCorners函数检测角点（需提前输入角点数）
3. 使用find4QuadCornerSubpix函数对角点进行亚像素精确化
4. 可用drawChessboardCorners将角点显示
5. 根据角点数和尺寸创建一个理想的棋盘格（用point向量存储所有理论上的角点坐标）
6. 通过calibrateCamera函数由理想坐标和实际图像坐标进行标定，可得到标定结果
7. 由projectPoints函数计算反向投影误差

![](https://cdn.nlark.com/yuque/0/2023/webp/36012760/1687663768616-862df2ce-85c5-46b6-9695-a0c9ea16bef7.webp#averageHue=%23303030&clientId=u066aa5a8-e0e4-4&from=paste&height=434&id=SJuki&originHeight=882&originWidth=1184&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=u1c996264-c94e-448c-8285-d36a0ac7531&title=&width=582)
<a name="poP1N"></a>
### **1.2.2 MATLAB**
MATLAB也提供了相机内参标定的工具箱，包括Camera Calibration Toolbox和Image Processing Toolbox等。Camera Calibration Toolbox使用标定板对相机进行标定，并提供了可视化工具，如ShowExtrinsics函数，用于显示相机的外参参数。Image Processing Toolbox提供了更加高级的算法，如多目相机标定，以支持更加复杂的应用场景。<br />简单过程如下：

1. 应用程序中找到Camera Calibration
2. 添加标定板拍摄图片（按Ctrl可一次添加多张）
3. 输入棋盘格每格的尺寸大小
4. 显示已检测出的棋盘格，点击Calibration，开始标定。
5. 得到标定结果（平均误差小于0.5即可认为结果可靠）
6. 可查看标定结果和程序

![](https://cdn.nlark.com/yuque/0/2023/webp/36012760/1687663682845-d22fc5a1-99a8-4335-b590-2996e28fda18.webp#averageHue=%23aaa89f&clientId=u066aa5a8-e0e4-4&from=paste&height=356&id=RP7n9&originHeight=679&originWidth=1184&originalType=url&ratio=1.1699999570846558&rotation=0&showTitle=false&status=done&style=none&taskId=uef08f4ec-b8ae-43a2-8f43-558c25f8784&title=&width=621.2625122070312)
<a name="emm9n"></a>
### **1.2.3 ROS**
ROS（Robot Operating System）是一种常用的机器人操作系统，其中包含相机内参标定的相关包，如camera_calibration。该包通过对标定板拍摄的多幅图像进行处理，计算出相机的内参参数，并自动保存标定结果。此外，ROS还提供了一系列可视化工具，如image_view，用于显示相机的图像和标定结果。
<a name="BATKX"></a>
## **参考**
[1.图像处理——4个坐标系及相关转换图像像素坐标系 图像物理坐标系 相机坐标系 世界坐标系](https://blog.csdn.net/MengYa_Dream/article/details/120233806)<br />[2.相机畸变模型及去畸变计算](https://blog.csdn.net/TimeRiverForever/article/details/117283430)<br />

[3.相机透镜及畸变](https://robot.czxy.com/docs/camera/chapter01/05-distortions/)

[4. 最详细、最完整的相机标定讲解_51CTO博客_相机标定](https://blog.51cto.com/u_15242250/2870251)

[5.鱼眼相机成像模型_sylvester0510的博客-CSDN博客](https://blog.csdn.net/u010128736/article/details/52864024)

[6.The Camera Basics for Visual SLAM](https://www.kudan.io/blog/camera-basics-visual-slam/)
