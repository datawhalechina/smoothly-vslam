# **7.积硅步以至千里：前端-视觉里程计之相对位姿估计**

志不强者智不达。——墨翟

---

> **本章主要内容**
> 1.已知3D-2D匹配关系，估计相机相对位姿的常用算法


PnP(Perspective-n-Point)算法是求解3D到2D点对运动的方法，它描述了当知道$n$个3D空间点及其投影位置时，如何估计相机的位姿。PnP问题有多种求解方法，例如，用3对点估计位姿的P3P、直接线性变换（DLT）、EPnP（Efficient PnP）等等。此外，还能用非线性优化的方式，构建最小二乘问题并迭代求解，光束法平差（Bundle Adjustment, BA）。
<a name="y48Fi"></a>
## 7.1 DLT法
考虑某个空间点$\!\!P\!\!$在世界坐标系下的齐次坐标为$\!\!P_w\!\!=\!\!(X_w,\!Y_w,\!Z_w,\!1)^T\!\!$，在相机坐标系下为$\!\!P_c\!=\!\!(X_c,\!Y_c,\!Z_c)^T\!\!$。在图像$\!I_1\!$中，投影到特征点$\!x_1=(u_1,v_1,1)^T\!$(以归一化平面齐次坐标表示)。定义增广矩阵$\!\![R|t]\!$为一个$\!3\times 4\!$的矩阵，包含了旋转与平移信息。世界坐标与观测的转换关系如下，像素坐标与相机坐标系的转换，深度乘左边比较好表示：<br />$s_1\left[\begin{matrix}u_1\\v_1\\1\\\end{matrix}\right]=K\left[\begin{matrix}X_c\\Y_c\\Z_c\\\end{matrix}\right]=K\left[\begin{matrix}R&t\\\end{matrix}\right]\left[\begin{matrix}X_w \\ Y_w \\Z_w \\1\\\end{matrix}\right]$<br />这里设：<br />$T=K\left[\begin{matrix}R&t\\\end{matrix}\right]=\ \left[\begin{matrix}t_1&t_2&t_3\\t_5&t_6&t_7\\t_9&t_{10}&t_{11}\\\end{matrix}\ \ \ \begin{matrix}t_4\\t_8\\t_{12}\\\end{matrix}\right]$<br />则原式子为：<br />$s_1\left[\begin{matrix}u_1\\v_1\\1\\\end{matrix}\right]=T\left[\begin{matrix}X_w \\ Y_w \\Z_w \\1\\\end{matrix}\right]$<br />如果是求相机$O_1$与相机$O_2$之间的相对关系，则这里的$\!P_w\!$就是$\!P_1\!$，而$R,t$为$O_1$在$O_2$下的姿态。<br />$s_1\left[\begin{matrix}u_1\\v_1\\1\\\end{matrix}\right]=\ \left[\begin{matrix}t_1&t_2&t_3\\t_5&t_6&t_7\\t_9&t_{10}&t_{11}\\\end{matrix}\ \ \ \begin{matrix}t_4\\t_8\\t_{12}\\\end{matrix}\right]\left[\begin{matrix}X_w \\ Y_w \\Z_w \\1\\\end{matrix}\right]$<br />简单的矩阵乘法，最后一行可以得到$s_1$的表达式，于是$\!u_1,v_1\!$的值为：<br />$\begin{array}{c}
u_{1}=\frac{t_{1} X_{W}+t_{2} Y_{W}+t_{3} Z_{W}+t_{4}}{t_{9} X_{W}+t_{10} Y_{W}+t_{11} Z_{W}+t_{12}}
\end{array}$<br />$\begin{array}{c}
v_{1}=\frac{t_{5} X_{W}+t_{6} Y_{W}+t_{7} Z_{W}+t_{8}}{t_{9} X_{W}+t_{10} Y_{W}+t_{11} Z_{W}+t_{12}}
\end{array}$<br />为简化表示，设：<br />$t_1=\ \left(t_1\ \begin{matrix}\ t_2&t_3&t_4\\\end{matrix}\right)^T$<br />$t_2=\ \left(t_5\ \begin{matrix}\ t_6&t_7&t_8\\\end{matrix}\right)^T$<br />$t_3=\ \left(t_9\ \begin{matrix}\ t_{10}&t_{11}&t_{12}\\\end{matrix}\right)^T$<br />于是：<br />$t_1^TP-t_3^TPu_1=0$<br />$t_2^TP-t_3^TPv_1=0$<br />其中$t$为待求变量，与求单应矩阵类似，每个点对可以提供两个关于$t$的约束，假设一共有$N$个特征点，则可以得到如下线性方程组：<br />$\left(\begin{array}{ccc}
\boldsymbol{P}_{1}^{T} & 0 & -u_{1} \boldsymbol{P}_{1}^{T} \\
0 & \boldsymbol{P}_{1}^{T} & -v_{1} \boldsymbol{P}_{1}^{T} \\
\ldots & \ldots & \ldots \\
\boldsymbol{P}_{N}^{T} & 0 & -u_{N} \boldsymbol{P}_{N}^{T} \\
0 & \boldsymbol{P}_{N}^{T} & -v_{N} \boldsymbol{P}_{N}^{T}
\end{array}\right)\left(\begin{array}{c}
t_{1} \\
t_{2} \\
t_{3}
\end{array}\right)=0$<br />$\!\!t\!\!$一共有12维，可以通过最少六对匹配点，实现对矩阵T的线性求解，这种方法称为直接线性法（DLT）。如果匹配点数大于6对，可以通过SVD方法对超定方程求最小二乘解。<br />求得矩阵后，需要先左乘一个内参矩阵的逆$\!\!K^{-1}\!\!$，然后获得$\!\!t\!$与$\!\!t\!\!$。<br />需要说明的是，这种方法忽略了未知数之间的关联，比如旋转$\!\!R\!\!\in\!\! SO(3)\!\!$，使用这种方式求出的R矩阵不一定满足该关系，其为一个一般性矩阵，需要找一个矩阵对其进行近视，近似的方法由QR分解完成，也可以如下计算：<br />$R\ \gets\left(RR^T\right)^{\left\{-\frac{1}{2}\right\}}R$<br />这种解法求得的解，既无尺度，也无约束，要得到尺度及约束，需要构建优化问题。

