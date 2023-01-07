---
title: "高数工本笔记"
date: "2022-08-12"
categories:
  - 学习	
toc: true
draft: true

math: mathjax
---

学习笔记: 空间几何与向量代数、多元函数微分学、重积分

<!--more-->
### 一、空间几何与向量代数

- 分清八个卦限 （略）
- 空间两点距离公式: d = $\sqrt{(x1-x2)^2 + (y1-y2)^2 + (z1-z2)^2}$
##### 向量
  - 向量的长度称为模。向量 $\vec{AB}$ 的模为 $\mid \vec{AB} \mid $ 或 $\mid\vec{\alpha}\mid$
  - 单位向量：若$\mid\vec{\alpha}\mid$ = 1，则 $\alpha$ 为单位向量
  - 零向量：若$\mid\vec{\alpha}\mid$ = 0，则 $\alpha$ 为零向量，方向看作任意的 
  - 向量平行：两个向量不看长度，方向相同或相反。
  - 向量相等：两个向量的长度和方向都相同。
##### 向量计算
  - 向量加法：参考书中平行四边形
  - 数量积：两个向量的模与其夹角的余弦值之积 
    - 公式：$\alpha \cdot \beta$ = $\mid\alpha\mid \cdot \mid\beta\mid \cos\theta$ 
    - 坐标公示：$\alpha \cdot \beta$ = $\alpha_x \cdot \beta_x + \alpha_y \cdot \beta_y + \alpha_z \cdot \beta_z$
    - 注意数量积符号是 $\cdot$ 而不是 $\times$  
    - 向量垂直，数量积为0
##### 平面方程
  - 方程：平面过点$(x,y,z)$，其法向量为 $n = \{A, B, C\}， 则Ax + By + Cz + D = 0$
  - 变形：过两个点，一个法向量：$A(x-x_0) + B(y-y_0) + C(z-z_0) + D = 0$ 
  - 点到平面距离 $d = \frac{|Ax + By + Cz + D|}{\sqrt{A^2 + B^2 + C^2}}$
  - 当D=0时，表示过原点的平面
  - 当A=0时，表示平行于x轴的平面
##### 直线方程
  - 直线过点 $P_0(x_0,y_0,z_0)$ ，方向向量为$n = \{A, B, C\}$
  - 对称式方程：$\frac{x-x_0}{A} = \frac{y-y_0}{B} = \frac{z-z_0}{C}$
##### 二次曲面

##### 习题
  - 判断点$(-1,2,-5)$所在卦限
  - 点$(1,-3,2)$关于x轴对称的点。 （x坐标不变，其余坐标取相反数。(1,3,-2)）
  - 向量 $\alpha = \{3,2,\frac{1}{2} \}, \beta = \{-1,1,0\}$，则 $2\alpha \cdot \beta$ =
  - 两个向量的标$\{8,-4,1\}、\{2,2,1\}$，求两个向量的夹角。 （套用两个数量积公式, 1/3）
  - 点$(2,3,7)$到平面 $2x+2y-z+6=0$ 的距离。（得出法向量{2,2,-1}，套用公式）




### 二、多元函数微分学
<!-- - 一元函数中，可导必连续。多元函数中，偏导数存在不一定连续
- 
- 函数可微 <=> 可导
- 函数单调性，求函数导数，若 >= 0，单调递增
- 导数为0的点称为驻点。
- 分清极大值和最小值不是最大值或者最小值
- 全微分：多元函数的导数，即偏导数的集合，记为 $\frac{\partial f}{\partial x_1, \partial x_2, \partial x_3, ...}$，其中 $x_1, x_2, x_3, ...$ 为自变量，$f$ 为函数，$\partial$ 为偏导数符号。 -->
##### 符号含义
- 导数符号 $d$
- 偏导数符号 $\partial$
- 定义域 D

##### 二重极限
- 若函数$f$无限接近点$P(x_0,y_0)$时，值无限接近常数$A$， 即 $\lim\limits_{(x,y) \to (x_0,y_0)} f(x,y) = A$，则$A$是$f$在点$P$处的二重极限。（极限值常数$A$与函数$f$在点$P$处是否有定义无关，只与该点的邻域有关。）
- 重要极限  
  1. $ lim\frac{sin t}{t} = 1$ 
  2. 
