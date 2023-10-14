# **8.在不确定中寻找确定：后端之卡尔曼滤波**

在观察的领域中，机遇只偏爱那种有准备的头脑。——巴斯德

---

> **本章主要内容**
> 卡尔曼滤波原理

<a name="Sv22o"></a>
## 8.1 滤波
滤波，我理解为，过滤一些东西。那么卡尔曼滤波过滤的是什么东西呢？答案就是过滤掉信号中的噪声。
> 卡尔曼滤波通常用来估计带噪信号中隐藏的真实信息。它的输入信号带有“过程噪声”和“观测噪声”，输出信号是对真实信号的最优估计，处理的本质是对噪声的滤除，因此称为“滤波”合情合理。
> 还可以认为卡尔曼滤波是一个估计器：每一时刻，它迭代式的对隐藏信息进行预测，再根据观测值进行修正，最后给出最优估计。
> 最后，还可以认为卡尔曼滤波是一系列数学方程的总称，如下表所示，经典的卡尔曼滤波算法由五个公式组成，希望你在读完本文后，可以对这五个公式有了深刻的理解。
> $\begin{aligned}
& \tilde{\mathbf{x}}_t^{-}=\mathbf{F} \tilde{\mathbf{x}}_{t-1}+\mathbf{B u} \\
& K_t=\frac{P_t^{-} H^T}{H P_t^{-} H^T+R} \\
& \tilde{\mathbf{x}}_t=\tilde{\mathbf{x}}_t^{-}+\mathbf{K}_{\mathbf{t}}\left(\mathbf{z}_{\mathbf{t}}-\mathbf{H} \tilde{\mathbf{x}}_t^{-}\right) \\
& P_t=\left(I-K_t H\right) P_t^{-} \\
& \mathbf{P}_{t+1}^{-}=\mathbf{F} \mathbf{P}_t \mathbf{F}^T+\mathbf{Q}_{\mathbf{t}+\mathbf{1}}
\end{aligned}$

<a name="cApXu"></a>
## 8.2 卡尔曼滤波原理
卡尔曼滤波是基于马尔科夫假设的，即下一时刻状态只与上一时刻有关。其主要针对于线性高斯系统，计算的流程如下：<br />假设状态$\mathbf{X}$的转移方程为：<br />$\begin{equation}
\begin{aligned}
\hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1} \\
\mathbf{P}_{k} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T}
\end{aligned}
\end{equation}$<br />其中$\mathbf{F}_{k}$为状态转移矩阵，而$\mathbf{P}_{k}$为$\mathbf{X}_{k}$的方差<br />$\mathbf{X}$只表示自身的状态，另外一些系统还会有外部控制因数，比如火车减速时，速度是状态量，但可以有刹车装置进行减速，如果系统存在这部分控制因数，需要把这部分加到状态转移中：<br />$\begin{aligned}
\hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1} +Bu_k\\
\mathbf{P}_{k} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T}
\end{aligned}$<br />另外，$\mathbf{X}$中包含环境的一些未知变量，我们假设为噪声，同时噪声分布假设服从高斯分布，于是有如下方程：<br />$\begin{aligned}
\hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1} +Bu_k +w_k\\
\mathbf{P}_{k} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T} +R
\end{aligned}$<br />卡尔曼滤波中，还需要使用观测方程来更新，一般观测方程是需要从状态变为观测量的，即需要有一个观测量到状态的转换，但很多时候这个转换方程都没有，但这并不影响我们假设，如果没有转换方程，到时候直接把转换矩阵设为单位矩阵就行，那现在假设观测方程为：<br />$z_k = H*x_k+v_k$<br />其中$H$为观测方程，$v_k$为观测的噪声分布，假设其服从$v_k\sim N\left(0,Q\right)$，即零均值，方差为Q的高斯分布：<br />到这里，需要说明一点，上一次滤波的结果，会作为下一次滤波的初始值，即由上一次后验概率，通过状态转移矩阵与控制向量，变为目前的先验值，所以原来的观测方程需要变为：<br />$\begin{aligned}
\hat{\mathbf{x}}_{\bar{k}} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1} +Bu_k +w_k\\
\mathbf{P}_{\bar{k}} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T} +R
\end{aligned}$<br />其中下标不带横杠的，表示后验值，带横杠的，代表先验值，这样就由上一时刻的最优估计，得到了当前预测的先验状态及先验方差。<br />对于观测方程，其方差主要由观测方差决定，即：<br />$P_{z_k} = R$<br />zk的均值为H*xk,即zk服从N(H*xk,R)的分布。<br />这两个分布都是高斯分布，一个是状态的先验分布，一个是传感器测量的分布，这个测量与xk的状态也有关，现在要求这两个分布的联合分布，并求其最大值，很简单，把两个分布乘起来，由于高斯分布的乘积，还是高斯分布，取其均值处，就是概率最大的状态。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684340645013-e837abfe-6084-4761-9ec1-7511a7736935.png#averageHue=%23fdfdfd&clientId=ud705f196-c300-4&from=paste&height=353&id=ub773d1ce&originHeight=441&originWidth=720&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=59646&status=done&style=none&taskId=u8bd53790-1012-4600-bae7-ceada7e80d0&title=&width=576)<br />乘积的部分，就是所说的状态更新，首先由高斯分布的公式可得：<br />$\mathcal{N}(x, \mu, \sigma)=\frac{1}{\sigma \sqrt{2 \pi}} e^{-\frac{(x-\mu)^{2}}{2 \sigma^{2}}}$<br /> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684340775920-336bcbe7-4fef-42d0-a4f9-382847769489.png#averageHue=%23ecece5&clientId=ud705f196-c300-4&from=paste&height=65&id=ub2acad42&originHeight=81&originWidth=370&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=20046&status=done&style=none&taskId=u43293ced-253c-4f7d-9f21-f6e9b80d8a8&title=&width=296)<br />那么两个分布相乘，系数结果为：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684340792537-69fa9ec7-7ec5-4eec-bfdc-8136aa44b775.png#averageHue=%23edece7&clientId=ud705f196-c300-4&from=paste&height=135&id=uf7b2cb96&originHeight=169&originWidth=221&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=26016&status=done&style=none&taskId=u1fd35b68-9970-40b4-ae11-b28f88ce9c0&title=&width=176.8)<br />其中均值u就是概率最大时的值，σ就是方差，其中这个式子就是最后我们要求的，但这个式子有点复杂，于是用一个系数化简为：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684340927621-51d076b6-bd29-4fcd-9196-dc197d120eec.png#averageHue=%23f0efea&clientId=ud705f196-c300-4&from=paste&height=146&id=u45252e44&originHeight=182&originWidth=203&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=24884&status=done&style=none&taskId=udebdd01d-a3a3-454c-ba0c-ae10cd15303&title=&width=162.4)<br />其中k称为卡尔曼增益，u0为预测值，u1为观测。可以看到，k为0到1之间的数，分子为预测的方差，如果预测方差越大，则越向观测值靠拢，如果预测方差越小，则越向预测值靠拢。<br />截至目前，我们有用矩阵$\left(\mu_0, \Sigma_0\right)=\left(H_k \hat{x}_k, H_k P_k H_k^T\right)$预测的分布，有用传感器读数$\left(\mu_1, \Sigma_1\right)=\left(\vec{z}_k, R_k\right)$预测的分布。把它们代入上节的矩阵等式中：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684341570383-5015b365-70f6-425c-a3ed-64c03ba69afe.png#averageHue=%23fafaf9&clientId=ud705f196-c300-4&from=paste&height=135&id=u5107c633&originHeight=329&originWidth=771&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=68732&status=done&style=none&taskId=u34f92720-17ab-4f0a-958d-21bb8baa301&title=&width=316)<br />相应的，卡尔曼增益就是：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684341570383-5015b365-70f6-425c-a3ed-64c03ba69afe.png#averageHue=%23fafaf9&clientId=ud705f196-c300-4&from=paste&height=110&id=wM391&originHeight=329&originWidth=771&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=68732&status=done&style=none&taskId=u34f92720-17ab-4f0a-958d-21bb8baa301&title=&width=258)

