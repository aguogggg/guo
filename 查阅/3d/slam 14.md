
旋转矩阵R、变换矩阵T


Eigen
Eigen是一个 C++ 开源线性代数库。它提供了快速的有关矩阵的线性代数运算，还包括解方程等功能。许多上层的软件库也使用 Eigen 进行矩阵运算，包括 g2o、Sophus 等。
定义矩阵、动态矩阵
	转置
	各元素和
	迹
	数乘
	逆
	行列式
实对称矩阵可以保证对角化成功
	特征值
	特征向量
解方程
	求逆自然是最直接的，但是求逆运算量大
	矩阵分解来求，例如QR分解，速度会快很多


旋转向量、欧拉角
罗德里格斯公式（Rodrigues’s Formula ）
	旋转向量 -> 旋转矩阵
	也可反求
	
欧拉角

四元数
	旋转向量 -> 四元数  四元数 -> 旋转向量
	运算
		加、减、共轭、模长、逆、数乘、点乘
	四元数表示旋转
	四元数 -> 旋转矩阵   旋转矩阵 ->  四元数

欧式变换、相似变换、仿射变换、摄影变换

Eigen表示上面的各种旋转
pangolin可视化

李群与李代数
在 SLAM 中位姿是未知的，而我们需要解决什么样的相机位姿最符合当前观测数据这样的问题。一种典型的方式是把它构建成一个优化问题，求解最优的 R, t，使得误差最小化。
旋转矩阵自身是带有约束的（正交且行列式为 1）。它们作为优化变量时，会引入额外的约束，使优化变得困难。
通过李群——李代数间的转换关系，我们希望把位姿估计变成无约束的优化问题，简化求解方式。
群
群（Group）是一种集合加上一种运算的代数结构。我们把集合记作 A，运算记作 ·，那么群可以记作 G = (A, ·)

SO(3)  so(3)
SE(3)   se(3)
指数映射

求导和扰动模型

Sophus
李代数库
如何构造 SO(3), SE(3) 对象，对它们进行指数、对数映射，以及当知道更新量后，如何对李群元素进行更新


相机
相机模型
	针孔相机模型
		像素坐标
		归一化坐标
		相机坐标
		世界坐标
	内参K
	外参R,t
	畸变
		径向畸变：桶形失真、枕形失真
		切向畸变
		畸变纠正公式
			去畸变像素点(u,v)
			转换到归一化(x,y)
			经过畸变公式得到(x_d,y_d)
			经过内参得到畸变图像像素点(u_d,v_d)
			把(u_d,v_d)的内容填充到(u,v)

openCV
	读取图像
	显示图像
	遍历图像
	拷贝
拼接点云
	5对图像，深度图和彩色图
	对应的相机位姿
	点云库PCL
	对每一张图的每一个点计算世界坐标和RGB值，放进点云


非线性优化
由于噪声的存在，运动方程和观测方程的等式必定不是精确成立的。
如何在有噪声的数据中进行准确的状态估计。
在 SLAM 问题中，同一个点往往会被一个相机在不同的时间内多次观测，同一个相机在每个时刻观测到的点也不止一个。这些因素交织在一起，使我们拥有了更多的约束，最终能够较好地从噪声数据中恢复出我们需要的东西。
运动方程 上一时刻到这一时刻的位置
观测方程 这一时刻的观测值
由观测值计算这一时刻的位置，后验概率
直接求后验分布是困难的，但是求一个状态最优估计，使得在该状态下，后验概率最大化（Maximize a Posterior，MAP），则是可行的
最大化似然和先验的乘积，先验则是这一时刻位置的概率估计，似然则是这一时刻位置下的观测值的估计(条件概率)
没有了先验，则最大似然估计，即在什么样的状态下，最可能产生现在观测到的数据。

通过误差项引出分布，计算最大似然估计


迭代的方式，从一个初始值出发，不断地更新当前的优化变量，使目标函数下降。
一阶和二阶梯度法
Gauss-Newton
Levenberg-Marquadt

