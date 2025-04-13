---
title: Hexapod Robot
date: 2025-02-25T14:33:50.101Z
tags: [笔记, Robotics, hexapod]
comments: true
draft: false
summary: 六足机器人学习笔记。
category: 笔记
---

在b站看到六足机器人的教学，记录一下学习笔记。对正逆运动学建立基本认知，并在webots中对六足机器人及控制器进行了复现，很有收获。

## 运动学

### 正向运动学（forward kinematics）

六足机器人的足由三段组成，旋转部件为髋（hip）也叫做关节（joint），尖端位置为尖（tip）。三段的旋转角度决定了一条腿的姿态（pose），可以用坐标变化，从世界坐标系开始，得到最终的尖端位置的坐标。正向运动学公式如下：

$$
W->P_0: R_z=\alpha => M_0 =
\begin{bmatrix}
R_x(\alpha) & T \\
0 & 1 \\
\end{bmatrix} \\
P_0->P_1: R_x=\beta, T_y=L_1 => M_1 =
\begin{bmatrix}
R_x(\beta) & [0, L_1, 0]^T \\
0 & 1 \\
\end{bmatrix} \\
P_1->P_2: R_x=\gamma, T_y=L_2 => M_2 =
\begin{bmatrix}
R_x(\gamma) & [0, L_2, 0]^T \\
0 & 1 \\
\end{bmatrix} \\
P_2->P_3: T_y=L_3 => M_3 =
\begin{bmatrix}
I & [0, L_3, 0]^T \\
0 & 1 \\
\end{bmatrix} \\
$$

这种已知三个角度来求解P3位置的叫做正向运动学。

### 逆向运动学（inverse kinematics）

相应的，知道P0、P3的位置，求解三个角度，叫做逆向运动学，逆向运动学往往更加重要。

#### 自由度高度受限的简单方法

在实际的机器人设计中，往往知道的是尖端的运动轨迹，比如让足抬起来走一步，要通过这个轨迹解算舵机的旋转角度，从而达到控制目标。
整条腿按照$\alpha$角度转动，可将腿部平面简化为二维，在知道P3的坐标时，通过三角函数可求得各点坐标以及关节旋转角度。
[![pE17mgU.png](https://s21.ax1x.com/2025/02/25/pE17mgU.png)](https://imgse.com/i/pE17mgU)

#### 更通用的简单方法

[IKPY](https://pypi.org/project/ikpy/)是一个开源的逆向运动学库，可以用来求解六足机器人的逆运动学。通过[urdf](http://wiki.ros.org/urdf)文件，可以方便地定义机器人的结构，并借助IKPY求解逆运动学。
需要注意的是urdf文件的建模里各个关节尺寸、坐标系、旋转方向等需要与webots建模中关节的旋转方向一致，不然解ik的时候会得到错误的解。

## 仿真

当整合成机器人时，如何通过给定的姿态逆运算腿的关节角度？
本质上，姿态的改变是机器人体接触点P0空间位置的改变，假定地接触点P3不变时，可通过P0 P3坐标，通过逆运动学求解。
但实际上，地面接触点未必不发生变化，因此在得到关节角度后，进行正向运动学时，还需要添加一些物理约束让运动合理。

基本约束，如连接所有地面接触点，得到的多边形可称为当前姿态的支撑多边形（support polygon），要保证支撑多边形所在平面与地面重合。

---

教程视频地址：[六足机器人拆解：从数学原理到计算机仿真](https://www.bilibili.com/video/BV1qF41167Sx?spm_id_from=333.788.videopod.sections&vd_source=50b7fbaac8495676da2c0ff3d4eb7885)

教程仓库地址：https://github.com/XuelongSun/HexapodWebots

复现仓库地址：https://github.com/timebiu/Hexapod-Robot
