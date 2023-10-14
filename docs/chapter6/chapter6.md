# **6.换一个视角看世界：前端-视觉里程计之对极几何**

荣誉和财富，若没有聪明才智，是很不牢靠的财产。——德谟克里特

---

> **本章主要内容** \
> 1.对极几何 \
> 2.本质矩阵及其求解 \
> 3.单应矩阵及其求解 \
> 4.三角测量

**对极几何**是**立体视觉中的几何关系**，描述相机从不同位置拍摄3D场景时，3D点与相机位姿，图像观测像素坐标之间的几何关系。这种几何关系可以作为约束应用到求解相机运动及特征点3D坐标中。<br />**关于立体视觉：**
> 立体视觉(Stereo Vision)是什么呢？我们可以这样理解：
> 立体视觉(StereoVision) = 寻找相关性(Correspondences) + 重建(Reconstruction)
> - Correspondences:给定张图片中的像素P点，寻找其在另一张图片中的对应点Pr。
> - Reconstruction:给定一组对应点对(P,P,),计算其在空间中对应点P的3D坐标。
> 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685265589313-38e0dd6a-8935-4178-8e6b-6517be012151.png#averageHue=%23e0dedc&clientId=ud185277d-3a15-4&from=paste&height=261&id=ucd4e513b&originHeight=326&originWidth=657&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=130765&status=done&style=none&taskId=u805942d2-4983-4a74-af33-f331d551ac8&title=&width=525.6)