Ceres库面向通用的最小二乘问题的求解
g2o基于图优化的库

 
视觉里程计  VO
它根据相邻图像的信息，估计出粗略的相机运动，给后端提供较好的初始值。
如何提取、匹配图像特征点，然后估计两帧之间的相机运动和场景结构，从而实现一个基本的两帧间视觉里程计。

特征点法
SIFT  SURF  ORB
特征点由关键点（Key-point）和描述子（Descriptor）两部分组成。
关键点是指该特征点在图像里的位置，有些特征点还具有朝向、大小等信息。
描述子通常是一个向量，按照某种人为设计的方式，描述了该关键点周围像素的信息。

ORB 特征
	FAST 角点提取，ORB 中计算了特征点的主方向，为后续的 BRIEF 描述子增加了旋转不变特性。**灰度质心法**
	BRIEF 描述子，对前一步提取出特征点的周围图像区域进行描述。

特征匹配
对两幅图像中的BRIEF描述子进行匹配，使用 Hamming 距离
	暴力匹配（Brute-Force Matcher）
	快速近似最近邻（FLANN）适合于匹配点数量极多的情况


1. 当相机为单目时，我们只知道 2D 的像素坐标，因而问题是根据两组 2D 点估计运动。该问题用对极几何来解决。
2. 当相机为双目、RGB-D 时，或者我们通过某种方法得到了距离信息，那问题就是根据两组 3D 点估计运动。该问题通常用 ICP 来解决。
3. 如果我们有 3D 点和它们在相机的投影位置，也能估计相机的运动。该问题通过 PnP
求解。

对极几何
对极约束
	基础矩阵（Fun-damental Matrix）F 和本质矩阵（Essential Matrix）E
	根据配对点的像素位置，求出 E 或者 F；八点法（Eight-point-algorithm）
	根据 E 或者 F，求出 R, t。奇异值分解（SVD）
单应矩阵（Homography）H 
	描述了两个平面之间的映射关系，若场景中的特征点都落在同一平面上（比如墙，地面等），则可以通过单应性来进行运动估计。

OpenCV 提供的算法进行求解，分解得到的 R, t 一共有四种可能性。不过 OpenCV 替我们使用三角化检测角点的深度是否为正，从而选出正确的解。

尺度不确定性
对 t 长度的归一化，直接导致了单目视觉的尺度不确定性（Scale Ambiguity）
以这时的 t 为单位 1，计算相机运动和特征点的 3D 位置。这被称为单目 SLAM 的**初始化**。在初始化之后，就可以用 **3D-2D** 来计算相机运动了。初始化之后的轨迹和地图的单位，就是初始化时固定的尺度。因此，单目 SLAM 有一步不可避免的初始化。初始化的两张图像必须有一定程度的平移，而后的轨迹和地图都将以此步的平移为单位。
初始化的纯旋转问题
从 E 分解到 R, t 的过程中，如果相机发生的是纯旋转，导致 t 为零，那么，得到的E 也将为零，这将导致我们无从求解 R。单目初始化不能只有纯旋转，必须要有一定程度的平移。在实践当中，如果初始化时平移太小，会使得位姿求解与三角化结果不稳定，从而导致失败。
多于八对点的情况
当给定的点数多于八对时（比如例程找到了 79 对匹配），我们可以计算一个最小二乘解。
存在误匹配的情况时，更倾向于使用随机采样一致性（Random Sample Concensus, RANSAC）

三角测量
通过在两处观察同一个点的夹角，确定该点的距离。
噪声的影响，这两条直线往往无法相交。因此，（又）可以通过最二小乘去求解。
三角测量的矛盾，当平移很小时，像素上的不确定性将导致较大的深度不确定性。平移较大时，在同样的相机分辨率下，三角化测量将更精确。
但增大平移，会导致匹配失效；而平移太小，则三角化精度不够。
假设特征点服从高斯分布，并且对它不断地进行观测，在信息正确的情况下，我们就能够期望它的方差会不断减小乃至收敛。这就得到了一个滤波器，称为深度滤波器（Depth Filter）。