##### 偏导数
  - 说明：若$ \lim\limits_{\Delta x \to 0} \frac{f(x_0 + \Delta x,y_0) - f(x_0,y_0)}{ \Delta x }$ 存在，此极限值为函数 $f(x,y)$ 在点 $P(x_0,y_0)$ 处关于自变量 $x$ 的偏导数。  （反应函数$f$在点$P$处沿$x$轴的变化率）
  - 个人理解：像物理里的加速度，在于变化率。
    - 一辆车要拐弯，形成的曲线，不能是个V, 而是个U，在U中的低谷点偏导数就是0。
  - 写法：
    - 函数$z=f(x,y)$关于x轴的偏导数写为 $\frac{\partial z}{\partial x}$ 或 $z_x$ 。
    - 函数$z=f(x,y)$在点$(x_0,y_0)$处关于x轴的偏导数：$z_x(x_0,y_0)$ 或 $f_x(x_0,y_0)$ 
  - 注意 $\frac{\partial z}{\partial x}$ 应该看作一个整体，分子分母不具有独立的意义。（不同于导数 $\frac{d f}{d x}$ 分子分母有独立的意义）
  - 例1：$f=x^9+y^2$，求 $f_y(2,1)$。
    - 关于$y$的导数，就把$x$看作常数，直接求导。
    - $f_y(2,1)=2y=2$ 
  - 例2: $f=(y-1)\arctan(x^2+y^3) + x^2y$，求 $f_x(2,1)$
    - 如果用例1的方式很麻烦，理解二元函数偏导数的本质是先转为一元函数求导。
    - 关于$x$的导数，先把$y=1$带入，再求导。
    - $f'=2x$，再把$x=2$带入，得到 $f_x(2,1) = 4$
##### 复合函数偏导数
- 例1: 求$u=\sqrt{x^2+y^2+z^2}$的偏导数
  - 将 $x^2+y^2+z^2$ 看作 $v$，则 $u_x = u_v \cdot v_x $。
  - $ u_x = \frac{1}{2\sqrt{x^2+y^2+z^2}} \cdot 2x = \frac{x}{u}$， 同理 $u_y = \frac{y}{u}$，$u_z = \frac{z}{u}$

##### 高阶偏导数
  - 关于x的二阶偏导数：$\frac{\partial}{\partial x} \cdot \frac{\partial z}{\partial x}$ ，写作 $\frac{\partial ^2 z}{\partial x^2} $ 或  $f_{xx}(x,y)$ 或 $z_{xx}$
  - 偏x偏y：$\frac{\partial ^2 z}{\partial x \partial y}$ 或 $f_{xy}(x,y)$ 或 $z_{xy}$
###### 全微分
  - 的

##### 隐函数的偏导数
- 二元函数 $F(x,y)$ 隐函数 $y_0=f(x_0)$，则有公式：$\frac{d_y}{d_x} = -\frac{F_x}{F_y}$
- 三元函数 $F(x,y,z)$ 隐函数 $z_0=f(x_0,y_0)$，则有公式：$\frac{\partial z}{\partial x} = -\frac{F_x}{F_z}$，$\frac{\partial z}{\partial y} = -\frac{F_y}{F_z}$
- 例1：$x^2+y^2=R^2$,求隐函数的$y=f(x)$确定的导数$\frac{d_y}{d_x}$
  - 令$F(x,y)=x^2+y^2-R^2$ 。（这里使用大F)
  - $\frac{d_y}{d_x} = -\frac{F_x}{F_y} = -\frac{x}{y}  $
- 例2: 
##### 多元函数的极值和最值
- 极值：极大值和极小值，对应的点叫做极值点。 
  - 极值是一个局部性的概念，不一定是最值，极大值不一定比极小值大。
  - 极值一定是内点，不是边界点。
  - 极值点可能偏导数不存在，举例V。

- 驻点：函数$z$，存在点 $z_x=0,z_y=0$，称为驻点。
  - 驻点不一定偏导数存在，举例V。
  - 驻点不一定是极值点，
- 用函数$z=f(x,y)$的驻点找极值点 
  - 找驻点，令$z_x=0, z_y=0$，此时能得到几个驻点坐标
  - 二阶偏导，$A=z_{xx}$, &emsp; $B=z_{xy}$, &emsp; $C=z_{yy}$
  - 根据$\Delta = B^2-AC $，把几个驻点分别带入
    - 若$\Delta > 0$，不是极值点 
    - 若$\Delta < 0 且 A < 0$，是极大值点。
    - 若$\Delta < 0 且 A > 0$，是极小值点。
