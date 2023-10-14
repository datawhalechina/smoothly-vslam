# **9.每次更好，就是最好：后端之非线性优化**

人生如同故事。重要的并不在有多长，而是在有多好。——塞涅卡

---

> **本章主要内容** \
> 1.最小二乘问题 \
> 2.牛顿法 \
> 3.高斯牛顿法 \
> 4.LM法 \
> 5.BA

在后端优化中，存在着两种优化算法，一种是利用矩阵分解求得最小二乘问题解的方法，不用迭代，这种我称之为线性代数的方法。第二种就是利用迭代的方式，不断调整参数，使残差最小，这种算法是本章要介绍的算法，也是我们通常所说的非线性优化的方法。
<a name="yWqfM"></a>
## 9.1 非线性优化
这里有类似于机器学习中监督学习的概念，使用预测值与观测值构建出一个残差，而状态值与环境观测则是需要调节的变量。把所有残差求和，迭代更新变量值，使loss不断下降，到一个最小值时，这时的状态量就被称之为最优估计值。<br />这里的loss，一般会构建为观测与预测的差值的平方，所以构建的问题一般为最小二乘问题。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684572172464-9a531ea9-9ea8-4ccf-a694-93a20b08a6bd.png#averageHue=%23f6f6f6&clientId=ue566fc9f-7bc8-4&from=paste&height=304&id=RHGRm&originHeight=380&originWidth=619&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=76876&status=done&style=none&taskId=u7c1f007a-17c1-4a39-b8b2-a379063d2aa&title=&width=495.2)
<a name="Y31RA"></a>
## 9.2牛顿法
牛顿法来源于求方程解，但求极值的过程，也是求函数导数为0的过程，所以二者本质上是一样。记残差函数为：<br />$F(x) = \frac {1}{2}||f(x)||^2$<br />对F(x)进行二阶泰勒展开<br />$F(x) = F(x_0) + F^{'}(x_0)(x-x_0)+\frac{1}{2}F^{''}(x_0)(x-x_0)^2 +\sigma^2(x-x_0)^2$<br />对F(x)的泰勒展开求导：<br />$F^{'}(x) = F^{'}(x_0)+F^{''}(x_0)(x-x_0)$<br />令导数为0，得到<br />$F^{''}(x_0)(x-x_0) = -F^{'}(x_0)$<br />将$x-x_0$视作$x_0$基础上的增量$\Delta x$，则$x = x_0 +\Delta x$，于是原方程变为<br />$F^{''}(x_0)\Delta x = -F^{'}(x_0)$<br />对于变量为向量的情况，这里的一阶导数与二阶导数，就变为雅克比矩阵J与海森矩阵H。上式可变为：<br />$H(x_0)\Delta x = -J(x_0)$<br />求得增量$\Delta x$后，更新$x_0$，在计算新x下loss，重复求增量，不断迭代直到收敛，最后得到的x就是我们要求的最优变量。<br />上面的方程也称为增量方程。
<a name="PvHzD"></a>
## 9.3 高斯牛顿法
对于复杂函数，特别2023是非线性函数，求二阶导数十分困难，甚至是难以求出的，牛顿法中直接求海森矩阵的做法十分困难，因此出现了高斯牛顿法：<br />具体做法是对f(x)做一阶泰勒展开，原loss函数变为<br />$\begin{array}{l}
F(x) = \frac {1}{2}||f(x)||^2 = \frac {1}{2}||f(x_0)+f^{'}(x_0)(x-x_0)||^2 \\
=\frac{1}{2}\left(\|f(x_0)\|^{2}+2 f(x_0) f^{'}(x_0) (x-x_0)+ f^{'}(x_0)^2(x-x_0)^2 \right)
\end{array}$<br />对上式关于x求导，得：<br />$\begin{array}{l}
F^{'}(x)= f(x_0) f^{'}(x_0) + f^{'}(x_0)^2(x-x_0)
\end{array}$<br />令导数等于0，有：<br />$f^{'}(x_0)^2(x-x_0) = -f(x_0) f^{'}(x_0)$<br />同样的，令$x-x_0$为$\Delta x$，对于多元函数，一阶导为雅克比矩阵，于是原方程变为：<br />$J(x_0)J(x_0)^{T} \Delta x = -f(x_0)J(x_0)$<br />同样的方法，求出增量x后，更新x，得到当前loss，不断迭代下去直到收敛。<br />对比于牛顿法的增量方程，可以看到高斯牛顿法左边用更容易求导的雅克比矩阵的乘积，近似代替了海森矩阵。右侧则多乘了一个$f(x_0)$<br />$H(x_0)\Delta x = -J(x_0)$

<a name="vMFnN"></a>
## 9.4 列文伯格-马夸而特法
在求增量方程的解时，涉及矩阵求逆，如下：<br />$\Delta x = -H(x_0)^{-1}J(x_0)$

在高斯牛顿法中,$H(x_0)$对应$J(x_0)J(x_0)^T$<br />但是，对于高斯牛顿法，并不能保证$H(x_0)$可逆，另外，如果增量$\Delta x$过大，线性近似的误差也会很大，为了解决这两种问题，出现了列文伯格-马夸而特法，也就是LM法。<br />LM法是对高斯牛顿法（GN法）的改进，通过引入信赖区域的残差，对近似范围进行了限制，其中的残差为：<br />$\min \frac{1}{2}\left\|f\left(x_k\right)+J\left(x_k\right)^T \Delta x_k\right\|^2, \quad \text { s.t } \quad\left\|D \Delta x_k\right\|^2 \leq \mu$<br />增量方程的获取，构建拉格朗日函数，$\lambda$是系数因子：<br />$L(\Delta x, \lambda)=\frac{1}{2}\left\|f(x)+J(x)^T \Delta x\right\|^2+\frac{\lambda}{2}\left(\|D \Delta x\|^2-\mu\right)$<br />这样的话，化简后求导就可以得到：<br />$J(x) f(x)+J(x) J^T(x) \Delta x+\lambda D^T D \Delta x=0$<br />我们化简后得到：<br />$\left(J J^T+\lambda D^T D\right) \Delta x=-J f$<br />在本文中，我们令$H=J J^T, g=-J f$。在实际使用中，通常用$I$来代替$D^TD$。所以，公式就变为：<br />$(H+\lambda I) \Delta x_k=g$<br />这样就得到了增量方程。<br />LM称为置信区域法，其会对线性近似程度的好坏进行度量，通过如下系数计算：<br />$\rho=\frac{f(x+\Delta x)-f(x)}{J(x)^{T} \Delta x}$<br />该分数代表近似的准确性，在一定范围内，会认为近似是有效的，因此LM法也称为基于区域搜索的方法，及信赖区域法，可以看到其分为以下几种情况：

   - ρ 接近1，近似是好的，不需要更改；
   - ρ 太小，则实际减少的值小于近似减少的值，近似较大，需要缩小近似的范围；
   - ρ 太大，则实际减少的值大于近似减少的值，近似较小，需要扩大近似的范围。

根据ρ的值，来进行步长的动态调整，即改变增量方程中的λ值。- ρ 越大，表明 L对F的近似效果越好，所以可以增大 λ以使得LM法接近梯度下降法<br />	相反，ρ 越小，表明 L的近似越差，所以应该减小μ以使得迭代公式接近高斯牛顿法。<br />由于系数矩阵中加入了λ乘以单位矩阵，可以保证系数矩阵的正定可逆。其次，通过改变λ值，可以起到调整步长的作用，当线性近似程度不好时，减小更新的步长，可以有更好的收敛性。
<a name="ioQeJ"></a>
## 9.5 BA
BA是VSLAM里广泛使用的技术，它通过同时优化地图点与相机位姿，使空间3维点与相机平面观测的误差最小，每个3D点光束都能穿过观测相机的光心。BA是VSLAM里常用的后端算法，但由于其灵活性，也可以在前端使用。相比于上面几种线性代数的方法，BA一方面是非线性方法，可以最大程度使用所有匹配点数据，另一方面可以优化三维点坐标，被称为slam中万金油的方法。<br />BA不仅可以求两帧之间的位移，还可以求多帧的姿态。如果固定住路标点，只优化位姿，这种BA叫做运动BA。而把路标点与姿态一起优化叫做完整BA。由于优化变量数量庞大，完整BA优化比较耗时。<br />BA优化时需要给一个迭代的初始值，一般做法是不直接进行BA，而是**先用前面介绍的PnP算法或者对极几何先获得相机位姿及3D点坐标初始值，然后把初始值交给BA做最终的优化调整**。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683995376650-0bd3f787-1ea0-49e9-b992-e98e8a2404dd.png#averageHue=%23f2efee&clientId=u2865029d-01ae-4&from=paste&height=397&id=uc539b179&originHeight=496&originWidth=692&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=269226&status=done&style=none&taskId=u8a2a4d6a-3c26-4b96-a75c-f42f39889a4&title=&width=553.6)
> BA算法步骤主要如下：
> 1.构建BA残差
> 2.构建增量线性方程
> 3.利用增量线性方程系数矩阵的稀疏性，进行边缘化，求解出位姿增量
> 4.再求3D点增量
> 5.更新变量
> 6.判断是否收敛，没有的话则返回1继续迭代计算

下面进行详细介绍：<br />**第一步，构建残差**<br />BA残差为特征点观测值与预测值之差的平方，其中观测值为像素坐标系下特征点坐标，预测值为3D点在该相机坐标系下的像素平面投影坐标。这里需要把3D点转换到当前相机坐标系，假设3D点为点P，相机相对旋转与平移为R,t<br />$P^{\prime} = Rp+t$<br />P'在归一化平面上坐标为：<br />$P_c = [u_c,v_c,1] = [P^{\prime}_x/P^{\prime}_z,P^{\prime}_y/P^{\prime}_z,1]$<br />通过相机镜头时，存在畸变，则畸变坐标为：<br />$\left\{\begin{array}{l}
u_{\mathrm{c}}^{\prime}=u_{\mathrm{c}}\left(1+k_{1} r_{\mathrm{c}}^{2}+k_{2} r_{\mathrm{c}}^{4}\right) \\
v_{\mathrm{c}}^{\prime}=v_{\mathrm{c}}\left(1+k_{1} r_{\mathrm{c}}^{2}+k_{2} r_{\mathrm{c}}^{4}\right)
\end{array}\right.$<br />根据相机模型，最后像素坐标为：<br />$\left\{\begin{array}{l}
u_{\mathrm{s}}= f_x*u_{\mathrm{c}}^{\prime} + c_x \\
v_{\mathrm{s}}= f_y*v_{\mathrm{c}}^{\prime} + c_y
\end{array}\right.$<br />以上，从原始3D点，到观测点的转换，称为观测模型，这里记为：<br />$z = h(T,p)$<br />其中T为包含旋转与平移的转换矩阵。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683992427303-97b0dc6d-5374-49ec-b1b8-6e27f91d8460.png#averageHue=%23f6f6f6&clientId=u55127b8a-7d04-4&from=paste&height=199&id=Blf1p&originHeight=249&originWidth=1016&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=87687&status=done&style=none&taskId=uf68d77ca-c8d7-48c3-b834-b1a6bceb3f6&title=&width=812.8)<br />转换过程如上图：<br />现在，设相机平面观测像素坐标为z',则观测误差为：<br />$e = z^{\prime} - h(T,p)$<br />将所有点的观测残差平方求和，得到如下观测残差：<br />$\frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{n}\left\|\bm{e}_{ij} \right\|^2=
\frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{n}\left\| \bm{z}_{ij}-h(T_i,p_j)\right\|^2.$<br />该残差为最小二乘残差，对其求最小化的过程，同时对位姿与3D点进行了调整，就是所谓的BA，到此，BA残差构建完毕！<br />可以看到，这个残差也是通过3D-2D点的关联完成的。

**第二步，构建增量方程**<br />将所有待求变量，写为一个向量形式为：<br />$\bm{x}=[T_1,\cdots,T_m,p1,\cdots,p_n]^\text{T}.$<br />向量中$\!T\!\!$表示相机位姿，即旋转与平移，$\!p\!$表示3D点坐标，如果是求两帧之间相对位移，则其中$\!T\!$就只有$\!T_1\!$。假设残差函数为$\!f(x)\!$，则当我们在$x$上加一个增量$\Delta_x$时，$\!f(x)\!$的变化为：<br />$\frac{1}{2}\| f(x+\Delta x) \|^2 \approx \frac{1}{2}\sum_{i=1}^{m}\sum_{i=1}^{n}
\|
e_{ij}+F_{ij}\Delta\xi_i+E_{ij}\Delta p_j
\|^2$<br />其中，$F$表示整个代价函数在当前状态下对相机姿态的偏导数，而$E$表示该函数对路标点位置的偏导。以上是对每一个点进行求和，现在把每个点及每个姿态，各自放在一起，形成一个向量，则上述式子变为：<br />$\frac{1}{2}\| f(x+\Delta x) \|^2 = \frac{1}{2} \|
e+F\Delta x_c+E\Delta x_p
\|^2$<br />对该函数求最小值，需要用到非线性优化中的最小二乘法，不对其进行展开讨论，只说明一般形式，不管选择何种最小二乘中的高斯牛顿还是LM法，最后都需要求增量方程，该方程由上式求导数为0得来：<br />$\bm{H}\Delta \bm{x}=\bm{g}.$<br />如果是高斯牛顿法，则增量方程如下：<br />$J^T(x_0)J(x_0)\Delta x=-J^T(x_0)f(x_0)$<br />高斯牛顿法和列文伯格一马夸尔特方法的主要差别在于，这里的$H$是取$J^TJ$还是$J^TJ+\lambda I$的形式。由于把变量归类成了位姿和空间点两种，所以雅可比矩阵可以分块为<br />$J=[F\,\, E]$<br />那么以高斯牛顿法为例，$H$矩阵为如下形式：<br />$H=J^TJ=
\begin{bmatrix}
F^TF & F^TE \\
E^TF & E^TE
\end{bmatrix}$<br />因为该方法包含了所有变量，一般特征点有几百个，这个$H$矩阵会非常大，在求增量时，如果直接求矩阵的逆，其运算复杂度为$O(n^3)$，计算会非常耗时，但幸运的是，$H$矩阵本身具有稀疏性，通过这个性质，可以加速增量方程的求解。

第三步，通过边缘化求解增量方程：<br />$H$矩阵的稀疏性是由雅可比矩阵$J(x)$引起的。考虑这些代价函数当中的其中一个$e\!$。注意，这个误差项只描述了在$T$看到$P$这件事，只涉及第$\!i\!$个相机位姿和第$\!j\!$个路标点，对其余部分的变量的导数都为0。所以该误差项对应的雅可比矩阵有下面的形式：<br />$J_{ij}(x)=\left(
\bm{0}_{2\times6},...,\bm{0}_{2\times6},\frac{\partial \bm{e}_{ij}}{\partial \bm{T}_i},
\bm{0}_{2\times6},...,\bm{0}_{2\times3},...,\bm{0}_{2\times3},\frac{\partial \bm{e}_{ij}}{\partial \bm{p}_j},
\bm{0}_{2\times3},...,\bm{0}_{2\times3}
\right).$<br />其中$\!\bm{0}_{2\times 6}\!$表示维度为$\!2\!\times\!6\!$的$\!\bm{0}\!$矩阵，同理，$\!\bm{0}_{2\times 3}\!$也是一样的。该误差项对相机姿态的偏导维度为$\!2\!\times\!6\!$，对路标点的偏导维度是$\!2\!\times\!3\!$。这个误差项的雅可比矩阵，除了这两处为非零块，其余地方都为零。这体现了该误差项与其他路标和轨迹无关的特性。<br />以图9-3为例，我们设$\!J_{ij}\!$只在$\!i\!,\!j\!$处有非零块，那么它对$\!H\!$的贡献为$\!\!J_{ij}^TJ_{ij}\!$，具有图上所画的稀疏形式。这个$\!J_{ij}^TJ_{ij}\!$矩阵也仅有4个非零块，位于$\!(i,i),\!(i,j),\!(j,i),\!(j,j)\!$。对于整体的$\!H\!$，有<br />$\bm{H}=\sum_{i,j}\bm{J}_{ij}^{T}\bm{J}_{ij},$<br />请注意，$\!i\!$在所有相机位姿中取值，$\!j\!$在所有路标点中取值。我们把$\!H\!$进行分块：<br />$H=
\begin{bmatrix}
H_{11} & H_{12} \\
H_{21} & H_{22}
\end{bmatrix}.$<br />则J的稀疏性对最后$\!H\!$矩阵稀疏性的影响如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683994394256-8a828950-419e-4327-b7d1-29be5078704a.png#averageHue=%23f5f5f5&clientId=u55127b8a-7d04-4&from=paste&height=221&id=u04e9227c&originHeight=276&originWidth=699&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=50713&status=done&style=none&taskId=u6ed8b425-2cdc-43be-8c89-9235d3912c8&title=&width=559.2)<br />则对原$H$矩阵，可以做如下划分：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1683994554810-e2e324bd-127c-48df-890c-7fa730ff78ea.png#averageHue=%23bababa&clientId=u55127b8a-7d04-4&from=paste&height=340&id=u60058352&originHeight=425&originWidth=433&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=144497&status=done&style=none&taskId=uf54266e3-1e6c-4565-8e30-a9482e743aa&title=&width=346.4)<br />于是，对应的线性方程组也可以由$\!H\Delta x=g\!$变为如下形式：<br />$\begin{bmatrix}
\bm{B} & \bm{E} \\
\bm{E}^T & \bm{C}
\end{bmatrix}
\begin{bmatrix}
\Delta \bm{x}_c \\
\Delta \bm{x}_p
\end{bmatrix} =
\begin{bmatrix}
\bm{v} \\
\bm{w}
\end{bmatrix}.$<br />其中$\!\bm{B}\!$是对角块矩阵，每个对角块的维度和相机参数的维度相同，对角块的个数是相机变量的个数。由于路标数量会远远大于相机变量个数，所以$\!\bm{C}\!$往往也远大于$\!\bm{B}\!$。三维空间中每个路标点为三维，于是$\!\bm{C}\!$矩阵为对角块矩阵，每个块为$\!3\!\times \!3\!$矩阵。对角块矩阵求逆的难度远小于对一般矩阵的求逆难度，因为我们只需要对那些对角线矩阵块分别求逆即可。考虑到这个特性，我们对线性方程组进行高斯消元，目标是消去右上角的非对角部分E,得<br />$\begin{bmatrix}
\bm{I} & -\bm{EC}^{-1} \\
0 & \bm{I}
\end{bmatrix}
\begin{bmatrix}
\bm{B} & \bm{E} \\
\bm{E}^T & \bm{C}
\end{bmatrix}
\begin{bmatrix}
\Delta\bm{x}_c \\
\Delta\bm{x}_p
\end{bmatrix} =
\begin{bmatrix}
\bm{I} & -\bm{EC}^{-1} \\
0 & \bm{I}
\end{bmatrix}
\begin{bmatrix}
\bm{v} \\
\bm{w}
\end{bmatrix}.$<br />整理，得<br />$\begin{bmatrix}
\bm{B}-\bm{EC}^{-1}\bm{E}^T & 0 \\
\bm{E}^T & \bm{C}
\end{bmatrix}
\begin{bmatrix}
\Delta\bm{x}_c \\
\Delta\bm{x}_p
\end{bmatrix} =
\begin{bmatrix}
\bm{v}-\bm{EC}^{-1}\bm{w} \\
\bm{w}
\end{bmatrix}.$<br />消元之后，方程组第一行变成和$\Delta\bm{x}$无关的项。单独把它拿出来，得到关于位姿部分的增量方程：<br />$[\bm{B}-\bm{EC}^{-1}\bm{E}^T]\Delta \bm{x}_c=\bm{v}-\bm{EC}^{-1}\bm{w}.$<br />这个线性方程的维度和$\!\!B\!$矩阵一样。我们的做法是先求解这个方程，然后把解得的$\!\!\Delta\bm{x}\!\!$代入原方程，求解$\!\!\Delta\bm{x}_p\!$。这个过程称为Marginalization，或者Schur消元（Schur Elimination）。相比于直接解线性方程的做法，它的优势在于：<br />1.在消元过程中，由于$\!\bm{C}\!$为对角块，所以$\!\bm{C}^{-1}\!$容易解出。<br />2.求解了$\!\!\Delta\bm{x}_c\!$之后，路标部分的增量方程由$\!\!\Delta\bm{x}_p\!=\!\bm{C}^{-1}(\bm{w}-\bm{E}^T\Delta\bm{x}_c)\!$给出。这依然用到了$\!\bm{C}^{-1}\!$易于求解的特性。

最后，在求得$\bm{x}$的增量后，更新变量值，不断迭代直到问题收敛，最后求得最优的相机位姿及3D点坐标。需要注意的是，**BA优化中需要给一个迭代的初始值，一般做法是不直接进行BA，而是先用PnP算法或者对极几何先获得相机位姿及3D点坐标初始值，然后把初始值交给BA做最终的优化调整**。
<a name="Z3Xri"></a>
## 本章小结
本章介绍了VSLAM后端常用的非线性优化算法，通常情况下，我们会将后端优化问题构建为一个最小二乘问题，然后使用迭代优化的方式求解。求解中常使用LM算法，但在定位中，可能高斯牛顿法用得也比较多。当然，这部分更多的是理论部分，实际很多优化框架中已经把这些算法都实现好了，如g2o与ceres，到时候直接调用就好了。
<a name="pc4fn"></a>
## 本章思考
1.LM法与GN法与梯度下降法的区别与联系是什么？<br />2.查阅资料，矩阵分解如何求解最小二乘问题。
<a name="yIo4G"></a>
## 参考
[1.理解牛顿法](https://zhuanlan.zhihu.com/p/37588590)<br />[2.非线性最小二乘法之Gauss Newton、L-M、Dog-Leg](https://blog.csdn.net/baoxiao7872/article/details/79914083)<br />[3.常见的几种最优化方法（梯度下降法、牛顿法、拟牛顿法、共轭梯度法等）](https://www.cnblogs.com/shixiangwan/p/7532830.html)<br />[4.机器学习中牛顿法凸优化的通俗解释](https://zhuanlan.zhihu.com/p/38541058)<br />[5.非线性优化整理-3.Levenberg-Marquardt法(LM法)](https://blog.csdn.net/boksic/article/details/79177055)<br />[6.贝叶斯理论在SLAM状态估计中的应用](https://zhuanlan.zhihu.com/p/113012522)<br />[7.最小二乘问题的四种解法——牛顿法，梯度下降法，高斯牛顿法和列文伯格-马夸特法的区别和联系](https://zhuanlan.zhihu.com/p/113946848)<br />[8.SLAM后端：BA优化（Bundle Adjustment）](https://blog.csdn.net/LittleEmperor/article/details/105274299)