对极几何中存在以下几个概念：对极点，对极线，对极平面。其中**对极线构成的约束称为对极线约束**，也称为**对极约束**。<br />![对极几何1.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685257312236-c843ef08-d537-49d7-9216-0660c1c2df8e.png#averageHue=%23d8f2bd&clientId=ud185277d-3a15-4&from=paste&height=245&id=uad61ea8f&originHeight=306&originWidth=457&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=39482&status=done&style=none&taskId=u9ba5f4d3-2984-4470-8e3b-905965c4f6d&title=&width=365.6)<br />**对极点：**左相机光心OL在右相机平面上的成像点eR，称为其中一个对极点，类似的，OR在左相机上的成像点eL，也是对极点。对极点是虚拟的点，如果相机之间不能观测到对方光心，则对极点会在图像之外。<br />**对极线：**在相机OL观测到一个点XL，实际情况该XL可能对应3D坐标中任意一个Xi，因为线**O**L**X**被因为与左相机中心重合而被左相机视为一个点。但对于右相机，每个Xi则将有不同观测，这些观测为其图像平面中的一条线，该线称为对极线。如图，右摄像机中的那条线（**e**R**X**R）就称为**对极线**。对称地，右相机视线**O**R**X**为一个点，而被左相机视为对极线（**e**L**X**L）。<br />对极线是3D空间中点**X**的位置的函数，一个兴趣点对应一组对极线（XLeL,XReR）。由于线**O**L**X**通过透镜**O**L的光学中心，因此右图中相应的对极线必须通过**e**R（并且对应于左图中的极线）。一幅图像中的所有对极线都包含该图像的对极点。<br />**对极平面：**趣点**X**与两相机中心**O**L、**O**R三点形成的平面称为**对极平面**。对极平面与每个相机的图像平面相交形成线即为对极线。无论**X**位于何处，所有对极平面和对极线都与对极点相交。<br />**基线：**两个相机光心相连的直线OLOR称为基线。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685258773597-f9f4ea5a-b764-4877-a000-ec9d3514c66f.png#averageHue=%236c725b&clientId=ud185277d-3a15-4&from=paste&height=266&id=MRkjP&originHeight=333&originWidth=842&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=396606&status=done&style=none&taskId=u5d8db893-fbe5-452d-bc8e-b71ec5f4e8b&title=&width=673.6)
<a name="qhyU2"></a>
## 6.1 对极约束
P点在图像_I1_中观测的位置是_P_1，在_I_2中观测的位置是_P_2，_O1_与_O2_为相机的光心。点P与_O1_，_O2_形成的平面称为极平面。极平面与图像平面的交线称为极线，即图中的l1与l2。其中e1与e2称为极点。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683984848664-6a891899-a7c6-4798-aa71-415b09695265.png#averageHue=%23f8f8f8&height=232&id=dOZVQ&originHeight=301&originWidth=603&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=464)<br />假设_O_1相机坐标系下P点坐标为P(X,Y,Z)，归一化坐标为Pu（X1,Y1,1），则根据针孔相机投影模型(第一节内容，也可以参考本节附录第一节)，观测的像素坐标_P_1（_u__1__,v__1_）为：<br />$\tag{6.1}
\left[\begin{matrix}u_1\\v_1\\1\\\end{matrix}\right]\ =\ \left[\begin{matrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\\\end{matrix}\right]\left[\begin{matrix}X_1\\Y_1\\1\\\end{matrix}\right]$<br />化为简洁的形式如下，其中K为相机内参：<br />$\begin{equation}
\tag{6.2}
\begin{array}{c}
p_1=KP_u^1
\end{array}
\end{equation}$<br />好了，现在假设_O__1_相机相对于_O2_的运动及旋转为t与R，那么根据坐标系变换的关系，P在_O2_坐标系下坐标为：<br />$P_2=RP_1+t$<br />同样，根据相机投影模型，可以得到观测像素坐标与局部三维坐标的关系为：<br />$p_2=KP_u^2=K\left(RP_1+t\right)_u$<br />为了描述对极约束，这里需要用到投影关系，即一个坐标等比例缩放的关系，物理含义是指它们是在同一条射线上，通过投影关系（对该处推导不理解的可以看附录中第二小节推导），可以得到在相机_O1_与_O2_下，对P点观测的归一化坐标关系为：<br />$p_u^2\ast\frac{1}{s_2}=Rp_u^1\ast\frac{1}{s_1}+t$<br />在等式左右同时左乘t^，上三角符号含义为取向量的反对称矩阵，运算结果为向量的外积：<br />$t^\land\ast p_u^2\ast\frac{1}{s_2}=t^\land Rp_u^1\ast\frac{1}{s_1}+t^\land t$<br />因为相同向量，外积为0，所以上式变为：<br />$t^\land\ast p_u^2\ast\frac{1}{s_2}=t^\land Rp_u^1\ast\frac{1}{s_1}$<br />两边同时乘以_p__2_的转置：<br />$\left(p_u^2\right)^T\ast t^\land\ast p_u^2\ast\frac{1}{s_2}=\left(p_u^2\right)^{T{\ast\ t}^\land}Rp_u^1\ast\frac{1}{s_1}$<br />其中左等式，t^p2u为一个与t及p2u垂直的向量（所以对极几何t一定不能为0，不然在推导这里就不成立），既然与自身垂直，那么两个垂直向量做内积，结果为0，左侧严格等于0。则此时去掉常数项也不会影响等式成立：<br />$\left(p_u^2\right)^Tt^\land Rp_u^1=0$<br />其中_p__1__u__，p__2__u_为物体在相机坐标系下的归一化坐标，其与物体真实坐标及像素的齐次坐标关系为：<br />$P_u^1=K^{-1}u_1^齐=s_1P^1$<br />通常有如下表示：<br />$E=t^\land R$<br />称E为对极几何中的本质矩阵（Essential Matrix）,如果把物体的归一化坐标换为像素齐次坐标，则有如下结果：<br />$u_2^齐K^{-T}t∧RK^{-1}u_1^齐=0$<br />其中有如下表示：<br />$F=K^{-T}EK^{-1}$<br />F包含内参，称为对极几何中的基础矩阵（Fundamental Matrix）
<a name="MgAvE"></a>
## 6.2 本质矩阵的求解-八点法
由上可知，一对匹配点，与本质矩阵的关系可以得到一个等式：<br />$\left(p_u^2\right)^TEp_u^1=0$<br />其中E矩阵为3x3矩阵，有9个未知数，但实际上E只有5个自由度，表明其最少可以用五个点来列方程来求解，但这五个自由度是建立在非线性性质之上的，求解比较复杂。如果只考虑其尺度等价性，则E有8个自由度，这种线性性质会让求解更简单些，所以就有了常用的8点法。设E为：<br />$E\ =\ \left[\begin{matrix}e_1&e_2&e_3\\e_4&e_5&e_6\\e_7&e_8&e_9\\\end{matrix}\right]$<br />则对极约束可以写为如下形式：<br />$\left[\begin{matrix}x_2&y_2&1\\\end{matrix}\right]\left[\begin{matrix}e_1&e_2&e_3\\e_4&e_5&e_6\\e_7&e_8&e_9\\\end{matrix}\right]\left[\begin{matrix}x_1\\y_1\\1\\\end{matrix}\right]\ =\ 0$<br />把E写为向量形式：<br />$e=\left[\begin{matrix}e_1&e_2&e_3\\\end{matrix}\ \ \ \begin{matrix}e_4&e_5&e_6\\\end{matrix}\ \ \ \begin{matrix}e_7&e_8&e_9\\\end{matrix}\right]^T$<br />则上式方程为：<br />$\left[\begin{matrix}x_2x_1&x_2y_1&x_2\\\end{matrix}\ \ \ \begin{matrix}y_2x_1&y_2y_1&y_2\\\end{matrix}\ \ \ \begin{matrix}x_1&y_1&1\\\end{matrix}\right]\ast e\ =\ 0$<br />使用8对匹配点，每一对匹配点构成上述的方程，那么就有8组方程，最后8组方程构成一个线性齐次方程组，这种将本质矩阵看做向量，然后通过求解线性方程组来获得矩阵的方式，也称为直接线性变换法（DLT）。如下：<br />$\left(\begin{array}{ccccccccc}
x_{2}^{1} x_{1}^{1} & x_{2}^{1} y_{1}^{1} & x_{2}^{1} & y_{2}^{1} x_{1}^{1} & y_{2}^{1} y_{1}^{1} & y_{2}^{1} & x_{1}^{1} & y_{1}^{1} & 1 \\
x_{2}^{2} x_{1}^{2} & x_{2}^{2} y_{1}^{2} & x_{2}^{2} & y_{2}^{2} x_{1}^{2} & y_{2}^{2} y_{1}^{2} & y_{2}^{2} & x_{1}^{2} & y_{1}^{2} & 1 \\
& ...             \\\
x_{2}^{8} x_{1}^{8} & x_{2}^{8} y_{1}^{8} & x_{2}^{8} & y_{2}^{8} x_{1}^{8} & y_{2}^{8} y_{1}^{8} & y_{2}^{8} & x_{1}^{8} & y_{1}^{8} & 1
\end{array}\right) e=0$<br />根据线性方程解的情况，左侧系数矩阵为8x9的矩阵，e一定存在非零解。求解该方程，就可以得到本质矩阵E的每个元素了。

<a name="udnlP"></a>
## 6.3 从本质矩阵恢复相机运动
在得到本质矩阵E之后，还需要从E中恢复相机的运动R与t。此时需要用到奇异值分解（SVD）,假设E的SVD为：<br />$E=U\sum V^T$<br />其中U与V为正交阵，∑为奇异值矩阵。有如下性质：∑ = diag(σ,σ,0)。于是可以配凑出t与R<br />$\begin{array}{c}
t_{1}^{\wedge}=U R_{z}\left(\frac{\pi}{2}\right) \sum V^{\mathrm{T}}, R_{1}=U R_{z}^{T}\left(\frac{\pi}{2}\right) V^{T} \\
t_{2}^{\wedge}=U R_{z}\left(-\frac{\pi}{2}\right) \sum V^{\mathrm{T}}, R_{2}=U R_{z}^{T}\left(-\frac{\pi}{2}\right) V^{T}
\end{array}$<br />其中：<br />$R_z\left(\frac{\pi}{2}\right)=\ \left[\begin{matrix}0&-1&0\\1&0&0\\0&0&1\\\end{matrix}\right],R_z\left(-\frac{\pi}{2}\right)=\ \left[-\begin{matrix}0&1&0\\1&0&0\\0&0&1\\\end{matrix}\right]$<br />由于E与-E的等价，这里t取负号也是成立的，所以一共有四组解：<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683984848931-e37a1257-3050-4379-bcbe-d7a6060c5e8c.png#averageHue=%23fdfdfd&id=f6kTT&originHeight=318&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />其中只有第一种情况，P点在两个相机下具有正的深度，所以只要把任意一点求解出深度，在两个相机坐标系下深度都为正，就可以得到真实解了。

<a name="uCxjZ"></a>
## 6.4 单应矩阵
多视图几何中，除了本质矩阵和基础矩阵，还存在另一种常见的矩阵：单应矩阵(Homography)H，它描述了两个平面之间的映射关系。若场景中的特征点都落在同一平面上（比如墙、地面等），则可以通过单应性进行运动估计。这种情况在无人机携带的俯视相机或扫地机携带的顶视相机中比较常见。<br />单应矩阵通常描述处于共同平面上的一些点在两张图像之间的变换关系。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685264330326-0fc3f5e9-58dc-4936-aa11-f6aa196213a6.png#averageHue=%23f7f7f4&clientId=ud185277d-3a15-4&from=paste&height=250&id=u642c8eee&originHeight=434&originWidth=574&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=162489&status=done&style=none&taskId=u80520f70-d217-4a5e-9df6-d2f8f1544c6&title=&width=331.20001220703125)<br />单应性在SLAM中具有重要意义。当特征点共面或者相机发生纯旋转时，基础矩阵的自由度下降，这就出现了所谓的退化(degenerate)。现实中的数据总包含一些噪声，这时如果继续使用八点法求解基础矩阵，基础矩阵多余出来的自由度将会主要由噪声决定。为了能够避免退化现象造成的影响，通常我们会同时估计基础矩阵F和单应矩阵H,选择重投影误差比较小的那个作为最终的运动估计矩阵。
<a name="Vr3Bb"></a>
## 6.5 单应矩阵的求解
如果特征点都在一个平面上，即点P满足：<br />$n^TP+d=0$<br />那么有：<br />$-\frac{n^TP}{d}\ =\ 1$<br />这里的平面，指在_O1_坐标系下的平面，借助这个平面模型，按照推导基础矩阵约束过程类似，在_O2_中对P点观测的齐次坐标为：<br />$p_2=s_2K\left(RP_1+t\right)$<br />$=s_2K\left(RP_1+t\left(-\frac{n^TP_1}{d}\right)\right)$<br />$=s_2K\left(R-\frac{tn^T}{d}\right)P_1$<br />$=s_2K\left(R-\frac{tn^T}{d}\right){\frac{1}{s_1}K}^{-1}p_1$<br />使用相机内参进行坐标转换时，如果只有内参K，那么点的坐标为物体归一化坐标到像素坐标，如果带深度或者比例系数s，则为物体实际坐标到像素坐标转换。记p2与p1之间的转换矩阵为H，则有<br />$p_2=Hp_1$<br />即：<br />$\left(\begin{matrix}x_2\\y_2\\1\\\end{matrix}\right)\ =\ \left(\begin{matrix}h_1&h_2&h_3\\h_4&h_5&h_6\\h_7&h_8&h_9\\\end{matrix}\right)\left(\begin{matrix}x_1\\y_1\\1\\\end{matrix}\right)$<br />由于p2与坐标转换后的p1在同一条射线上，所以等式右边乘以任意非零常数仍然成立。这里可以通过系数调整，使h9为1，于是上述方程，可以整理得如下等式：<br />$\begin{array}{l}
u_{2}=\frac{h_{1} u_{1}+h_{2} v_{1}+h_{3}}{h_{7} u_{1}+h_{8} v_{1}+h_{9}} \\
v_{2}=\frac{h_{4} u_{1}+h_{5} v_{1}+h_{6}}{h_{7} u_{1}+h_{8} v_{1}+h_{9}}
\end{array}$<br />$\begin{array}{l}
h_{1} u_{1}+h_{2} v_{1}+h_{3}-h_{7} u_{1} u_{2}-h_{8} v_{1} u_{2}=u_{2} \\
h_{4} u_{1}+h_{5} v_{1}+h_{6}-h_{7} u_{1} v_{2}-h_{8} v_{1} v_{2}=v_{2}
\end{array}$<br />这样一对匹配点，就可以获得两个方程，当有4对匹配点时，则可以得到如下方程组：<br />$\left(\begin{array}{cccccccc}
u_{1}^{1} & v_{1}^{1} & 1 & 0 & 0 & 0 & -u_{1}^{1} u_{2}^{1} & -v_{1}^{1} u_{2}^{1} \\
0 & 0 & 0 & u_{1}^{1} & v_{1}^{1} & 1 & -u_{1}^{1} v_{2}^{1} & -v_{1}^{1} v_{2}^{1} \\
u_{1}^{2} & v_{1}^{2} & 1 & 0 & 0 & 0 & -u_{1}^{2} u_{2}^{2} & -v_{1}^{2} u_{2}^{2} \\
0 & 0 & 0 & u_{1}^{2} & v_{1}^{2} & 1 & -u_{1}^{2} v_{2}^{2} & -v_{1}^{2} v_{2}^{2} \\
u_{1}^{3} & v_{1}^{3} & 1 & 0 & 0 & 0 & -u_{1}^{3} u_{2}^{3} & -v_{1}^{3} u_{2}^{3} \\
0 & 0 & 0 & u_{1}^{3} & v_{1}^{3} & 1 & -u_{1}^{3} v_{2}^{3} & -v_{1}^{3} v_{2}^{3} \\
u_{1}^{4} & v_{1}^{4} & 1 & 0 & 0 & 0 & -u_{1}^{4} u_{2}^{4} & -v_{1}^{4} u_{2}^{4} \\
0 & 0 & 0 & u_{1}^{4} & v_{1}^{4} & 1 & -u_{1}^{4} v_{2}^{4} & -v_{1}^{4} v_{2}^{4}
\end{array}\right)\left(\begin{array}{c}
h_{1} \\
h_{2} \\
h_{3} \\
h_{4} \\
h_{5} \\
h_{6} \\
h_{7} \\
h_{8}
\end{array}\right)=\left(\begin{array}{c}
u_{2}^{1} \\
v_{2}^{1} \\
u_{2}^{2} \\
v_{2}^{2} \\
u_{2}^{3} \\
v_{2}^{3} \\
u_{2}^{4} \\
v_{2}^{4}
\end{array}\right)$<br />求解该非齐次方程组，可以得到H矩阵的每个系数。
<a name="jcTNV"></a>
## 6.6 从单应矩阵恢复相机运动

而从H矩阵恢复相机运动，也可以通过奇异值分解的方法，即：

分解的结果会有八组解，此时会把八组解都进行验证，取重投影误差最小的一组，作为最优解。

$\begin{split}
&1.d_1=d_2>d_3,d'=\pm d_1=\pm d_2,x_1=x_2=0,x_3=\pm 1 \\
&2.d_1=d_2=d_3,d'=\pm d_1=\pm d_2 =\pm d_3 \\
&3.d_1>d_2=d_3,d'=\pm d_2=\pm d_3,x_2=x_3=0,x_1=\pm 1 \\
&4.d_1>d_2>d_3,d'=\pm d_2,x_2 = 0
\end{split}$<br />$\begin{equation}
\tag{11}
\begin{array}{l}
H=U\Lambda V^T \\
\Lambda=U^TAV=dU^TRV+(U^Tt)(V^Tn)^T \\
\left\{
\begin{array}{l}
R'=sU^TRV \\
t'=U^Tt \\
n'=V^Tn \\
d' =sd \\
s=\det U \det V
\end{array}
\right.
\end{array}
\end{equation}$<br />单应矩阵在相机发生纯旋转时，仍然可以求得旋转，而不是像本质矩阵，在发生纯旋转时，方程其实是失效的，此时求得的矩阵受噪声影响很大，而单应矩阵能更好的应对这个纯旋转问题，来恢复相机的运动。
<a name="URoZf"></a>
## 6.7 三角测量
单张图像是无法得到特征点深度的，在有两张图像后，通过本质矩阵或者单应矩阵，我们可以恢复相机之间的运动。在求得相机运动后，可以通过三角测量，来求得特征点在相机坐标系下的坐标。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1685264984193-b1781b9d-f2cc-488f-b2f0-2dfae821e442.png#averageHue=%23f8f8f8&clientId=ud185277d-3a15-4&from=paste&height=257&id=u38f27af1&originHeight=435&originWidth=816&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=39091&status=done&style=none&taskId=u35787c12-ead2-4b25-ab23-0be3299f28b&title=&width=481.79998779296875)<br />假设x1,x2是两个特征点（实际物体）的归一化坐标，那么它们满足：<br />$s_2x_2=s_1Rx_1+t$<br />其中R与t为O2下O1的位姿。上式两边，同时乘以x2^，可得：<br />$s_2x_2^\land x_2=s_1x_2^\land Rx_1+x_2^\land t$<br />显然左式等于0，于是有：<br />$s_1x_2^\land Rx_1+x_2^\land t=\ 0$<br />该式中只有s1一个未知数，可以很方便得求出p1的深度。有了p1深度，p2的深度s2也很容易求出了。实际中由于噪声的存在，R,t不一定能使方程准确的等于0，更常见的做法是通过二小二乘的方式求得点的坐标，而不是直接求解。
<a name="rXxG6"></a>
## 本章小结
对极几何是求解特征点2D-2D关联时的算法，分为求本质矩阵与单应矩阵。求解完之后，通过BA可以得到特征点的三维坐标，后面的关联就变为3D-2D甚至是3D-3D的数据关联，如何求这两种关联下的相机运动，则在下一节来介绍。
<a name="YiAjq"></a>
## 本章思考
1.本质矩阵的自由度为多少？<br />2.直接法求本质矩阵的过程涉及求解齐次线性方程，而对于齐次线性方程的解，要么只有零解，要么有无穷多个解，这里取哪一个解呢？
<a name="v2Drz"></a>
## 附录
<a name="e1l3a"></a>
### 1.相机成像模型
根据针孔相机成像原理，相机坐标系下P(X,Y,Z)点投影在像平面上坐标为：<br />$\left[\begin{matrix}x\\y\\f\\\end{matrix}\right]\ =\frac{1}{Z}\ \left[\begin{matrix}f_x&0&0\\0&f_y&0\\0&0&f\\\end{matrix}\right]\left[\begin{matrix}X\\Y\\Z\\\end{matrix}\right]$<br />但是一般会省略这个焦距，让其变为1（大多说这里是使用齐次坐标，个人感觉这说不通，齐次坐标是为了矩阵运算的维度相等，加的一个1，但这个是直接替换。个人认为是二维平面上第三个坐标本来没有意义，所以直接省略了）。<br />然后，转为像素坐标，即在OpenCV中处理的格式：<br />$\left[\begin{matrix}u\\v\\1\\\end{matrix}\right]\ =\frac{1}{Z}\ \left[\begin{matrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\\\end{matrix}\right]\left[\begin{matrix}X\\Y\\Z\\\end{matrix}\right]$<br />最后使用归一化坐标，化为最简形式。就是使用P点的投影在距离光心，深度为1m的地方的坐标，来进行像素坐标计算：<br />$\left[\begin{matrix}u\\v\\1\\\end{matrix}\right]\ =\ \left[\begin{matrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\\\end{matrix}\right]\left[\begin{matrix}X\\Y\\1\\\end{matrix}\right]$

<a name="mFmRO"></a>
### 2.坐标系中投影关系的数学描述
引用高翔博士的视觉SLAM十四讲中的表述：
> 有时，我们会使用齐次坐标表示像素点。在使用齐次坐标时，一个向量将等于它自身乘上任
> 意的非零常数。这通常用于表达一个投影关系。例如，s1p1和p1成投影关系，它们在齐次坐标
> 的意义下是相等的。我们称这种相等关系为尺度意义下相等(equal up to a scale),记作：
> $sp\ \cong\ p$

个人更喜欢直接使用数值等号来描述，感觉更不容易混淆。那么在坐标系中，两个向量在同一直线上的投影描述就为：<br />$p_1=sP_2$<br />P点的归一化坐标与相机_O1_坐标系下坐标P1关系为：<br />$P_u^1=K^{-1}u_1^齐=s_1P^1$<br />可得：<br />$P^1=p_u^1\ast\frac{1}{s_1}$<br />即P点在相机_O1_坐标系下的归一化坐标，等于像素坐标的齐次转换，同时也等于P在_O1_坐标系下缩放一个系数（由于深度未知）<br />对于相机_O2_有：<br />$P_2=RP_1+t\ =\ Rp_u^1\ast\frac{1}{s_1}+t$<br />对于相机O2下P点的观测，同样的，投影关系：<br />$P^2=p_u^2\ast\frac{1}{s_2}$<br />所以有：<br />$p_u^2\ast\frac{1}{s_2}=Rp_u^1\ast\frac{1}{s_1}+t$
<a name="ZfwyV"></a>
### 3.本质矩阵性质分析
根据这个等式，可以构建一个E与归一化坐标的方程，E为3x3的矩阵，共有9个量，但实际中，由于t与R各有三个自由度，而t具有尺度等价性，会少一个自由度，所以E的自由度为5，也就是最少可以用5对配对点来求E。

<a name="LPvqg"></a>
## 参考
1.视觉SLAM十四讲<br />[2.对极几何](https://zh.wikipedia.org/wiki/%E5%AF%B9%E6%9E%81%E5%87%A0%E4%BD%95#)<br />[3.立体视觉中的对极几何——如何更好更快地寻找对应点](https://zhuanlan.zhihu.com/p/158703394)