- 例：求$f(x,y)=x^3-y^3-3x^2+27y$的极值。
  - 找驻点。令$f_x(x,y)=0$,&emsp; $f_y(x,y)=0$，
  - 计算$f_x(x,y)=3x^2-6x = 0, f_y(x,y)= -3y^2+27 = 0$
  - 得到$x(x-2)=0,(3-y)(3+y)=0$，得到四个驻点：$(0,-3),(0,3),(2,-3),(2,3)$
  - 计算二阶偏导数。$A=6x-6$, &emsp; $B=0$, &emsp; $C=-6y$
  - 根据定理：$\Delta = B^2-AC $，分别带入四个驻点列出
  - 则$(0,3)$是极大值点，$(2,-3)$是极小值点。


- 例题：
  - $\lim\limits_{x \to 0, y \to 0} \frac{2-\sqrt{xy+4}}{xy}$ , 解析：设$xy=u$, 答案：$-1/4$


### 三、重积分

##### 二重积分
- 定积分符号 $\int$，二重积分符号 $\iint$，西格玛 $\sigma$
- 二重积分公式：$\iint\limits_D f(x,y)d\sigma$
  - 其中D是积分区域， $f(x,y)$叫做被积函数，$x,y$叫做积分变量，$d\sigma$叫面积元素
  - 二重积分的几何意义是被积区域的面积
- 性质：
  1. 常数可以提到积分外。$\iint\limits_D k f(x,y)d\sigma = k \iint\limits_D f(x,y)d\sigma$
  2. 函数和或差的积分，等于积分的和或差。$\iint\limits_D [f(x,y) \pm g(x,y)]d\sigma = \iint\limits_D f(x,y)d\sigma \pm \iint\limits_D g(x,y)d\sigma$
  3. 区域可加性。$D=D_1+D_2$，则$\iint\limits_D f(x,y)d\sigma = \iint\limits_{D_1} f(x,y)d\sigma + \iint\limits_{D_2} f(x,y)d\sigma$
  4. 单调性。若 $f(x,y)>g(x,y)$，则$\iint\limits_D f(x,y)d\sigma > \iint\limits_D g(x,y)d\sigma$
  5. 若$f(x,y)\geq0$, 则$\iint\limits_D f(x,y)d\sigma \geq 0$
  6. 若在$D$上$a<f(x,y)<b$，则 $ a|D| < \iint\limits_D f(x,y)d\sigma < b|D|$， ($|D|$为$D$的面积)
  7. 当$f(x,y)=1$时，$\iint\limits_D 1d\sigma = |D|$
  
##### 直角坐标下的二重积分
- 一般计算方式：
  - 若D可以用不等式 $a \leq x \leq b, \phi_1(x) \leq y \leq \phi_2(x) $ , 则 $\iint\limits_Df(x,y)d_xd_y = \int_a^b d_x \int_{\phi_1(x)}^{\phi_2(x)} f(x,y) d_y $，此为X型区域，从下边界到上边界。
  - 若D可以用不等式 $c \leq y \leq d, \phi_1(y) \leq x \leq \phi_2(y) $ , 则 $\iint\limits_Df(x,y) d_x d_y = \int_c^d d_y \int_{\phi_1(x)}^{\phi_2(x)} f(x,y) d_x $，此为Y型区域，从左边界到右边界。
  - 例：求二重积分$\iint\limits_D xy d_x d_y$，其中D是 $y=x^2$和$y=x$围成的区域。（1/24）
- 奇偶性计算方式
  - 若被积函数$f(x,y)$是关于$x$的奇函数，对于任何$y$都有$f(-x,y)=-f(x,y)$，则二重积分$\iint\limits_D f(x,y)dxdy = 0$
  - 若被积函数$f(x,y)$是关于$x$的偶函数，对于任何$y$都有$f(-x,y)=-f(x,y)$，则二重积分$\iint\limits_D f(x,y)dxdy = 2\iint\limits_{D左} f(x,y)dxdy = 2\iint\limits_{D右} f(x,y)dxdy$

##### 极坐标下的二重积分

##### 三重积分
- 公式：$\iiint\limits_\Omega f(x,y,z)d\nu	$ 
  - 三重积分的几何意义是被积区域的体积
- 例：计算三重积分 $ I = \iiint\limits_\Omega xd_xd_yd_z	$ ，其中$\Omega$是由三个坐标面及平面$x + 2y + z = 1$围成。 （1/48）*