<a name="3GKpZ"></a>
## 7.2 P3P
在知道世界坐标系或者其他相机坐标系下三维点与当前相机3个点的对应关系时，最少可以通过3对配对点来求当前相机相对于三维点坐标系的旋转和平移。<br />P3P需要利用给定的$\!\!\!3\!\!\!$个点的几何关系。它的输入数据为$\!\!\!3\!\!\!$对$\!\!3\text{D}\!\!-\!\!2\text{D}\!\!$匹配点。记$\!\!3\text{D}\!\!$点为$\!\!A\!,\!B\!,\!C\!\!$,$\!\!2\text{D}\!\!$点为$\!\!a,\!b,\!c\!\!$，其中小写字母代表的点为对应大写字母代表的点在相机成像平面上的投影，如下图所示：<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683985374990-677bd69d-f083-4b07-89a2-39040c1a56db.png#averageHue=%23fafafa&id=aQ2hi&originHeight=368&originWidth=482&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />P3P算法求得的结果，是另一个坐标系中三维点（比如世界坐标系中的$\!\!A\!,\!B\!,\!C\!\!$），在当前坐标系下的坐标（比如相机成像平面的$\!\!a,\!b,\!c\!\!$对应的相机坐标系下的3D坐标）。从而把一个3D-2D的关联，转化为3D-3D的关联，把PnP问题转换为ICP问题，通过求解ICP的方式获得旋转和平移。需要注意的是，P3P算法最后方程的解有四个，需要额外的一个点来验证，一般会选取重投影误差最小的解作为最优解。<br />三维点$\!\!A\!,\!B\!,\!C\!\!$，相机成像平面点$\!\!a,\!b,\!c\!$和相机光心$\!O\!$组成的三角形有如下对应关系：<br />$\Delta O_{ab}-\Delta O_{AB},\quad\Delta O_{bc}-\Delta O_{BC},\quad\Delta O_{ac}-\Delta O_{AC}$<br />首先回顾一下三角形余弦定理，余弦定理可以看做是勾股定理的推广，表征的是三角形中的边角关系，推导余弦定理可以用勾股定理。含义为对边平方等于夹边平方和减去夹边乘积与对角余弦乘积的2倍。<br />![](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683985375303-006e318c-1b4c-4083-af5f-51dce5eb3b41.png#averageHue=%2315574a&id=P4GIu&originHeight=336&originWidth=611&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)<br />根据余弦定理，对于这个大的空间三角形，有如下关系：<br />$\begin{array}{l}
O A^{2}+O B^{2}-2 O A \cdot O B \cdot \cos \langle a, b\rangle=A B^{2} \\
O B^{2}+O C^{2}-2 O B \cdot O C \cdot \cos \langle b, c\rangle=B C^{2} \\
O A^{2}+O C^{2}-2 O A \cdot O C \cdot \cos \langle a, c\rangle=A C^{2} .
\end{array}$<br />对该方程进行消元，方法是除以$OC^2$，并设$x=OA/OC,y=OB/OC$，则方程变为：<br />$\begin{array}{l}
x^{2}+y^{2}-2 \cdot x \cdot y \cdot \cos \langle a, b\rangle =\frac{A B^{2}}{O C^{2}} \\
x^{2}+1-2 \cdot x \cdot \cos \langle a, c\rangle=\frac{A C^{2}}{O C^{2}} \\
y^{2}+1-2 \cdot y \cdot \cos \langle b, c\rangle=\frac{B C^{2}}{O C^{2}}
\end{array}$<br />然后再进行替换，令：<br />$u\ =\ \frac{AB^2}{OC^2},\ v=\frac{BC^2}{AB^2},\ w=\frac{AC^2}{AB^2}$<br />可得：<br />$\begin{split}
&x^2+y^2-2 \cdot x \cdot y \cdot cos<a,b> = u \\
&x^2 + 1 - 2 \cdot x \cdot cos<a,c>=wu \\
&y^2 + 1 - 2 \cdot y \cdot cos<b,c>=uv
\end{split}$<br />将第一个式子，带入第二第三个式子，可得：<br />$\begin{split}
&(1-w)x^2-w \cdot y^2 - 2 \cdot x \cdot cos<a,c> + 2 \cdot w \cdot x \cdot y \ cdot cos<a, b> + 1 = 0 \\
&(1-v)y^2-v \cdot x^2 - 2 \cdot y \cdot cos<b,c> + 2 \cdot v \cdot x \cdot y \ cdot cos<a, b> + 1 = 0
\end{split}$<br />因为$\!\!ABC\!\!$之间的相对位置关系是不变的，所以其中$\!v,w\!$值为已知值，而三个角度余弦值可以通过像素观测坐标求得，对于上述方程，只有$\!x,y\!$为未知数，为一个二元二次方程组，求解这个方程组，需要用到吴消元法。<br />它可以将原方程等效成一组特征列（Characteristic Serial, CS），凡是原方程组的解都会是CS的解，但是CS的解不一定是原方程的解，所以需要验证，这里的等效方程为：<br />$\begin{cases}
a_4 x^4+a_3 x^3 + a_2 x^2 + a_1 x^1 + a_0 = 0 \\
b_1 y - b_0 = 0
\end{cases}$<br />求得了$\!x,y\!$的值，就可以求取$\!\!OA,\!OB,\!OC\!\!$的值，根据下面的公式，$\!\!AB\!\!$已知，可以先求$\!\!OC\!\!$，然后分别求解$\!\!OB,OA\!$：<br />$x^2 + y^2 - 2 \cdot x \cdot y \cdot cos<a,b>=\frac{AB^2}{OC^2}\qquad y=\frac{OB}{OC},x=\frac{OA}{OC}$<br />求得三个点在相机坐标系下的深度之后，就可以通过像素坐标与三维点的投影关系，求出相机坐标系下点的坐标。<br />$\bm{A}=\vec{a} \cdot \left \| \bm{PA} \right \|$<br />附：吴消元法系数<br />$p=2 \cdot cos<a,c> \quad q = 2\cdot cos<b, c> \quad r=2\cdot cos<a,b>$<br />$\left\{\begin{aligned}
a_{4}= & a^{2}+b^{2}-2 a-2 b+2\left(1-r^{2}\right) b a+1 \\
a_{3}= & -2 q a^{2}-r p b^{2}+4 q a+(2 q+p r) b+\left(r^{2} q-2 q+r p\right) a b-2 q \\
a_{2}= & \left(2+q^{2}\right) a^{2}+\left(p^{2}+r^{2}-2\right) b^{2}-\left(4+2 q^{2}\right) a-\left(p q r+p^{2}\right) b-\left(p q r+r^{2}\right) a b+q^{2}+2 \\
a_{1}= & -2 q a^{2}-r p b^{2}+4 q a+\left(p r+q p^{2}-2 q\right) b+(r p+2 q) a b-2 q \\
a_{0}= & a^{2}+b^{2}-2 a+\left(2-p^{2}\right) b-2 a b+1 \\
b_{1}= & b\left(\left(p^{2}-p q r+r^{2}\right) a+\left(p^{2}-r^{2}\right) b-p^{2}+p q r-r^{2}\right)^{2} \\
b_{0}= & \left.\left((1-a-b) x^{2}+(a-1) q x-a+b+1\right)\right) \\
& \left(\left(r^{3}\left(a^{2}+b^{2}-2 a-2 b+\left(2-r^{2}\right) a b+1\right) x^{3}+\right.\right. \\
& r^{2}\left(p+p a^{2}-2 r q a b+2 r q b-2 r q-2 p a-2 p b+p r^{2} b+4 r q a+q r^{3} a b-2 r q a^{2}+2 p a b+p b^{2}-r^{2} p b^{2}\right) x^{2}+ \\
& \left(r^{5}\left(b^{2}-a b\right)-r^{4} p q b+r^{3}\left(q^{2}-4 a-2 q^{2} a+q^{2} a^{2}+2 a^{2}-2 b^{2}+2\right)+r^{2}\left(4 p q a-2 p q a b+2 p q b-2 p q-2 p q a^{2}\right)+\right. \\
& \left.r\left(p^{2} b^{2}-2 p^{2} b+2 p^{2} a b-2 p^{2} a+p^{2}+p^{2} a^{2}\right)\right) x+ \\
& \left(2 p r^{2}-2 r^{3} q+p^{3}-2 p^{2} q r+p q^{2} r^{2}\right) a^{2}+\left(p^{3}-2 p r^{2}\right) b^{2}+\left(4 q r^{3}-4 p r^{2}-2 p^{3}+4 p^{2} q r-2 p q^{2} r^{2}\right) a+ \\
& \left(-2 q r^{3}+p r^{4}+2 p^{2} q r-2 p^{3}\right) b+\left(2 p^{3}+2 q r^{3}-2 p^{2} q r\right) a b+ \\
& \left.p q^{2} r^{2}-2 p^{2} q r+2 p r^{2}+p^{3}-2 r^{3} q\right)
\end{aligned}\right.$

<a name="KQ6YV"></a>
## 7.3 EPnP
EPnP算法最后求解在相机坐标系下，$\!3D\!$点在控制点下坐标时，复杂度为$\!\!O(n)\!\!$，其中$\!\!n\!\!$为配对点对数，比较高效，其名称也是如此得来的。其求解的思路为，先通过世界坐标系或者上一坐标系下3D点的坐标，计算出四个控制点坐标。基于控制点坐标，计算出3D点在控制点坐标系下坐标，而可以证明，在世界坐标系下控制点坐标，与在此时相机坐标系下控制点坐标是相同的。于是问题转为求此时相机坐标系下控制点坐标，此时先假设四个局部坐标系下控制点坐标，通过控制点坐标与观测坐标对应关系，即针孔相机投影，每一对匹配点，可以得到两个方程。把所有配对点都联立方程，得到一个12个未知数，$\!2n\!$个方程的齐次方程组，求解这个方程，方法为求矩阵零特征值的特征向量。由于方程组解系中非共线解向量数量和矩阵的秩与12的差值有关，这里只考虑有1到4个非共线解向量的情况。求得解后，再通过世界坐标系下控制点坐标与局部坐标系下控制点坐标具有相同的相对位置关系，得到解系中系数，从而得到相机坐标系下控制点坐标。这样就把3D-2D的关联，转为3D-3D的关联，最后使用带匹配关系的ICP算法，得到两个坐标系下的R和t。这时候会有四组解，取其中重投影误差最小的一组解为最后的解。<br />其中求解解系中系数时，可以用高斯牛顿法建立世界坐标系下控制点相对坐标与局部坐标系下相对坐标之差，对系数进一步优化。<br />所以这些算法，都是线性代数，非线性优化。<br />下面对该算法进行介绍：<br />与其他方法相比，EPnP方法的复杂度为O(n)，对于点对数量较多的PnP问题，非常高效。<br />核心思想是将3D点表示为4个控制点的组合，优化也只针对4个控制点，所以速度很快，在求解 Mx=0时，最多考虑了4个奇异向量，因此精度也很高。<br />第一步，选择世界坐标系下控制点坐标：<br />选择3D点的质心，作为第一个控制点，<br />$\bm{c}_1^w=\frac{1}{n}\sum_{i=1}^{n} \bm{P}_i^w$<br />对n个3D点进行去重心操作：<br />$\bm{A}=\begin{bmatrix}
(\bm{P}_1^w)^T-(\bm{c}_1^w)^t \\
\cdots \\
(\bm{P}_n^w)^T-(\bm{c}_1^w)^t
\end{bmatrix}$<br />计算A的协方差矩阵，并求其特征值与对应的特征向量，剩余的三个控制点，由这三个特征值与特征向量构成：<br />$\begin{split}
&\bm{c}_2^{w}=\bm{c}_1^{w}+\sqrt{\lambda_1}\bm{V}_1 \\
&\bm{c}_3^{w}=\bm{c}_1^{w}+\sqrt{\lambda_2}\bm{V}_2 \\
&\bm{c}_4^{w}=\bm{c}_1^{w}+\sqrt{\lambda_3}\bm{V}_3 \\
\end{split}$<br />第二步，求解齐次重心坐标，也就是在上面的控制点坐标系下的坐标：<br />$\bm{P}_{i}^{w}=\sum_{j=1}^{4}\alpha_{ij}\bm{c}_{j}^{w},\quad with \, \sum_{j=1}^{4}\alpha_{ij}=1$<br />建立3D点坐标与控制点坐标关系：<br />$\begin{bmatrix}
\bm{P}_{i}^{w} \\
1
\end{bmatrix} = \bm{C}
\begin{bmatrix}
\alpha_{i1} \\
\alpha_{i2} \\
\alpha_{i3} \\
\alpha_{i4} \\
\end{bmatrix} = 
\begin{bmatrix}
\bm{c}_1^{w} & \bm{c}_2^{w} & \bm{c}_3^{w} & \bm{c}_4^{w} \\
1 & 1 & 1 & 1
\end{bmatrix}
\begin{bmatrix}
\alpha_{i1} \\
\alpha_{i2} \\
\alpha_{i3} \\
\alpha_{i4}
\end{bmatrix}$<br />及：<br />$\begin{bmatrix}
\alpha_{i1} \\
\alpha_{i2} \\
\alpha_{i3} \\
\alpha_{i4}
\end{bmatrix}_{4\times1} = 
\begin{bmatrix}
\bm{c}_1^{w} & \bm{c}_2^{w} & \bm{c}_3^{w} & \bm{c}_4^{w} \\
1 & 1 & 1 & 1
\end{bmatrix}_{4\times 4}^{-1}
\begin{bmatrix}
\bm{P}_i^w \\
1
\end{bmatrix}_{4\times1} = 
\bm{C}^{-1}
\begin{bmatrix}
\bm{P}_i^w \\
1
\end{bmatrix}$

对于世界坐标系与当前相机坐标系下观测点之间转换关系，有：<br />$\mathbf{c}_{j}^{c}=\left[\begin{array}{ll}
\mathbf{R} & \mathbf{t}
\end{array}\right]\left[\begin{array}{c}
\mathbf{c}_{j}^{w} \\
1
\end{array}\right] \\$<br />$\mathbf{p}_{i}^{c}=\left[\begin{array}{ll}
\mathbf{R} & \mathbf{t}
\end{array}\right]\left[\begin{array}{c}
\mathbf{p}_{i}^{w} \\
1
\end{array}\right]=\left[\begin{array}{ll}
\mathbf{R} & \mathbf{t}
\end{array}\right]\left[\begin{array}{c}
\sum_{j=1}^{4} \alpha_{i j} \mathbf{c}_{j}^{w} \\
1
\end{array}\right] \\$<br />$\mathbf{p}_{i}^{c}=\left[\begin{array}{ll}
\mathbf{R} & \mathbf{t}
\end{array}\right]\left[\begin{array}{c}
\sum_{j=1}^{4} \alpha_{i j} \mathbf{c}_{j}^{w} \\
\sum_{j=1}^{4} \alpha_{i j}
\end{array}\right]=\sum_{j=1}^{4} \alpha_{i j}\left[\begin{array}{ll}
\mathbf{R} & \mathbf{t}
\end{array}\right]\left[\begin{array}{c}
\mathbf{c}_{j}^{w} \\
1
\end{array}\right]=\sum_{j=1}^{4} \alpha_{i j} \mathbf{c}_{j}^{c}$<br />即，相机坐标系下控制点也是由世界坐标系下控制点刚体转换而来的，而控制点的系数不变。

第三步，在相机坐标系下，使用控制点坐标系数，建立观测方程：<br />$w_i
\begin{bmatrix}
u_i \\
v_i \\
1
\end{bmatrix} = 
\begin{bmatrix}
f_u & 0 & u_c \\
0 & f_v & v_c \\
0 & 0 & 1
\end{bmatrix}
\sum_{j=1}^{4}\alpha_{ij}
\begin{bmatrix}
x_j^{c} \\
y_j^c \\
z_j^c
\end{bmatrix}$<br />从而可以得到两个线性方程：<br />$\$$\begin{aligned}
&\sum_{j=1}^{4}\alpha_{ij}f_ux_j^c + \alpha_{ij}(u_c-u_i)z_j^c=0 \\
&\sum_{j=1}^{4}\alpha_{ij}f_vy_j^c+\alpha_{ij}(v_c-v_j)z_j^c=0
\end{aligned}$<br />把所有n 个点串联起来，我们可以得到一个线性方程组：

上式中，齐次重心坐标，相机内参数和2D像点坐标都是已知量，未知量是4个控制点在相机坐标系下的坐标。共12个位置参数，一个像点可以列2个方程，n 个像点可以列出2n 个方程。<br />其中$\text{x}=[c_1^{c^T},c_2^{c^T},c_3^{c^T},c_4^{c^T}]$,$\text{x}$就是控制点在摄像头坐标系下的坐标，显然这是一个$12 \times 1$的向量，且$\text{x}$在$M$的右零空间中，或者$\text{x}\in ker(M)$:<br />$\text{x}=\sum_{i=1}^{N}\beta_i \text{v}_i$<br />上式中，$\text{v}_i$是$M$的$N$个零特征值对应的$N$特征向量。对于第$i$个控制点：<br />$\bm{c}_i^c=\sum_{k=1}^N \beta_k \text{v}_k^{[i]}$<br />求得特征向量后，还需要求解系数β，该步通过世界坐标系下控制点相对坐标与局部坐标系下控制点相对坐标一样来解决：<br />根据仿真发现$M^TM$为0的特征值的个数喝摄像头的焦距有关（注意：是和焦距有关，而不是和参考点的位置有关！），EPnP算法建议只考虑$N=1,2,3,4$的情况。摄像头的外参描述的只是坐标变换，不会改变控制点之间的距离，$\left \| \bm{c}_i^c - \bm{c}_j^c \right \|^2=\left \| \bm{c}_i^w - \bm{c}_j^w \right \|^2$,<br />$\left \|
\sum_{k=1}^{N}\beta_k \bm{\text{v}}_k^{[i]}-\sum_{k=1}^{N}\beta_k \bm{\text{v}}_k^{[j]}
\right\|^2 = 
\left \|
\bm{c}_i^w - \bm{c}_j^w
\right \|^2$<br />这是一个关于$\{\beta_k\}_{k=1,\cdots,N}$的二次方程，这个方程的特点是没有关于$\{\beta\}_{k=1,\cdots,N}$的一次项。如果将这些二次项$\beta_i\beta_j$变量替换为$\beta_{ij}$,那么该方程是$\{\beta_{ij}\}_{i,j=1,\cdots,N}$的线性方程了。对于4个控制点，可以得到$C_{4}^{2}=6$个这样的方程。<br />如果不挖掘任何附加条件，$N$取不同值的时候，新的线性未知数的个数分别为：

   - $N=1$，线性未知数的个数为1；
   - $N=2$，线性未知数的个数为3；
   - $N=3$，线性未知数的个数为6；
   - $N=4$，线性未知数的个数为10；

$N\!=\!4\!$的时候，未知数的个数多于方程的个数了。注意到$\!\beta_{ab}\beta_{cd}\!=\!\beta_{a}\beta_{b}\beta_{c}\beta_{d}\!=\beta_{a'b'}\beta_{c'd'}\!$,其中$\!\{a',b',c',d'\}\!$是$\{a,b,c,d \}$的一个排列，我们可以减少未知数的个数。举例，如果我们求出了$\beta_{11},\beta_{12},\beta_{13}$,那么我们可以得到$\beta_{23}=\frac{\beta_{12}\beta_{13}}{\beta_{11}}$。这样，即使对于$N=4$，我们也可以求解出$\{\beta_{ij}\}_{i,j=1,\cdots,N}$了。<br />**高斯-牛顿最优化**<br />优化的目标函数为：<br />$\text{Error}(\beta)=\sum_{(i,j)s.t.\,\,i<j}\left( \left\| \bm{c}_i^{c} - \bm{c}_j^c \right\|^2 - \left\| \bm{c}_i^{w} - \bm{c}_j^w \right\|^2\right)^2$<br />这是一个无约束的非线性最优化问题，简单。<br />求得系数后，便可以得到世界坐标系下3D点坐标在相机坐标系下坐标，接下来使用icp算法进行求解。<br />icp算法求解的步骤为：<br />**Step1：计算点云质心坐标**<br />$\begin{aligned}
&\bm{\text{P}}_c^c=\frac{1}{n}\sum_{i=1}^{n}\bm{P}_i^c 
\end{aligned}$<br />$\begin{aligned}
&\bm{\text{P}}_c^w=\frac{1}{n}\sum_{i=1}^{n}\bm{P}_i^w
\end{aligned}$<br />**Step2：点云去质心**<br />$\bm{\bar{\text{P}}}^c=
\begin{bmatrix}
({\bm{\text{p}}_1^c})^T-({\bm{\text{p}}_c^c})^T \\
\cdots \\
({\bm{\text{p}}_n^c})^T-({\bm{\text{p}}_c^c})^T
\end{bmatrix}$<br />$\bm{\bar{\text{P}}}^c=
\begin{bmatrix}
({\bm{\text{p}}_1^w})^T-({\bm{\text{p}}_c^w})^T \\
\cdots \\
({\bm{\text{p}}_n^w})^T-({\bm{\text{p}}_c^w})^T
\end{bmatrix}$<br />**Step3：计算旋转矩阵**$\text{R}$<br />$\text{W}=\left(\bm{\bar{\text{P}}}^c\right)^T \bm{\bar{\text{P}}}^w$<br />$[\bm{\text{U}\Sigma \text{V}}]=\text{SVD}(\bm{\text{W}})$<br />$\bm{\text{R}}=\bm{\text{UV}^T}$<br />如果$det(\bm{\text{R}})<0,\,\bm{\text{R}}(2,:)=-\bm{\text{R}}(2,:)$<br />**Step4：计算平移向量t**<br />$\bm{\text{t}}=\bm{\text{p}}_c^c-\bm{\text{Rp}}_c^w$<br />注意上面提到R和t有四种情况的解，最后通过计算反投影误差。采用反投影误差最小的求解结果。
<a name="oIcJF"></a>
## 7.4 ICP
ICP方法是用于求解$\!\!3\text{D}\!-\!3\text{D}\!\!$的位姿估计问题，ICP算法为迭代最近点算法，是求两个点云之间精配准的算法，其流程主要如下：

   1. 寻找最近点
   2. 计算loss,得到当前使loss最小的旋转和平移
   3. 变换原点云，再次寻找最近点
   4. 判断此时距离是否小于某个阈值，即收敛，没有则再次回到1进行迭代。

计算的loss函数如下，其中pt,ps为target点云下对应点及source点云下对应点。<br />$R^\ast,t^\ast=\arg{\min\limits_{R,t} }\frac{1}{{\left|P_s\right|} } \sum_{i=1}^{\left|P_s\right|}{\left|p_t^i-\left(R\cdot p_s^i+t\right)\right|^2}$<br />第一步，寻找最近点，有如下两种方法：<br />1.设置距离阈值，当点与点距离小于一定阈值就认为找到了对应点，不用遍历完整个点集；<br />2.使用 ANN(Approximate Nearest Neighbor) 加速查找，常用的有 KD-tree；KD-tree 建树的计算复杂度为 O(N log(N))，查找通常复杂度为 O(log(N))（最坏情况下 O(N)）。<br />第二步，寻找最优变换<br />对于 point-to-point ICP 问题，求最优变换是有闭形式解（closed-form solution）的，可以借助 SVD 分解来计算。<br />计算两个点云的质心μt, μs<br />$\mu_p=\frac{1}{n}\sum_{i=1}^{n}p_i \mu_q=\frac{1}{n} \sum_{i=1}^{n}q_i$<br />对匹配点距离误差进行变换：<br />$\begin{aligned}
&\frac{1}{2}\sum_{i=1}^{n} \left\| \bm{\text{q}}_i-\bm{\text{Rp}}_i- \bm{\text{t}} \right\|^2 \\
=&\frac{1}{2}\sum_{i=1}^{n} \left\| \bm{\text{q}}_i-\bm{\text{Rp}}_i- \bm{\text{t}} - (\mu_q-\bm{\text{R}\mu}_p)+(\bm{\mu}_q-\bm{\text{R}\mu}_p) \right\|^2 \\
=&\frac{1}{2}\sum_{i=1}^{n} \left\| \bm{(\text{q}}_i-\bm{\mu}_q - \bm{\text{R}}(\bm{\text{p}}_i-\bm{\mu_p}))+(\bm{\mu}_q-\bm{\text{R}\mu}_p -\bm{\text{t}}) \right\|^2 \\
=&\frac{1}{2}\sum_{i=1}^{n}\left( \left\| \bm{\text{q}}_i-\bm{\mu}_q - \bm{\text{R}}(\bm{\text{p}}_i-\bm{\mu_p}) \right\|^2+
\left\| \bm{\mu}_q-\bm{\text{R}\mu}_p -\bm{\text{t}} \right\|^2+
2(\bm{\text{q}}_i-\bm{\mu_q}-\bm{\text{R}}(\bm{\text{p}}_i-\bm{\mu_p}))^T
(\bm{\mu_q}-\bm{\text{R}}\bm{\mu_p-t})
\right)
\end{aligned}$<br />对于最后一项，有<br />$\begin{aligned}
&\sum_{i=1}^{n}(\bm{\text{q}}_i-\bm{\mu}_q-\bm{\text{R}}(\bm{\text{p}}_i-\bm{\mu}_p))^T(\bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p-\bm{\text{t}}) \\

=&(\bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p-\bm{t})^T\sum_{i=1}^{n}(\bm{\text{q}}_i-\bm{\mu}_q-\bm{\text{R}}(\bm{\text{p}}_i-\bm{\mu}_p)) \\

=&(\bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p-\bm{t})^T(n\bm{\mu}_q-n\bm{\mu}_q-\bm{\text{R}}(n\bm{\mu}_p-n\bm{\mu}_p)) \\
=&\bm{0}
\end{aligned}$<br />再进一步变换：<br />设$\bm{\text{p}}_i^{\prime}=\bm{\text{p}}_i-\bm{\mu}_p,\,\bm{\text{q}}_i^{\prime}=\bm{\text{q}}_i-\bm{\mu}_q$，目标函数简化为：<br />$\frac{1}{2}\sum_{i=1}^{n}\left(\left\| \bm{\text{q}}_i^{\prime}-\bm{\text{R}}\bm{\text{p}}_i^{\prime} \right\|^2+\left\| \bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p-\bm{t} \right\|^2\right)$<br />设$\bm{\text{R}}^\ast,\bm{\text{t}}^\ast$为最优解，可以将优化变为如下问题：

   1. $\bm{\text{R}}^\ast=\argmin_{\bm{\text{R}}}\frac{1}{2}\sum_{i=1}^{n}\left\| \bm{\text{q}}_i^\prime-\bm{\text{R}}\bm{\text{p}}_i^\prime \right\|^2$
   2. $\bm{\text{t}}^\ast = \bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p$

对于第一步，有：<br />$\begin{aligned}
\bm{\text{R}}^\ast &=\argmin_{\bm{\text{R}}}\frac{1}{2}\sum_{i=1}^{n}\left( {\bm{\text{q}}_i^\prime}^T \bm{\text{q}}_i + 
{\bm{\text{p}}_i^\prime}^T \bm{\text{R}}^T\bm{\text{R}} \bm{\text{p}}_i^\prime
-2{\bm{\text{q}}_i^\prime}^T \bm{\text{R}} \bm{\text{p}}_i^\prime
\right) \\

&=\argmin_{\bm{\text{R}}}\frac{1}{2}\sum_{i=1}^{n}\left( {\bm{\text{q}}_i^\prime}^T \bm{\text{q}}_i + \bm{\text{I}}-2{\bm{\text{q}}_i^\prime}^T \bm{\text{R}} \bm{\text{p}}_i^\prime \right) \\
&=\argmin_{\bm{\text{R}}}\sum_{i=1}^{n}-{\bm{\text{q}}_i^\prime}^T \bm{\text{R}} \bm{\text{p}}_i^\prime
\end{aligned}$<br />对这个优化，有如下最优解：<br />令$\bm{\text{W}}=\sum_{i=1}^{n}\bm{\text{q}}_i^\prime {\bm{\text{p}}_i^\prime}^T$，通过SVD分解有<br />$\bm{\text{W}}=\bm{\text{U}}\Sigma\bm{\text{V}}^T$<br />求得旋转和平移为：<br />当$\bm{\text{W}}$满秩事有：<br />$\Sigma=
\begin{bmatrix}
\sigma_1 & 0 & 0 \\
0 & \sigma_2 & 0 \\
0 & 0 & \sigma_3
\end{bmatrix}$<br />且对应唯一的$\bm{\text{U}},\bm{\text{V}}$组合，对应地<br />$\begin{aligned}
&\bm{\text{R}}^\ast=\bm{\text{U}}\bm{\text{V}}^T \\
&\bm{\text{t}}^\ast=\bm{\mu}_q-\bm{\text{R}}\bm{\mu}_p
\end{aligned}$<br />一般需要对求得的旋转矩阵进行检查，即判断其是否为一个旋转矩阵，判断方式如下：<br />最后还需要进行Orientation rectification，即验证$\bm{R}^\ast=\bm{V}\bm{U}^T$是不是一个旋转矩阵（检查是否有$|R|=1$, 因为存在$|R|=-1$的可能，此时$R$表示的不是旋转而是一个reflection，所以我们还要给上述优化解加上一个$|R|=1$的约束）。<br />根据矩阵行列式的性质，以及$\bm{\text{U,V}}$都是正交阵：<br />$\begin{aligned}
|\bm{M}|&=|\bm{V}^T||\bm{U}||\bm{R}| \\
&=|\bm{V}^T||\bm{U}|=\pm1
\end{aligned}$<br />如果$\!|VUT|\!=\!1\!$，则$\!|M|\!=\!1\!$,$\!\bm{R}^\ast\!=\!VU^T\!$已经给出最有旋转：如果$\!|VU^T|\!=\!-1\!$，则$\!|M|\!=\!-1\!$，我们需要求解此时的$\!R\!$，也就是分析$\!M\!$应该具有何种形式。经分析可得出结论：当$\!|M|\!=\!-1\!$时，使得$\!trace(\Sigma M)\!$最大的$M$为：<br />$M=
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & -1
\end{bmatrix}$<br />综合考虑$\!|M|\!=\!1\!$和$\!|M|\!=\!-1\!$两种情况，我们可以得到：<br />$R^\ast=V
\begin{bmatrix}
1 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & |VU^T|
\end{bmatrix} U^T$<br />第三步，迭代<br />每一次迭代我们都会得到当前的最优变换参数$R_k,t_k\!$，然后将该变换作用于当前源点云；"找最近对应点"和"求解最优变换"这两步不停迭代进行，直到满足迭代终止条件，常用的终止条件有：<br />1.$R_k,t_k\!$的变化量小于一定值<br />2.loss 变化量小于一定值<br />3.达到最大迭代次数<br />迭代结束后，把所有的平移和旋转求和，就是我们要求的平移和旋转了。
<a name="pXsuI"></a>
## 本章小结
本章主要介绍了3D-2D中常用算法，最常见的是DLT算法。在EPNP，P3P中，EPNP由于能使用更多点信息，在实际工程中更为常见。ICP则既可以在视觉SLAM中使用，更多的是在激光中使用。
<a name="SzdkY"></a>
## 本章思考
1.EPNP算法为什么会更高效？<br />2.在特征点匹配过程中，不可避免地会遇到误匹配的情况。如果我们把错误匹配输入到PNP或ICP中，会发生怎样的情况？你能想到哪些避免误匹配的方法？
<a name="zmRIC"></a>
## 参考
[1.PnP算法详解（超详细公式推导）](https://www.jianshu.com/p/a35fa8ac0803)<br />[2.【Frobenius norm（弗罗贝尼乌斯-范数）（F-范数）】](https://blog.csdn.net/weixin_45251808/article/details/102968002)<br />[3.3D-2D：PnP算法原理](https://blog.csdn.net/u014709760/article/details/88029841)<br />[4.PnP(Perspective-n-Point)问题：各种算法总结分析](https://zhuanlan.zhihu.com/p/399140251)<br />[5.位姿估计之PnP算法](https://blog.csdn.net/feng_float/article/details/122693009)<br />[6.相机位姿求解——P3P问题](https://www.cnblogs.com/mafuqiang/p/8302663.html)<br />[7.[PnP]PnP问题之EPnP解法](https://zhuanlan.zhihu.com/p/59070440)<br />[8.深入EPnP算法](https://blog.csdn.net/jessecw79/article/details/82945918)<br />[9.EPnP原理与源码详解](https://zhuanlan.zhihu.com/p/361791835)<br />[10.三维点云配准之ICP算法](https://wangwenlonggg.github.io/2021/05/26/%25E4%25B8%2589%25E7%25BB%25B4%25E7%2582%25B9%25E4%25BA%2591%25E9%2585%258D%25E5%2587%2586%25E4%25B9%258BICP%25E7%25AE%2597%25E6%25B3%2595/)<br />[11.三维点云配准 -- ICP 算法原理及推导](https://zhuanlan.zhihu.com/p/104735380)

