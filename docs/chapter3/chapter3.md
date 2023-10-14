# **3.描述状态不简单：三维空间刚体运动**

知识就是力量。——培根

---

> **本章主要内容** \
> 1.世界坐标系与里程计坐标系  \
> 2.三维刚体运动 \
> 3.旋转的参数化

三维空间刚体的运动，简单来说由平移与旋转构成。平移指两个点之间的位移，三维空间中其自由度为三。旋转则是绕着三个轴各种旋转的角度，自由度也为三。平移加旋转可构成一次变换，这个变换称为欧式变换。<br />在上节中我们介绍了坐标系变换。其实三维空间的刚体运动与其有紧密的联系，可以理解为刚体运动为持续地把运动坐标系中的坐标变换到固定坐标系之中。但区别是需要加上旋转，那接下来我们就看一下这个具体过程。
<a name="bMwKN"></a>
## 3.1 世界坐标系与里程计坐标系
![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1688892099466-75bc1fdb-bcb6-4f5b-b7c2-1ed2d5367a8b.png#averageHue=%23f3f2f2&clientId=u0f99556f-3fe0-4&from=paste&height=191&id=u06d02566&originHeight=299&originWidth=918&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=16700&status=done&style=none&taskId=u74894042-b1a5-4c38-9cf6-2cb9f068c1e&title=&width=587.4000244140625)<br />世界坐标系：世界坐标系（world坐标系）一般指固定坐标系，在系统运行过程其全局坐标不发生改变。在建图时，经常会认为起点为世界坐标系原点，而本体初始旋转角度为0。<br />里程计坐标系：里程计坐标系一般指运动坐标系，在建图过程一般指传感器坐标系，视觉SLAM中指的就是相机坐标系（camera坐标系）。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1688892946337-2b46e7c4-e9b1-4ae9-8876-43e0a07e69ff.png#averageHue=%23f5f4f4&clientId=u0f99556f-3fe0-4&from=paste&height=263&id=ua88239c7&originHeight=329&originWidth=1079&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=24827&status=done&style=none&taskId=u19b433db-5e7e-4008-847c-f4f2e9890bd&title=&width=863.2)<br />与坐标系变换的区别与联系。坐标系变换中，向量A不动，在已知A在C1坐标系下的坐标，如何转换到在C2坐标系视角下，求其在C2坐标系下坐标，或者转到w坐标系，即世界坐标系下，求A在w视角下的坐标。但这个过程没有对向量A的姿态是没有进行转换的。<br />在描述刚体运动时，我们一般会求得下一个时刻，相对于上一个时刻坐标系下的平移与旋转，如上图右图所示。这相当于我们知道了c2在c1坐标系下的坐标（平移）与相对于c1的旋转（局部坐标）。这个时候，利用c1在w下的坐标，将c2的平移与旋转转换到世界坐标系下，就得到了c2时刻的全局坐标。然后对于c2，得到下一时刻c3相对于c2的平移与旋转（局部坐标），将其转换到w坐标系下，不断进行下去，就得到了以运动坐标系表征的本地在世界坐标系下的刚体运动。
<a name="Zutc5"></a>
## 3.2 欧式变换的累加
假设c1原点在w坐标系下坐标为P1，旋转为R1，c2原点在c1坐标系下坐标为△P，旋转为△R，则c2在w坐标系下的欧式变换P2与R2为：<br />P2 = R1*△P+P1<br />这一步与坐标系变换是一样的，接下来是旋转。<br />R2 = R1*△R;<br />因为△R是相对于c1坐标系下的旋转，所以是右乘。关于旋转左乘与右乘的区别的，见参考目录。<br />欧式变换为一个平移+一个旋转，为了简洁，可以使用一个矩阵表示：<br />$T = \left[\begin{array}{ll}
\boldsymbol{R} & \boldsymbol{t} \\
\boldsymbol{0}^{\mathrm{T}} & 1
\end{array}\right]$<br />那么上述变换，就可以写为：<br />$T_2 = \left[\begin{array}{ll}
\boldsymbol{R}_1 & \boldsymbol{t}_1 \\
\boldsymbol{0}^{\mathrm{T}} & 1
\end{array}\right]\left[\begin{array}{ll}
\bigtriangleup \boldsymbol{R} & \bigtriangleup \boldsymbol{t} \\
\boldsymbol{0}^{\mathrm{T}} & 1
\end{array}\right] =T_1 \bigtriangleup T^{1}_{2}$<br />可以看到，只要求得当前时刻，在上一坐标系下的位移及旋转，就可以得到当前时刻的全局欧式变换。不断累加下去，就可以得到所有时刻的欧式变换，连起来就是传感器在三维空间的刚体运动。在加上上一节的外参变换，就可以得到本体在空间中的刚体运动了。
<a name="bjNpq"></a>
## 3.3 平移与旋转的表示
<a name="W5431"></a>
### 3.3.1平移
平移的表示比较简单，就是一个三维向量t(x,y,z)，需要注意的是，平移为一个向量。
<a name="gvIv9"></a>
### 3.3.2旋转
> 特殊正交群SO(3)，也称旋转群，表示在复合作用下绕原点旋转的群。旋转是保持向量长度和相对向量方向（即旋向性）的线性变换。它在机器人学中的重要性在于表达刚体在3D空间中的旋转：一个rigid motion（刚体运动） 精确地要求刚体在运动时必须保持距离，角度和相对方向。否则，如果不能保持模长，角度或相对方向，则不能认为物体是刚性的。

旋转群通常由一组旋转矩阵表示。但是，四元数也组成了旋转群的一个很好表达。这里首先介绍SO(3)中的旋转矩阵和四元数。但这两种旋转表达缺乏几何直观，对人类交互不太优化。大数学家欧拉发明了欧拉角和轴角来表达旋转，这两者可以与SO(3)群相互转换。另外，对于SO(3)群，其没有加法，在优化中不方便对旋转变量进行调整，需要使用到其对应的李代数进行转化，这部分内容本节不做讲解，感兴趣的同学可以看参考目录4([4.第四讲：李群和李代数](https://zhuanlan.zhihu.com/p/33156814))。<br />补充阅读：<br />三维空间里表述旋转的计算方式常见的有两种：矩阵(Matrix)和四元数（Quaternion），为了防止矩阵方式存在万向节死锁（Gimbal Lock）问题，通常采用四元数来计算旋转。但在自动驾驶领域里很少这么干，因为相机是固定在车子上，只有垂直于地面的轴（一般是Z轴）才会发生360度的旋转，根本无法引发万向节问题，总不至于用户坚持在翻车的阶段仍旧保持自动驾驶这个诡异的需求。所以在自动驾驶目标检测或车道线检测里的代码里通常就是矩阵形式，SLAM因为还会用在AR和其它领域，相机不是相对固定的，所以会采用四元数。
<a name="GEGsY"></a>
#### 3.3.2.1 旋转矩阵
旋转矩阵是一个3x3的正交矩阵，用来描述一个刚体绕着某个轴旋转的情况。旋转矩阵的每一列代表了一个坐标系中的向量，因此可以通过旋转矩阵将一个向量从一个坐标系转换到另一个坐标系。<br />旋转矩阵的几何推导，在第2节中已经写过了。这部分主要从其他方面对其进行推导。<br />假设空间中有一向量a，在坐标系1中的坐标为a1，坐标系1旋转一个角度后的坐标系为2，假设在2坐标系中向量a坐标为a2，则有：<br />$a_2 = R^2_1 a_1$<br />向量在坐标系中的表示是基的线性组合，即：<br />$a = \left[\boldsymbol{e}_{1}, \boldsymbol{e}_{2}, \boldsymbol{e}_{3}\right]\left[\begin{array}{c}
a_{1} \\
a_{2} \\
a_{3}
\end{array}\right]$<br />旋转过程中，向量不变，则有：<br />$a = \left[\boldsymbol{e}_{1}, \boldsymbol{e}_{2}, \boldsymbol{e}_{3}\right]\left[\begin{array}{c}
a_{1} \\
a_{2} \\
a_{3}
\end{array}\right]=\left[\boldsymbol{e}_{1}^{\prime}, \boldsymbol{e}_{2}^{\prime}, \boldsymbol{e}_{3}^{\prime}\right]\left[\begin{array}{c}
a_{1}^{\prime} \\
a_{2}^{\prime} \\
a_{3}^{\prime}
\end{array}\right]$<br />在两边同时乘以坐标系1基的转置，则有：<br />$a = \left[\begin{array}{l}
a_{1} \\
a_{2} \\
a_{3}
\end{array}\right]=\left[\begin{array}{lll}
\boldsymbol{e}_{1}^{\mathrm{T}} \boldsymbol{e}_{1}^{\prime} & \boldsymbol{e}_{1}^{\mathrm{T}} \boldsymbol{e}_{2}^{\prime} & \boldsymbol{e}_{1}^{\mathrm{T}} \boldsymbol{e}_{3}^{\prime} \\
\boldsymbol{e}_{2}^{\mathrm{T}} \boldsymbol{e}_{1}^{\prime} & \boldsymbol{e}_{2}^{\mathrm{T}} \boldsymbol{e}_{2}^{\prime} & \boldsymbol{e}_{2}^{\mathrm{T}} \boldsymbol{e}_{3}^{\prime} \\
\boldsymbol{e}_{3}^{\mathrm{T}} \boldsymbol{e}_{1}^{\prime} & \boldsymbol{e}_{3}^{\mathrm{T}} \boldsymbol{e}_{2}^{\prime} & \boldsymbol{e}_{3}^{\mathrm{T}} \boldsymbol{e}_{3}^{\prime}
\end{array}\right]\left[\begin{array}{c}
a_{1}^{\prime} \\
a_{2}^{\prime} \\
a_{3}^{\prime}
\end{array}\right] \stackrel{\text { def }}{=} \boldsymbol{R} \boldsymbol{a}^{\prime}$<br />可以看出，旋转矩阵为两个坐标系的内积，由于基向量的长度为1，所以实际上是各基向量夹角的余弦值。所以这个矩阵也叫方向余弦矩阵(Direction Cosine Matrix)。<br />关于旋转矩阵的自由度：其一共有9个数，每个列向量模为1，一共有3个自由度。两两垂直的约束，有2个，另外遵循右手系或者左手系，有一个约束。所以旋转矩阵自由度为9 - 3 - 2 -1 = 3。即旋转矩阵自由度为3。
<a name="edtaj"></a>
#### 3.3.2.2 四元数
如果我们由两个复数A = a + b i 和 C = c + d i ，那么构造Q = A + C j并定义k ≜ i j k 为四元数空间H中的一个数。<br />$\mathrm{Q}=\mathrm{a}+\mathrm{bi}+\mathrm{cj}+\mathrm{dk} \in \mathbb{H}\\$<br />其中{ a , b , c , d } ∈ R 且{ i , j , k }是如下定义的三个虚单位数。<br />$\mathrm{i}^{2}=\mathrm{j}^{2}=\mathrm{k}^{2}=\mathrm{ijk}=-1 \\$<br />从其中我们可以推导<br />$\mathrm{ij}=-\mathrm{ji}=\mathrm{k}, \quad \mathrm{jk}=-\mathrm{kj}=\mathrm{i}, \quad \mathrm{ki}=-\mathrm{ik}=\mathrm{j}$<br />四元数：从上式可以看到在四元数定义中嵌入复数，从而可以将实数和虚数嵌入，即实数，虚数和复数确实是四元数。<br />$\mathrm{Q}=\mathrm{a} \in \mathbb{R} \subset \mathbb{H}, \quad \mathrm{Q}=\mathrm{bi} \in \mathbb{I} \subset \mathbb{H}, \quad \mathrm{Q}=\mathrm{a}+\mathrm{bi} \in \mathbb{Z} \subset \mathbb{H}\\$<br />同样，为了完整起见，我们可以在三维中定义数字H的虚子空间。我们将其称为 pure quaternions（纯四元数），并且，令H p = I m ( H ) 为纯四元数空间。<br />$\mathrm{Q}=\mathrm{bi}+\mathrm{cj}+\mathrm{dk} \in \mathbb{H}_{\mathrm{p}} \subset \mathbb{H} \text {. }\\$<br /> 	四元数表示3D空间中的旋转:<br />$\mathbf{x}^{\prime}=\mathbf{q} \otimes \mathbf{x} \otimes \mathbf{q}^{*}$<br />其中q*为q的共轭，需要注意，只有单位四元数才表示旋转。<br />旋转矩阵用9个量描述3自由度的旋转，具有冗余性。后面提到的欧拉角和旋转向量是紧凑的，但具有奇异性。事实上，我们找不到不带奇异性的三维向量描述旋转的方式。而四元数(Quaternion)，它既是紧凑的，也没有奇异性。如果说缺点，四元数不够直观，其运算稍复杂些。<br />四元数与旋转矩阵的对比：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1689994670732-f691832c-ea00-49e7-948b-d88d7c199e88.png#averageHue=%23f7f6f5&clientId=ud0c2875e-b0fd-4&from=paste&height=626&id=ubc1c0e9c&originHeight=782&originWidth=700&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=110567&status=done&style=none&taskId=u008ebcd6-d2d7-425e-aeba-b7e5a4a357c&title=&width=560)<br />实+虚的表示法对于我们的目的并不总是方便的。假设使用了代数，则四元数可以表示为标量+向量的形式。<br />$\mathrm{Q}=\mathrm{q}_{\mathrm{w}}+\mathrm{q}_{\mathrm{x}} \mathrm{i}+\mathrm{q}_{\mathrm{y}} \mathrm{j}+\mathrm{q}_{\mathrm{z}} \mathrm{k} \Leftrightarrow \mathrm{Q}=\mathrm{q}_{\mathrm{w}}+\mathbf{q}_{\mathrm{v}} \\$<br />其中qw是实部或标量，而qv = qxi + qyj + qzk = ( qx , qy , qz ) 是 虚部或向量部分。它也可以被定义为标量-向量的有序对。<br />$\mathrm{Q}=\left\langle\mathrm{q}_{\mathrm{w}}, \mathbf{q}_{\mathrm{v}}\right\rangle .$<br />我们大多数时候将一个四元素Q表达为一个四维向量q：<br />$\mathbf{q} \triangleq\left[\begin{array}{l}
q_{w} \\
q_{v}
\end{array}\right]=\left[\begin{array}{l}
q_{w} \\
q_{x} \\
q_{y} \\
q_{z}
\end{array}\right]$<br />它允许我们对涉及到四元数的操作使用矩阵代数。在特定的场合，我们可能会通过滥用符号“ =”来允许自己混合符号，典型的例子是实四元数和纯四元数。<br />$一般:  \mathbf{q}=\mathrm{q}_{\mathrm{w}}+\mathbf{q}_{\mathrm{v}}=\left[\begin{array}{l}\mathbf{q}_{\mathrm{w}} \\ \mathbf{q}_{\mathrm{v}}\end{array}\right] \in \mathbb{H} , 实四元数:  \mathbf{q}_{\mathrm{w}}=\left[\begin{array}{l}\mathbf{q}_{\mathrm{w}} \\ \mathbf{0}_{\mathrm{v}}\end{array}\right] \in \mathbb{R} , 纯四元数:  \mathbf{q}_{\mathrm{v}}=\left[\begin{array}{c}0 \\ \mathbf{q}_{\mathrm{v}}\end{array}\right] \in \mathbb{H}_{\mathrm{p}} .$<br />四元数标量与向量顺序和ijk方向的定义没有固定的形式。多种选择导致四元数有12种不同的组合。历史的发展使某些约定胜于其他约定（Chou，1992； Yazell，2009）。今天，在现有文献中，我们发现了许多四元数flavors，如Hamilton，STS【框架变换系统，俗称美国宇航局的航天飞机】，JPL【喷气推进实验室】，ISS【国际空间站】，ESA【欧洲航天局】，工程学，机器人学以及可能有更多的教派。这些形式中的大多数可能相同，其他的则不同，但是这个事实很少被明确指出，并且许多工作对上述四种选择都缺乏足够的四元数描述。<br />Hamilton和JPL是两种最常用的约定，也是记录最好的约定。表2总结了它们的特性。 JPL主要用于航空领域，而Hamilton则在其他工程领域（例如机器人技术）中更为常见。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1689995651556-4264a6d5-8596-40d6-b1b1-b7dc1d9e0d5e.png#averageHue=%23f4f2f1&clientId=ud0c2875e-b0fd-4&from=paste&height=205&id=u3864bd4c&originHeight=256&originWidth=596&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=35361&status=done&style=none&taskId=u0d98e169-420d-408b-8cd5-a5c6d46f656&title=&width=476.8)<br />本教程的四元数指Hamilton，这也是很多C++库如Eigen，ceres中使用的四元数定义。
<a name="l71c0"></a>
#### 3.3.2.3 欧拉角
无论是旋转矩阵还是四元数，它们虽然能描述旋转，但对人类来说是非常不直观的。当我们看到一个旋转矩阵或旋转向量时，很难想象出这个旋转究竟是什么样的。当它们变换时，我们也不知道物体是在向哪个方向转动。而欧拉角则提供了一种非常直观的方式来描述旋转。<br />它使用了3个分离的转角，把一个旋转分解成3次绕不同轴的旋转。而人类很容易理解绕单个轴旋转的过程。在特定领域内，欧拉角通常有统一的定义方式。你或许在航空、航模中听说过“俯仰角”“偏航角”这些词。欧拉角当中比较常用的一种，便是用“偏航-俯仰-滚转”(yaw-pitch-roll)3个角度来描述一个旋转。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1690014993100-26c84fd9-605c-4679-bd0f-9150176fadc6.png#averageHue=%23f8f8f8&clientId=ud0c2875e-b0fd-4&from=paste&height=235&id=u77b38fbe&originHeight=294&originWidth=912&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=84982&status=done&style=none&taskId=ufcb292e1-9e32-4acb-b311-88b2afec860&title=&width=729.6)<br />欧拉角可以细分为欧拉角(Euler-angles)和泰特布莱恩角(Tait-Bryan-angles)。欧拉角的旋转轴选取顺序有(x,y,x),(x,z,x),(y,x,y),(y,z,y),(z,x,z),(z,y,z)这6种，选取顺序是a,b,a。泰特布莱恩角的旋转轴选取有(x,y,z),(x,z,y),(y,x,z),(y,z,x),(z,x,y),(z,y,x)这6种，也就是遍历笛卡尔坐标系的三轴，比如我们最常见到的Roll-Pitch-Yw角就是其中(x,y,z)的情况。<br />欧拉角是在空间中用最直观的方式和最少的参数表示任意方向的通用方法，用它们表示方向没有计算要求和容量需求的区别。图中演示的就是用(z,x,z')方法表示方向的过程。分别绕着原坐标z轴（蓝），一次旋转以后的x轴（绿）以及两次旋转以后的z轴（红）旋转，最终产生的红色坐标系即表示出目标方向。<br />![](https://cdn.nlark.com/yuque/0/2023/webp/1782698/1689557362029-c89d9485-5d12-4191-95d6-4640a55e2d56.webp#clientId=u96d715ce-fc82-4&from=paste&height=267&id=bcNxp&originHeight=348&originWidth=368&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u18bca8d7-dd7e-43f9-9741-64ed890b59c&title=&width=282)<br />欧拉角对应的旋转矩阵计算方式很简单，就是把三个连续的旋转对应的旋转矩阵右乘在一起，就得到最终的旋转矩阵M。<br />$\begin{array}{l}
M=\operatorname{Rot}(z, \gamma) \cdot \operatorname{Rot}(x, \beta) \cdot \operatorname{Rot}(z, \alpha) \\
=\left(\begin{array}{ccc}
\cos \gamma & -\sin \gamma & 0 \\
\sin \gamma & \cos \gamma & 0 \\
0 & 0 & 1
\end{array}\right)\left(\begin{array}{ccc}
1 & 0 & 0 \\
0 & \cos \beta & -\sin \beta \\
0 & \sin \beta & \cos \beta
\end{array}\right)\left(\begin{array}{ccc}
\cos \alpha & -\sin \alpha & 0 \\
\sin \alpha & \cos \alpha & 0 \\
0 & 0 & 1
\end{array}\right) \\
M=\left(\begin{array}{ccc}
\cos \alpha \cos \gamma-\sin \alpha \cos \beta \sin \gamma & -\sin \alpha \cos \gamma-\cos \alpha \cos \beta \sin \gamma & \sin \beta \sin \gamma \\
\cos \alpha \sin \gamma+\sin \alpha \cos \beta \cos \gamma & -\sin \alpha \sin \gamma+\cos \alpha \cos \beta \cos \gamma & -\sin \beta \cos \gamma \\
\sin \alpha \sin \beta & \cos \alpha \sin \beta & \cos \beta
\end{array}\right)
\end{array}$<br />当然，这是欧拉角 (z,x,z) 的情况，大家如果感兴趣，可以去画画看其他的欧拉角和泰特布莱恩角，也可以试着去推一推公式，我们可以看出，无论哪种表示方式，记录这样一个变换，至少需要三个角的sine和cos值，也就是一共存储6个单位数据。<br />欧拉角有两个问题，一个是非一一对应的问题，另外一个是存在万向锁问题。<br />首先是非一一对应问题。用欧拉角表示方向（或者说，方向变换）只需要用到三个参数，即三个旋转角度（因为坐标轴是旋转轴，所以不用增加特别的参数描述旋转轴），这样做有一个非常大的优点，就是表述清晰易懂。但随之而来的有一个很重要的问题，就是Ambiguity（歧义性），简单来说就是，当给定了欧拉角以后，我们很容易找到欧拉角表述的方向，但是当我们获得了一个方向以后，却不一定能反推回目标欧拉角，原因很简单，三维空间中的任意一个方向都可以通过至少两种不同欧拉角表示。<br />第二个问题是万向锁，如果第一次旋转角度为90°的话，会出现旋转丢掉一个自由度的现象，这样在丢失的自由度上就永远无法得到调节，即在丢失自由度上被锁死。具体细节可看参考目录3（[3.欧拉角万向节死锁](https://zhuanlan.zhihu.com/p/344050856)）。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1690015075569-2c35673d-adb9-463a-a783-72230a0061d5.png#averageHue=%23f6f6f6&clientId=ud0c2875e-b0fd-4&from=paste&height=199&id=ue4da259f&originHeight=249&originWidth=878&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=83074&status=done&style=none&taskId=u1ca5e646-c132-4228-b032-1a5ccfb8002&title=&width=702.4)
<a name="tXMkc"></a>
#### 3.3.2.4 旋转向量
回到开头部分，矩阵表示旋转至少有以下两个缺点：<br />1.S0(3)的旋转矩阵有9个量，但一次旋转只有3个自由度。因此这种表达方式是冗余的。同理，变换矩阵用16个量表达了6自由度的变换。那么，是否有更紧凑的表示呢？<br />2.旋转矩阵自身带有约束：它必须是个正交矩阵，且行列式为1。变换矩阵也是如此。当想估计或优化一个旋转矩阵或变换矩阵时，这些约束会使得求解变得更困难。<br />因此，我们希望有一种方式能够紧凑地描述旋转和平移。例如，用一个三维向量表达旋转，用一个六维向量表达变换，可行吗？事实上，任意旋转都可以用一个旋转轴和一个旋转角来刻画。于是，我们可以使用一个向量，其方向与旋转轴一致，而长度等于旋转角。这种向量称为旋转向量(或轴角/角轴，Axis-Angle),只需一个三维向量即可描述旋转。同样，对于变换矩阵，我们使用一个旋转向量和一个平移向量即可表达一次变换。这时的变量维数正好是六维。<br />任意旋转都可以用一个旋转轴和一个旋转角来刻画。对应于一个三维向量，其方向与旋转轴一致，而长度等于旋转角。这种向量称为旋转向量(或轴角/角轴，Axis-Angle)，该三维向量即可描述旋转。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1688917447482-8586dff2-8b5b-4f05-9c9d-79e84a7f68e9.png#averageHue=%23f5f5f5&clientId=u7e48237d-7f4e-4&from=paste&height=226&id=ucbfb5f8f&originHeight=385&originWidth=837&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=36698&status=done&style=none&taskId=u2f998267-c7d5-4bd6-8b94-04619948116&title=&width=490.6000061035156)<br />如图所示，u为单位向量，为旋转的轴，该次旋转一共旋转了Φ°，则轴角为Φn。从轴角到旋转矩阵，可以用罗德里格斯公式来转换：<br />$\begin{array}{l}
\boldsymbol{R}=\cos \theta \boldsymbol{I}+(1-\cos \theta) \boldsymbol{n} \boldsymbol{n}^{\mathrm{T}}+\sin \theta \boldsymbol{n}^{\wedge}\\
\end{array}$<br />符号^是向量到反对称矩阵的转换符。反之，我们也可以计算从一个旋转矩阵到旋转向量的转换。对于转角0，取两边的迹，有<br />$\begin{aligned}
\operatorname{tr}(\boldsymbol{R}) & =\cos \theta \operatorname{tr}(\boldsymbol{I})+(1-\cos \theta) \operatorname{tr}\left(\boldsymbol{n} \boldsymbol{n}^{\mathrm{T}}\right)+\sin \theta \operatorname{tr}\left(\boldsymbol{n}^{\wedge}\right) \\
& =3 \cos \theta+(1-\cos \theta) \\
& =1+2 \cos \theta
\end{aligned}\\$<br />所以有：<br />$\theta=\arccos \frac{\operatorname{tr}(\boldsymbol{R})-1}{2}$
<a name="hT8tM"></a>
## 本章小结
本节主要介绍三维空间中的刚体运动如何描述，及旋转的四种描述形式。可以看到，对于同一种量，会有不同描述形式，这个过程为参数化过程，而不同参数化形式对问题会有不同的求解性能。四种旋转的描述可以相互间转换，转换详情见参考目录6（[6.刚体运动中的坐标变换-旋转矩阵、旋转向量、欧拉角及四元数](https://blog.csdn.net/hu_hao/article/details/117197727?spm=1001.2014.3001.5502)），感兴趣的同学可以自行查阅。

<a name="Qk86d"></a>
## 本章思考
1.验证四元数旋转某个点后，结果是一个虚四元数（实部为零），所以仍然对应到一个三维<br />空间点。<br />2.一般线性方程Ac=b有哪几种做法？你能在Eigen中实现吗？

<a name="L4k4k"></a>
## 本章练习-以H3.6M三维人体姿态数据集为例进行说明
Hunman3.6M是一个用于 3D 人体位姿估计研究的大型公开数据集，是目前基于多视图的 3D 人体位姿研究最为重要的一个数据集。该数据集由 4 台数码相机收集的 360 万个不同的人体姿势组成，因此得名H3.6M。在 [paperswithcode](https://paperswithcode.com/sota/3d-human-pose-estimation-on-human36m) 网站中可以看到在此数据集上提出的各类 [SOTA](https://so.csdn.net/so/search?q=SOTA&spm=1001.2101.3001.7020) 算法及模型。<br />选取这个数据集的原因：**相机内参、外参均已给定**，可以通过旋转矩阵和平移向量的修改来改变数据。这样有两个用处：1、可以以此测试算法的鲁棒性。2、可以简单实现数据的扩增来提升算法精度。<br />回顾一下相机内参外参：

1. 从相机坐标系转换到像素坐标系中，相机内参的作用。
2. 从世界坐标系转换到相机坐标系中，相机外参的作用。

以下是H3.6M数据集给出的内外参代码段，在这段代码中读者可以看到所学内容是如何在代码中呈现出来的：

```python
h36m_cameras_intrinsic_params = [
    {
        'id': '54138969',
        'center': [512.54150390625, 515.4514770507812],
        'focal_length': [1145.0494384765625, 1143.7811279296875],
        'radial_distortion': [-0.20709891617298126, 0.24777518212795258, -0.0030751503072679043],
        'tangential_distortion': [-0.0009756988729350269, -0.00142447161488235],
        'res_w': 1000,
        'res_h': 1002,
        'azimuth': 70,  
    },
    {
        'id': '55011271',
        'center': [508.8486328125, 508.0649108886719],
        'focal_length': [1149.6756591796875, 1147.5916748046875],
        'radial_distortion': [-0.1942136287689209, 0.2404085397720337, 0.006819975562393665],
        'tangential_distortion': [-0.0016190266469493508, -0.0027408944442868233],
        'res_w': 1000,
        'res_h': 1000,
        'azimuth': -70, 
    },
    {
        'id': '58860488',
        'center': [519.8158569335938, 501.40264892578125],
        'focal_length': [1149.1407470703125, 1148.7989501953125],
        'radial_distortion': [-0.2083381861448288, 0.25548800826072693, -0.0024604974314570427],
        'tangential_distortion': [0.0014843869721516967, -0.0007599993259645998],
        'res_w': 1000,
        'res_h': 1000,
        'azimuth': 110, 
    },
    {
        'id': '60457274',
        'center': [514.9682006835938, 501.88201904296875],
        'focal_length': [1145.5113525390625, 1144.77392578125],
        'radial_distortion': [-0.198384091258049, 0.21832367777824402, -0.008947807364165783],
        'tangential_distortion': [-0.0005872055771760643, -0.0018133620033040643],
        'res_w': 1000,
        'res_h': 1002,
        'azimuth': -110,  
    },
]

h36m_cameras_extrinsic_params = {
    'S1': [
        {
            'orientation': [0.1407056450843811, -0.1500701755285263, -0.755240797996521, 0.6223280429840088],
            'translation': [1841.1070556640625, 4955.28466796875, 1563.4454345703125],
        },
        {
            'orientation': [0.6157187819480896, -0.764836311340332, -0.14833825826644897, 0.11794740706682205],
            'translation': [1761.278564453125, -5078.0068359375, 1606.2650146484375],
        },
        {
            'orientation': [0.14651472866535187, -0.14647851884365082, 0.7653023600578308, -0.6094175577163696],
            'translation': [-1846.7777099609375, 5215.04638671875, 1491.972412109375],
        },
        {
            'orientation': [0.5834008455276489, -0.7853162288665771, 0.14548823237419128, -0.14749594032764435],
            'translation': [-1794.7896728515625, -3722.698974609375, 1574.8927001953125],
        },
    ],
    'S2': [
        {},
        {},
        {},
        {},
    ],
    'S3': [
        {},
        {},
        {},
        {},
    ],
    'S4': [
        {},
        {},
        {},
        {},
    ],
    'S5': [
        {
            'orientation': [-0.42792025, 0.64015153, -0.5395463, 0.34055846],
            'translation': [2097.3916015625, 4880.94482421875, 1605.732421875],
        },
        {
            'orientation': [0.43592995, 0.64571192, -0.51878033, -0.35197751],
            'translation': [2031.7008056640625, -5167.93310546875, 1612.923095703125],
        },
        {
            'orientation': [0.64472645, -0.43757454, 0.32732173, -0.53452485],
            'translation': [-1620.5948486328125, 5171.65869140625, 1496.43701171875],
        },
        {
            'orientation': [0.65817815, 0.45242671, -0.30823131, -0.51682207],
            'translation': [-1637.1737060546875, -3867.3173828125, 1547.033203125],
        },
    ],
    'S6': [
        {
            'orientation': [0.1337897777557373, -0.15692396461963654, -0.7571090459823608, 0.6198879480361938],
            'translation': [1935.4517822265625, 4950.24560546875, 1618.0838623046875],
        },
        {
            'orientation': [0.6147197484970093, -0.7628812789916992, -0.16174767911434174, 0.11819244921207428],
            'translation': [1969.803955078125, -5128.73876953125, 1632.77880859375],
        },
        {
            'orientation': [0.1529948115348816, -0.13529130816459656, 0.7646096348762512, -0.6112781167030334],
            'translation': [-1769.596435546875, 5185.361328125, 1476.993408203125],
        },
        {
            'orientation': [0.5916101336479187, -0.7804774045944214, 0.12832270562648773, -0.1561593860387802],
            'translation': [-1721.668701171875, -3884.13134765625, 1540.4879150390625],
        },
    ],
    'S7': [
        {
            'orientation': [0.1435241848230362, -0.1631336808204651, -0.7548328638076782, 0.6188824772834778],
            'translation': [1974.512939453125, 4926.3544921875, 1597.8326416015625],
        },
        {
            'orientation': [0.6141672730445862, -0.7638262510299683, -0.1596645563840866, 0.1177929937839508],
            'translation': [1937.0584716796875, -5119.7900390625, 1631.5665283203125],
        },
        {
            'orientation': [0.14550060033798218, -0.12874816358089447, 0.7660516500473022, -0.6127139329910278],
            'translation': [-1741.8111572265625, 5208.24951171875, 1464.8245849609375],
        },
        {
            'orientation': [0.5912848114967346, -0.7821764349937439, 0.12445473670959473, -0.15196487307548523],
            'translation': [-1734.7105712890625, -3832.42138671875, 1548.5830078125],
        },
    ],
    'S8': [
        {
            'orientation': [0.14110587537288666, -0.15589867532253265, -0.7561917304992676, 0.619644045829773],
            'translation': [2150.65185546875, 4896.1611328125, 1611.9046630859375],
        },
        {
            'orientation': [0.6169601678848267, -0.7647668123245239, -0.14846350252628326, 0.11158157885074615],
            'translation': [2219.965576171875, -5148.453125, 1613.0440673828125],
        },
        {
            'orientation': [0.1471444070339203, -0.13377119600772858, 0.7670128345489502, -0.6100369691848755],
            'translation': [-1571.2215576171875, 5137.0185546875, 1498.1761474609375],
        },
        {
            'orientation': [0.5927824378013611, -0.7825870513916016, 0.12147816270589828, -0.14631995558738708],
            'translation': [-1476.913330078125, -3896.7412109375, 1547.97216796875],
        },
    ],
    'S9': [
        {
            'orientation': [0.15540587902069092, -0.15548215806484222, -0.7532095313072205, 0.6199594736099243],
            'translation': [2044.45849609375, 4935.1171875, 1481.2275390625],
        },
        {
            'orientation': [0.618784487247467, -0.7634735107421875, -0.14132238924503326, 0.11933968216180801],
            'translation': [1990.959716796875, -5123.810546875, 1568.8048095703125],
        },
        {
            'orientation': [0.13357827067375183, -0.1367100477218628, 0.7689454555511475, -0.6100738644599915],
            'translation': [-1670.9921875, 5211.98583984375, 1528.387939453125],
        },
        {
            'orientation': [0.5879399180412292, -0.7823407053947449, 0.1427614390850067, -0.14794869720935822],
            'translation': [-1696.04345703125, -3827.099853515625, 1591.4127197265625],
        },
    ],
    'S11': [
        {
            'orientation': [0.15232472121715546, -0.15442320704460144, -0.7547563314437866, 0.6191070079803467],
            'translation': [2098.440185546875, 4926.5546875, 1500.278564453125],
        },
        {
            'orientation': [0.6189449429512024, -0.7600917220115662, -0.15300633013248444, 0.1255258321762085],
            'translation': [2083.182373046875, -4912.1728515625, 1561.07861328125],
        },
        {
            'orientation': [0.14943228662014008, -0.15650227665901184, 0.7681233882904053, -0.6026304364204407],
            'translation': [-1609.8153076171875, 5177.3359375, 1537.896728515625],
        },
        {
            'orientation': [0.5894251465797424, -0.7818877100944519, 0.13991211354732513, -0.14715361595153809],
            'translation': [-1590.738037109375, -3854.1689453125, 1578.017578125],
        },
    ],
}
```

有了上面的信息，我们可以使用python里的numpy和scipy库进行姿态变换。首先将四元数转换为旋转矩阵，之后定义一个新的旋转矩阵并进行矩阵点乘；平移向量相加即可实现变换。

```python
import numpy as np
from scipy.spatial.transform import Rotation

# 定义姿态变换参数: 在这里定义姿态变换参数，例如旋转矩阵R或欧拉角yaw/pitch/roll等 
# 以下是一个简单的旋转矩阵示例
# 对每个人体姿势数据进行姿态变换，以单个为例
q0, qx, qy, qz = [0.5834008455276489, -0.7853162288665771, 0.14548823237419128, -0.14749594032764435]

R_input = Rotation.from_quat([qx, qy, qz, q0]).as_matrix()
R_new = np.array([[0, -1, 0],
                  [1, 0, 0],
                  [0, 0, 1]])
R_output = np.dot(R_new, R_input)
q = Rotation.from_matrix(R_output).as_quat()

# 平移向量的修改
translation = [2044.45849609375, 4935.1171875, 1481.2275390625]
translation_new = np.array([0, 0, 1])
translation += translation_new

#输出修改后的数据：
print(q)
print(translation)
```

<a name="z2MyK"></a>
## 参考
0.《视觉SLAM十四讲》<br />[1.旋转的左乘与右乘](https://zhuanlan.zhihu.com/p/128155013)<br />[2.如何通俗地解释欧拉角？之后为何要引入四元数？]()<br />[3.欧拉角万向节死锁](https://zhuanlan.zhihu.com/p/344050856)<br />[4.第四讲：李群和李代数](https://zhuanlan.zhihu.com/p/33156814)<br />[5.文章翻译—基于误差状态卡尔曼滤波器的四元数运动学—第1章](https://blog.csdn.net/sunqin_csdn/article/details/108560582)<br />[6.刚体运动中的坐标变换-旋转矩阵、旋转向量、欧拉角及四元数](https://blog.csdn.net/hu_hao/article/details/117197727?spm=1001.2014.3001.5502)<br />[7.如何通俗易懂的解释自动驾驶中的BEV和SLAM？](https://zhuanlan.zhihu.com/p/646310465)