3D-2D  PnP
直接线性变换


P3P
仅使用三对匹配点
三角形相似性质
用验证点来计算最可能的解，得到 A, B, C 在相机坐标系下的 3D 坐标。然后，根据 3D-3D 的点对，计算相机的运动 R, t。
知道的是A, B, C 在世界坐标系中的坐标，而不是在相机坐标系中的坐标。一旦 3D 点在相机坐标系下的坐标能够算出，我们就得到了 3D-3D 的对应点，把 PnP 问题转换为了 ICP 问题。
 EPnP、UPnP  它们利用更多的信息，而且用迭代的方式对相机位姿进行优化，以尽可能地消除噪声的影响。
 在 SLAM 当中，通常的做法是先使用 P3P/EPnP 等方法估计相机位姿，然后构建最小二乘优化问题对估计值进行调整（Bundle Adjustment）


非线性优化   Bundle Adjustment
把相机位姿和空间点位置都看成优化变量，放在一起优化。
用它对 PnP 或 ICP 给出的结果进行优化。
在 PnP 中，这个 Bundle Adjustment 问题，是一个最小化重投影误差（Reprojection error）的问题。
考虑 n 个三维空间点 P 和它们的投影 p，我们希望计算相机的位姿 R, t，它的李代数表示为 ξ。
把误差求和，构建最小二乘问题，然后寻找最好的相机位姿，使它最小化，该问题的误差项，是将像素坐标（观测到的投影位置）与 3D 点按照当前估计的位姿进行投影得到的位置相比较得到的误差，所以称之为重投影误差。

笔记：
在单目中使用对极几何求得R和t，然后进行初始化，初始化后可以算得第一张图特征点在相机坐标系下的3d位置，然后就可以用3D-2D来计算相机运动
在有深度相机的情况下，第一张图的特征点直接读取深度，求该相机坐标系下的3d位置，利用3D-2D来计算第二张图的位姿，PnP、P3P、BA


ICP
ICP是根据前后两帧图像中匹配好的特征点在相机坐标系下的三维坐标，求解帧间运动的一种算法
求解
	线性代数SVD
	非线性优化，迭代的方式去找最优值
>对ICP的用途还是模糊


















9.3**问题**跑project 0.3
运行问题
CmakeLists.txt
set( CMAKE_CXX_FLAGS "-std=c++11 -march=native -O3" )
去掉-march=native


地图点不在当前帧的视野范围内，则删除地图点
地图点被成功特征匹配上的次数和被观测到的次数之比小于阈值，说明这个地图点的匹配特性不好，则删除地图点：被观测到的次数是在进行特征匹配的时候，首先判断地图点是否在当前相机视野内，如果在那么被观测的次数+1。被成功特征匹配上的次数是在求解PnP的时候，使用solvePnPRansac保留了正确匹配的地图点，此时地图点成功匹配的次数+1。
当前相机光心指向地图点的方向向量，和地图点被加入到地图时观测到它的那帧图像的相机光心指向地图点的方向向量，如果二者角度差大于30度，说明此时观测地图点和首次观测到这个地图点角度差比较大了，删除地图点。为什么？比较大了不应该是更准确吗？
如果当前帧匹配到的2D特征点数目太少，那么就把当前帧特征点全部反投影到世界坐标系中，生成新的地图点，并添加到地图中。
如果当前地图中的地图点数量太多了，那么就提高第二条擦除的比例，就是成功匹配的次数/观测的次数比例要大，这样才是好的地图点。
如果这个地图点不是一个好点，比如这个地图点的深度值非常不准确，那么可以根据两次观测到的特征点的匹配进行三角化得到一个更加准确的深度值，从而修正RGB-D的深度值。

ch10
ceres版本2.0.0，有一处代码需要修改
cmakelist里面添加eigen