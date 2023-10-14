# smoothly-vslam
基本信息 \
本项目涉及VSLAM基础概念及算法原理，通过本项目内容，降低学习者后续学习开源vslam算法的难度，平滑小白学习vslam的学习曲线，为后续学习者掌握复杂的vslam算法打下基础。

    学习周期：20天，每天平均花费时间 2小时-4小时不等，根据个人学习接受能力强弱有所浮动

    学习形式：教程学习+实践

    人群定位：具备一定编程基础，有学习和梳理VSLAM算法的需求。
    先修内容：无
    难度系数：中

## 任务安排

### Task 1: 相机投影及内外参 （3天）

    
    理论部分
    1.1 介绍针孔相机模型
    1.2相机成像的几个坐标系
    1.3 内参矩阵及去畸变的方法
    2.1介绍 坐标系变换
    2.2介绍 相机外参标定
教程地址:

 [1.一幅图像的诞生：相机投影及相机畸变](https://www.yuque.com/u1507140/vslam-hmh/ogc8v31hbzb6efy8) \
 [2. 差异与统一：坐标系变换与外参标定](https://www.yuque.com/u1507140/vslam-hmh/csqub9k4nax99i19)


### Task 2: 三维空间刚体运动 （2天）
    理论部分
    介绍 世界坐标系与里程计坐标系
    介绍 三维刚体运动
    介绍 旋转的参数化

教程地址

[3.描述状态不简单：三维空间刚体运动](https://www.yuque.com/u1507140/vslam-hmh/pn5az0nwigy51f25)


### Task 3: 视觉SLAM简介 （1天）
    理论部分
    介绍 什么是视觉SLAM
    介绍 视觉SLAM的主流框架及分类

教程地址

[4. 也见森林：视觉SLAM简介](https://www.yuque.com/u1507140/vslam-hmh/xkqtacgm3gg6crk5)


### Task 4: 前端-视觉里程计之特征点 （3天）
    理论部分
    介绍 图像特征简述
    介绍 常用特征点

教程地址

[5.以不变应万变：前端-视觉里程计之特征点](https://www.yuque.com/u1507140/vslam-hmh/rcvyw38lhgchkb6g)

### Task 5: 前端-对极几何及相对位姿估计 （3天）
    理论部分
    1.1介绍 对极几何
    1.2介绍 本质矩阵及其求解
    1.3介绍 单应矩阵及其求解
    1.4介绍 三角测量
    2.1介绍 已知3D-2D匹配关系，估计相机相对位姿的常用算法

教程地址

[6.换一个视角看世界：前端-视觉里程计之对极几何](https://www.yuque.com/u1507140/vslam-hmh/yu9032oczfnbnr2y)
[7.积硅步以至千里：前端-视觉里程计之相对位姿估计](https://www.yuque.com/u1507140/vslam-hmh/lusyrekv9r5g0q4n)



### Task 6: 后端之卡尔曼滤波与非线性优化 （4天）
    理论部分
    1.1 介绍 卡尔曼滤波原理
    2.1介绍 最小二乘问题
    2.2介绍 牛顿法
    2.3介绍 高斯牛顿法
    2.4介绍 LM法

教程地址

[8.在不确定中寻找确定：后端之卡尔曼滤波](https://www.yuque.com/u1507140/vslam-hmh/fsblfmf5te9egulq)
[9.每次更好，就是最好：后端之非线性优化](https://www.yuque.com/u1507140/vslam-hmh/swh0g4a0t452pwux)


### Task 7: 回环检测 （2天）
    理论部分
    介绍 回环检测的作用
    介绍 视觉常用回环检测方法
    介绍 使用词袋模型进行回环检测

教程地址

[10.又回到曾经的地点：回环检测](https://www.yuque.com/u1507140/vslam-hmh/mrx3a5aotqr412qe)

### Task 8: 建图 （2天）
    理论部分
    介绍 地图的类型
    介绍 稠密重建
    介绍 点云地图
    介绍 TSDF地图
    介绍 实时三维重建

教程地址

[11.终识庐山真面目：建图](https://www.yuque.com/u1507140/vslam-hmh/ow0woydmogl8wc6n)

### Task 9: 设计VSLAM系统 （2天）选做
    理论部分
    介绍 C++的工程结构
    介绍 系统设计方法
    练习 设计一个VSLAM系统

教程地址

[*12.实践是检验真理的唯一标准：设计VSLAM系统](https://www.yuque.com/u1507140/vslam-hmh/cgl78erifda00hge)