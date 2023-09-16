---
layout: formal_post
title:  'Paper Reading: Ego-planner'
date:   2023-09-16 
description: Sept
tags: formatting links
categories: sample-posts
related_publications: enwiki_sdf， 
bibliography: 2023-09-16-paper_reading_ego_planner.bib
giscus_comments: true
featured: true
toc:
  - name: Abstract
  - name: Basic Knowledge
  - name: Introduction
#     subsections:
#       - 
  - name: Citations
---



<div class="col-sm mt-3 mt-md-0">
        {% 
        include figure.html 
        path="assets/img/posts/ego_planner/ego_planner_head.png" 
        class="img-fluid rounded z-depth-1" 
        zoomable=true%}
</div>

# EGO-Planner
## An **E**SDF-free **G**radient-based L**o**cal Planner for Quadrotors

### EGO-Planner 算法摘要：
1.  通过比较碰撞轨迹与无碰撞引导路径，得到惩罚函数中的碰撞项
2.  轨迹优化器只提取当前轨迹撞到的障碍物的信息
3.  如果某段轨迹动力学不可行，则延长该段轨迹分配的时间
4.  异性曲线拟合算法——在保持原有轨迹形状的情况下降低轨迹的阶数

## Abstract：

基于梯度的规划器广泛用于四旋翼本地规划，其中**`欧几里得有符号距离场（ESDF）`**对于评估梯度大小和方向至关重要。然而，计算这样的场会存在很多冗余，因为轨迹优化过程仅覆盖ESDF更新范围的一个非常有限的子空间。本文提出了一种**`无需ESDF的基于梯度的规划框架`**（ESDF-free gradient-based planning framework），大大减少了计算时间。主要改进在于通过将`碰撞轨迹`与`无碰撞引导路径`进行`比较`来制定惩罚函数中的`碰撞项`。**仅当轨迹与新障碍物碰撞时，才会存储生成的障碍信息，使规划器仅提取必要的障碍信息。然后，如果违反了动力学可行性，我们会延长时间分配。引入一种各向****`异性曲线拟合算法`****，以调整轨迹的高阶导数，同时保持原始形状。**基准比较和实际实验验证了其鲁棒性和高性能。源代码以ROS软件包形式发布。


## Basic Knowledge

1. 距离函数 -> 有符号距离函数 -> 欧氏有符号距离函数
2. 距离场 -> 有符号距离场 -> 欧氏有符号距离场 

### 1 距离函数

#### 有符号距离函数 Signed Distance Functions (SDF) <d-cite key="enwiki_sdf"></d-cite>

- 输入：度量空间中的点 $$x$$，度量空间的 子集 $$\Omega$$。

- 输出：有符号距离 $$ f(x， \Omega) $$，点在内部为正，点在外部为负。

$$
f(x)=\left\{
        \begin{array}{ll}
        d(x, \partial \Omega) & \text { if } x \in \Omega \\ 
        -d(x, \partial \Omega) & \text { if } x \in \Omega^{c}
        \end{array}
        \right.
$$

### 2 距离场

<div class="col-sm mt-3 mt-md-0">
        {% 
        include figure.html 
        path="assets/img/posts/ego_planner/sdf_world_visualizations.png" 
        class="img-fluid rounded z-depth-1" 
        zoomable=true
        %}
</div>
<div class="caption">
Input mask and two visualizations of the resulting SDF. <d-cite key="distance_fields_blog"></d-cite>
</div>

`距离场(Distance Field)`是图形学中的一个重要概念，是一个与物体表面距离的标量函数。对于二维或者三维空间中的一个闭合曲面S，距离场d(x)定义为从空间中任意一点x到曲面S的最近距离。

1. 距离场可以用于快速计算一个点到曲面上的最近点，只需要沿着梯度方向移动一个距离 $$d(x)$$ 就能到达。这可用于三维模型的碰撞检测等。
2. 距离场可以用于构建隐式表面(Implicit Surface)，即 $$d(x)=0$$ 的等值面。这可以表示比较复杂的三维模型。
3. 可以根据距离场构建等距曲线或等距面，得到三维模型的轮廓。这可用于图像处理中的形态学操作。
4. 可以用距离变换把图像转换为距离场表示，然后进行模糊、膨胀腐蚀等操作。这可用于图片修复去噪等处理。
5. 可以利用距离场进行实时渲染，特别是在GPU上进行并行计算，可以大大提升渲染效率。

#### (欧氏) 有符号距离场 (Euclidean) Signed Distance Field

采用有符号距离函数作为距离函数 $$d$$ 所建立的距离场。




## Introduction

1. 四旋翼无人机在线（且局部）的路径规划方法 极大地拓展了无人机的自主性边界；
2. 众多轨迹生成与优化方法中，基于梯度的方法通过**平滑轨迹**并利用**梯度信息**，能有效地提高轨迹的可行性，显示出巨大的潜力且越来越受欢迎。
3. 传统的基于梯度的路径规划器：
   1. 用预先构建的`欧氏有符号距离场`地图来计算梯度的大小和方向
   2. 使用数值优化方法生成局部最优解
   3. 优化程序能快速收敛
   4. 预先计算ESDF地图占据了路径规划总时间的70%，已经成为瓶颈
   5. 常用方法：`增量全局更新方法` <d-cite key="FIESTA"></d-cite> 与 `批处理局部计算方法` <d-cite key="distance_transforms"></d-cite>，但都没有关注轨迹本身，计算了许多对规划没有贡献的`ESDF`值。
   6. 在一般的自主导航情境中，无人机只需要避免局部碰撞，无需建立全局的`EDSDF`地图
4. 本文提出了一种无需`ESDF`、具有碰撞检测和轨迹优化功能的、基于梯度的局部路径规划框架，成为`EGO`。
   1. 通过平滑度、碰撞和动力可行性等项进行轨迹优化。通过将`障碍物内的轨迹`与`引导的无碰撞路径`进行比较来对碰撞成本进行建模。
   2. 将 `避碰力` 投影到碰撞轨迹上，生成预估的 `梯度` 以将轨迹包裹在障碍物外部。
   只在必要时计算梯度，并避免在与局部轨迹无关的区域计算ESDF。
   3. 若生成的轨迹不满足要求，则启动后续优化，即给更多的时间进行轨迹优化。
   4. 首次实现了无 `ESDF` 的基于梯度的局部路径规划方法，使计算时间降低了一个数量级。
   5. 进行了全面的仿真与实际测试。
5. 本文主要贡献：
   1. 我们提出了一种轻量级但有效的轨迹优化算法，通过采用各向异性误差惩罚来建模轨迹拟合问题，从而生成更加平滑的轨迹。
   2. 我们将提出的方法集成到一个完全自主的四旋翼系统中，并将我们的软件发布给社区参考。
   3. 路径陷入了一个局部最小值，这是非常常见的情况，因为相机无法看到障碍物的背面。

<div class="col-sm mt-3 mt-md-0">
        {% 
        include figure.html 
        path="assets/img/posts/ego_planner/ego_planner_1_limited_space_of_ESDF.png" 
        class="img-fluid rounded z-depth-1" 
        zoomable=true
        %}
</div>