两个式子左边都有不少Hk矩阵，同时把这个矩阵去掉，则K变为：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/1782698/1684341669218-6197f9b4-c149-4762-a3c1-c1f11bc58158.png#averageHue=%23eff3e6&clientId=ud705f196-c300-4&from=paste&height=128&id=u0db3976f&originHeight=160&originWidth=313&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=31158&status=done&style=none&taskId=u5aeeae1e-f537-4732-980b-6114a1df822&title=&width=250.4)<br />于是，我们得到最后卡尔曼更新的公式：<br />$\begin{array}{c}
K_{k}=\frac{P_{k}^{-} H^{T}}{H P_{k}^{-} H^{T}+R} \\
\hat{x}_{k}=\hat{x}_{\bar{k}}+K_{k}\left(z_{k}-H \hat{x}_{\bar{k}}\right) \\
P_{k}=\left(I-K_{k} H\right) P_{\bar{k}}
\end{array}$<br />其中计算K的都是使用先验方差，R为传感器方差。<br />Zk为实际观测值，Hxk为预测的观测值<br />最后使用K及先验方差，得到后验方差及后验均值。<br />卡尔曼滤波更新结束，是不是很简单。
<a name="oAPwt"></a>
## 本章小结
本章主要介绍什么是卡尔曼滤波，及其算法过程。
<a name="iUcSc"></a>
## 本章思考
1.卡尔曼滤波适用于什么系统？<br />2.卡尔曼增益的含义是什么？
<a name="yG8Fe"></a>
## 参考
[1.图说卡尔曼滤波，一份通俗易懂的教程](https://zhuanlan.zhihu.com/p/39912633)<br />[2.卡尔曼滤波五个公式各个参数的意义](https://blog.csdn.net/wccsu1994/article/details/84643221)<br />[3.卡尔曼滤波（Kalman Filter）原理与公式推导](https://zhuanlan.zhihu.com/p/48876718)<br />[4.卡尔曼滤波算法的五大核心公式含义](https://blog.csdn.net/qq_36812406/article/details/127821768?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-127821768-blog-84643221.235^v36^pc_relevant_default_base&spm=1001.2101.3001.4242.1&utm_relevant_index=3)<br />[5.一文看懂卡尔曼滤波（附全网最详细公式推导和Matlab代码）](https://zhuanlan.zhihu.com/p/559191083)
