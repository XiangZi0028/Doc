## 向量计算公式

$$
\begin{align*}
&\vec{a}=(x_1,y_1,z_1)^T \qquad(1)\\
&\vec{b}=(x_2,y_2,z_2)^T  \qquad(2)\\
&\vec{a}·\vec{b}=x_1*x2+y_1*y_2+z_1*z_2 \qquad(3)\\
&\vec{a}·\vec{b}=|\vec{a}|·|\vec{b}| ·cos<\vec{a},\vec{b}> \qquad(4)\\
&\vec{a}×\vec{b}=\begin{pmatrix}y_1z_2-y_2z_1\\z_1x_2-x_1z_2\\x_1y_2-y_1x_2
\end{pmatrix}=\begin{pmatrix}0 &-z_1 &y_1\\z_1 &0 &-x_1\\-y_1 &x_1 &0\end{pmatrix}\begin{pmatrix}x_2\\y_2\\z_2\end{pmatrix} \qquad(5)
\end{align*}
$$

点乘的几何意义：向量夹角的余弦值

叉乘的几何意义：

 1. 左侧还是右侧，逆时针还是顺时针

    主要看叉乘结果的正负

    ![image-20220504000650160](..\101Pic\1.png)

 2. 内侧还是外侧

    p点一直在三条边的左边还是右边

![image-20220504000805567](..\101Pic\2.png)

- 基础变换（仿射变换）：旋转、缩放、切变、平移
  - 先后顺序：缩放一定要在平移的前面进行，不然缩放会影响平移的量
  - 仿射变换=线性变换+平移
  - 线性变换=旋转+缩放+切变

- 齐次坐标系：n维的点/向量，用[n+1 x 1]的列向量表示
  - 变换用[n+1 x n+1]的矩阵左乘来表示
  - 多个变换可以组合起来变成一个矩阵→加速计算

旋转：

​	欧拉角和四元数

MVP：三个空间变换矩阵

M：世界坐标系下有很多Object，用一个变化矩阵把它们的顶点坐标从Local坐标系转换到世界World坐标系。

推演以2D为例：

（1）首先是缩放，当我们在对一个向量进行缩放，就是对向量的长度进行缩放，但是保持向量的方向不改变，如图所示，将图(1)-1变换到图(1)-2，我们可以发现我们只需要把图(1)-1中的每个点的坐标变换到图(1)-2上，也就是我们只需要把对应的向量长度进行相应的缩小就可以达到，只需要按照下面的方式转换 我们把它改写成矩阵运算的形式。
$$
\begin{pmatrix}x'=ax\\y'=by\end{pmatrix}=>
\begin{pmatrix}x'\\y'\end{pmatrix}=
\begin{bmatrix}a &0\\0 &b\end{bmatrix}
\begin{pmatrix}x\\y\end{pmatrix}
$$
​	（2）旋转，其实很容易理解，如图所示，图(2)-1中的立方体逆时针旋转θ°，之后得到图(2)-2所示，同样的道理我们只需要找到旋转之后的坐标和旋转之间的关系，那么我们就能通过表达式的形式，写出这个变换的过程。
$$
\begin{matrix}x=rcosφ \\
			   y=rsinφ\end{matrix}且
\begin{matrix}x'=rcos(φ+θ) \\
			   y'=rsin(φ+θ)\end{matrix}
$$
通过三角函数展开，我们将得到：
$$
\begin{matrix}x'=rcos(φ+θ)=rcosφcosθ-rsinφsinθ=xcosθ-ysinθ \\
			  y'=rsin(φ+θ)=rsinφcosθ+rcosφsinθ=xsinθ+ycosθ\end{matrix}
$$
把它转换成矩阵相乘的形式：
$$
\begin{pmatrix}x'\\y'\end{pmatrix}=
\begin{bmatrix}cosθ &-sinθ\\sinθ &cosθ\end{bmatrix}
\begin{pmatrix}x\\y\end{pmatrix}
$$
​	（3）移动也是，同样的道理，我们需要把图(3)-1平移到(3)-2中去，也就是我们只需要把x都朝x正方向平移tx个单位，y都朝y正方形平移ty个单位。这样就有如下的式子：
$$
\begin{pmatrix}x'=x+t_x\\y'=y+t_y\end{pmatrix}
\begin{matrix}?\\==>\end{matrix}
\begin{pmatrix}x'\\y'\end{pmatrix}=
\begin{bmatrix}a &b\\c &d\end{bmatrix}
\begin{pmatrix}x\\y\end{pmatrix}
$$
​	根据上面缩放和旋转的经验，我们希望统一表示成 Mat * Vec的形式，这样就能够统一进行管理了，比如当你把一个物体进行若干的缩放、旋转、平移操作就可以写成一个Mat*Vec进行表示了，但是很显然在这里2D世界中，平移无法表示成Mat2x2 *Vec2，怎么办呢，这个时候我们就可以引入一个新的概念，齐次坐标。

​	什么是齐次坐标呢？也就是在向量的后面加上一个标志位，这个标志位是干嘛的呢，他可以用来表示我们这个Vec是表示一个点还是一个向量。同时我们还得让它满足向量的运算法则才行。

齐次坐标系中的规则：
$$
\begin{align}
(x,y,0)代表一个向量\qquad\quad(x,y,1)代表一个点\\
(ax,ay,a)(a!=0)和(x,y,1)代表的是同一个点
\end{align}
$$
根据上面的规则我们可以发现：

​	point-point=Vec；point+vec=point；vec+vec=vce；point+point=midPoint；

有了这个这个齐次坐标系之后，我们可以把之前的缩放和旋转矩阵根据矩阵运算规则进行改写，同时我们能够写出平移矩阵的式子出来，它们如下：
$$
S:\begin{matrix}
\begin{pmatrix}x'\\y'\\1\end{pmatrix}=
\begin{bmatrix}a &0 &0\\0 &b &0\\0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\1\end{pmatrix}
\end{matrix}\qquad
R:
\begin{matrix}
\begin{pmatrix}x'\\y'\\1\end{pmatrix}=
\begin{bmatrix}cosθ &-sinθ &0\\sinθ &cosθ &0\\0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\1\end{pmatrix}
\end{matrix}\qquad
T:
\begin{matrix}
\begin{pmatrix}x'\\y'\\1\end{pmatrix}=
\begin{bmatrix}1 &0 &t_x\\0 &1 &t_y\\0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\1\end{pmatrix}
\end{matrix}\qquad
$$
​	这样我们要对一个二维物体进行连续的变换，只需要将它每个点都不停的左乘相应的Mat3x3就可以达到效果了。不过这里要注意一个问题：一般来说我们需要先进行缩放操作，然后再进行平移操作，为什么呢？

​	这里假如我们希望一个物体能像x轴的正方向平移3个单位，并且物体的大小放大两倍。如果我们先做平移操作，那么物体x‘=x+3，x’‘=2*(x')=2*(x+3)=2x+6; 可以发现被平移的距离也被放大了2倍。在这个地方这不是我们想要的效果。

​	那么到此，2D的变换我们就说完了，现在让我们看看3D的情况下模型的变换有该怎么做，和2D一样。缩放的话我们只需要把每个点的坐标都放大或缩小成对应的倍数也就是如下所示：
$$
S:\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}a &0 &0 &0\\0 &b &0 &0\\0 &0 &z &0\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}\qquad
$$
​	3D的旋转就相对更复杂一些，不过我们可以分步来看它，我们可以把它看作绕x，y，z轴进行了三次旋转，这样的话就相对会简单一些，比如绕z轴（z不变）逆时针旋转θ角的话，我们可以把它写成：
$$
R_z:
\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}cosθ &-sinθ &0 &0\\sinθ &cosθ &0 &0\\0 &0 &1 &0\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}\qquad
$$
同理如果我们绕x轴(x不变)逆时针旋转θ角的话，如下：
$$
R_x:\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}1 &0 &0 &0\\0 &cosθ &-sinθ  &0\\0 &sinθ &cosθ &0\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}\qquad
$$
如果我们绕y轴(y不变)逆时针旋转θ角的话，如下：
$$
R_y:\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}cos(-θ) &0 &-sin(-θ) &0\\0 &1 &0 &0\\sin(-θ) &0 &cos(-θ)  &0\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}=\begin{bmatrix}cosθ &0 &sinθ &0\\0 &1 &0 &0\\-sinθ &0 &cosθ  &0\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
$$
这里有点不同，我们可以看见这里逆时针旋转θ角的时候式子中的θ写成了-θ。

在笛卡尔坐标系中我们XYZ轴按照xyz的顺序，X×Y=Z,Y×Z=X；但是Z×X=Y，好吧我解释不清楚了，描述不出来，后续补一下再描述吧。

平移矩阵就很好描述了：
$$
T:
\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}1 &0 &0 &t_x\\0 &1 &0 &t_y\\0 &0 &1 &t_z\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}\qquad
$$
最终我们把这个模型变换矩阵可以描述为：
$$
MatModel:
\begin{matrix}
\begin{pmatrix}x'\\y'\\z'\\1\end{pmatrix}=
\begin{bmatrix}a &b &c &t_x\\d &e &f &t_y\\g &h &i &t_z\\0 &0 &0 &1\end{bmatrix}
\begin{pmatrix}x\\y\\z\\1\end{pmatrix}
\end{matrix}\qquad
$$
V：我们看到的画面由摄像机捕捉，摄像机参数决定了我们在屏幕上看到的东西，这一步可以将世界坐标系转换到摄像机坐标系。

​	我们在模型缩放、旋转、平移都都是在模型本身的空间进行的，什么意思呢，也就是以模型自己的坐标系来进行的，比如你自己是一个模型，你可以定义你身体的某个部位为坐标原点，那么其它部位的坐标就是相对与你自己的坐标原点而言的。

​	那么视图变换又是什么呢，通俗的说就是把模型移动到照相机的世界里面去，也就是求出模型的每个点在相机世界里面的位置。

​	在3D世界中我们应该如何定义一个相机空间呢，首先我们要知道相机look at哪个方向，然后要知道相机朝上的是哪个方向，通过这两个方向我们就可以定义一个相机空间。

​	比如现在在p(0,0,0,1)点有一个相机，它聚焦与点(0,0,1,1)，它向上的方向是(0,1,0,0)。那么根据这3个数据我们可以计算出相机空间的x，y，z三个向量轴。然后我们只需要把世界坐标的原点移动到相机空间的远点，然后再把世界坐标的x、y、z三个轴旋转到相机空间的x，y，z三个轴，就完成了视图变换了。

P：摄像机坐标系，视锥体，再规整一下

​	世界中的物体都是三维的，我们应该如果让这些三维的物体呈现再2D的屏幕上并且给我们3D的效果呢？想实现这样的效果就需要我们的投影变换，投影变换也分为2种，一种是正交投影，一种是正交投影。

​	所谓的正交投影也就是垂直投影，换句话来说也就是，物体投射到屏幕上的大小不会受物体的远近影响。正交投影矩阵：

​		正交投影举证的推导，首先我们需要定义一个视线范围，然后将这个范围压缩到[-1,1]^3（opengl坐标系统）。

屏幕由像素点构成，是离散的，二维的

计算机生成图像中，最基本的二维元素是三角形，对其的离散化操作就是生成图像的重点

如何离散化/光栅化采样。

一个像素点对应一个坐标点，对这个坐标点采样，判断它在不在三角形里面，

采样的缺点：以点代面，有失偏颇 → Aliasing 走样，表现为锯齿

Sampling Artifacts (Errors / Mistakes / Inaccuracies) in Computer Graphics：

- Jaggies (Staircase Pattern)：an example of “aliasing” – a sampling error
  - 空间 上信号变化太快，频率太高，但采样频率太慢（Signals are changing too fast (high frequency), but sampled too slowly）
- Moiré Patterns in Imaging 摩尔纹
- Wagon Wheel Illusion (False Motion)：倒着转的轮子
  - 时间上：信号变化太快，频率太高，但采样频率太慢

## GPU图形管线

![27](..\101Pic\27.png)

- API命令（以GL为例）：

OpenGL它是隐式图形API，我们不知道这些API内部到底干了些什么，或许经过这3步操作之后数据也并没有写到GPU缓存中去。或许底层驱动的实现会根据 `glBufferData(GLenum target, GLsizeiptr size, const void *data, GLenum usage) `的最后一个参数usage来决定数据存储的位置，以及数据什么时候被写如到GPU缓存中去。

![image-20220314173933026](..\101Pic\28.png)

​	在应用程序中我们还要负责对数据进行解释。举个例子吧，如下图所示，每一行代表一个顶点，也就是一共有6个顶点，每个顶点包含了它的pos、uv、normal三个数据，总共n个浮点数，如果我们不对这些数据进行说明，那你的流水线输入头也不知道该如何来定义这些数据，他就只能按照它自己的理解来进行解释了。但是这不一定是我们想要的结果，所以我们需要在应用程序中自己组织这些数据，比如那几个数表示的是pos，哪几个点表示的是uv，哪几个点表示的normal。明确这些意义之后我们就可以很好的解释vertexBuffer中的数据了。如下所示：![image-20220314174538005](..\Doc\Graphic\pic\image-20220314174538005.png)

- 顶点着色

  - 首先我们的数据依次的进入到我们的顶点着色器中，一般情况下我们在顶点着色器中会进行三种变换，也就是模型变换、视图变换和投影变换。这是为了模型的坐标转移到GL的世界坐标系中去。
  - Model, View, Projection transforms
  - Shading, Texture mapping
  - Output: Vertex Stream

- 图元装配

  - 问题来了，这些经过顶点着色器冲洗的顶点就是一个点呀，最终我们要在屏幕上看到的肯定不能是这么一个一个的点呀，所以下一步我们就要把这些数据按照我们的在应用程序中的设置给组装起来，在应用程序中我们会调用`glDrawArrays(GLenum mode, GLint first, GLsizei count)`或者`glDrawElements(GLenum mode, GLsizei count, GLenum type, const void *indices)`来发起绘制命令，其中第一个参数就是mode就是指定图元的类型。可以是点、线、三角形等。
  - Output: Triangle Stream

- 裁剪

  - 以图元是三角形为例，那么现在我们就拿到了三个点Vertex[0]、Vertex[1]、Vertex[2]，这三个点会组成一个平面，之后我们会进行平面进行裁剪，裁剪的主要目的也是说减少计算吧，对于哪些不在GL [-1,1]^3空间中的部分，或者处于物体背面看不见的部分(`glEnable(GL_CULL_FACE)`)，我们的流水线也没有必要对它进行加工，如果不进行裁剪，那比如所有的三角形都在可视空间之外，那我们的GPU就可以闲置休息，没必要做无用功。平面经过裁剪剩下的部分是我们最终需要渲染的部分，那么我们如何把这个平面转化成我们屏幕上的点呢。
  - Sampling
  - Output: Fragment Stream

- 光栅化

  - 首先是屏幕映射，也就是把每个点映射到屏幕上去。这里后面我会用我写的软渲染器的代码来进行描述。经过屏幕映射之后又进行了三角形的设置。但是为什么这里又要进行图元的设置。

    ![image-20220314181742049](..\101Pic\29.png)

    根据上图所示有两个经过各种变换三角形平面，从图2中我们可以看到上面的三角形有一个顶点会落到屏幕之外，之前我们说到，流水线中会对三角平面进行裁剪，裁剪之后如果我们补重新设置三角形的话，那么将会如图4所示，所有这也是我认为此次图元设置的意义是什么。时候我们要进行三角形遍历。

  - Z-Buffer Visibility Tests

  - Shading, Texture mapping

  - Output: Shaded Fragments

- 片段着色器

  - 遍历的内容主要是我们映射到屏幕上的像素点，每个像素点都会经过片段着色器，片段着色器和顶点着色器一样，需要我们自己来编写shader，在片段着色器中我们可以计算每个点的光照，也就是最终呈现到我们平屏幕上的颜色。这些像素会经过各种测试，比如深度测试(`glEnable(GL_DEPTH_TEST)`)，只有通过深度测试的点才会经过片段着色器的洗礼。片段着色器进行光照结束之后，这些值会写入到我们的帧缓冲区或者我们的RenderTarget渲染目标上。这基本上就是GPU渲染流水线的整个流程。
  - Output: image (pixels)

Shader Programs

- More Shader:
  - Geometry Shader
  - Compute Shader

## 着色模型

- flat shading: 每个三角形的法线是一样的，shading一次获得颜色值
  - 效果不好
- Gouraud Shading
  - 每个顶点做一次Shading，中间对颜色值插值
- Phong Shading
  - 对法线值做插值，对每个点做Shading
  - Not the Blinn-Phong Reﬂectance Model

几何足够复杂的情况下，用简单的Shading方法也可以达到好的效果

好的效果一般需要大的计算量

顶点法线方向的计算：相邻面的法线的平均

插值：Barycentric interpolation

法线向量记得Normalize

### Blinn-Phong Reflectance 

Blinn-Phong Reflectance Model 光照模型 着色模型

- Diffuse

  - Lambertian (Diffuse) Shading

    - 和view的方向无关

    $$
    L_{d}=k_{d}\left(I / r^{2}\right) \max (0, \mathbf{n} \cdot \mathbf{l})
    $$

    - 几个变量：k_d, r, n, l

      k_d：漫反射系数

      r：光源与采样点的距离

      I：光强度

      n：着色点法向量

      L：LightDir

- Specular

  - View越靠近光反射方向——>半程向量越接近法向量

  ![image-20220504085559185](..\101Pic\3.png)

  - 几个变量：k_s, r, l, v, n, p

    k_s：镜面反射系数

    r：光源距这色点的距离

    I：光照强度

    v：ViewDir 

    n：着色点的法线向量

    p一般100～200，控制高光大小

- Ambient

  - 补充暗部细节，粗略模拟GI效果
    $$
    L_{a}=k_{a} I_{a}
    $$

  - 提升亮度
  - 真的：全局光照（GI）十分复杂

Bllin-phong光照计算公式：
$$
\begin{aligned} L &=L_{a}+L_{d}+L_{s} \\ &=k_{a} I_{a}+k_{d}\left(I / r^{2}\right) \max (0, \mathbf{n} \cdot \mathbf{l})+k_{s}\left(I / r^{2}\right) \max (0, \mathbf{n} \cdot \mathbf{h})^{p} \end{aligned}
$$

- h是半程向量

Shading is Local：对某个点进行计算，局部，只看自己，不管其他物体的存在

光的能量密度的 平方反比定律

- 点离光源的radiance

点离观察点的距离无关

## Texture Mapping 

纹理映射

- 三维物体表面都是二维的
- 纹理：图，有弹性，可以映射到表面
- uv：[0,1]^2
- 三角形每个顶点对应一个uv坐标
- 一张纹理可以使用多次
- 纹理本身设计可以无缝衔接→tilable
  - 一种方法：Wang Tiling

## 重心插值

Barycentric coordinates 重心坐标→插值

P为三角形ABC内部的一点，则他们满足如下关系：
$$
\begin{array}{r} P_(x, y, z)=\alpha A+\beta B+\gamma C \\ 
\alpha+\beta+\gamma=1 \end{array}
$$
系数的计算：面积比
$$
\begin{aligned} \alpha &=\frac{-\left(x-x_{B}\right)\left(y_{C}-y_{B}\right)+\left(y-y_{B}\right)\left(x_{C}-x_{B}\right)}{-\left(x_{A}-x_{B}\right)\left(y_{C}-y_{B}\right)+\left(y_{A}-y_{B}\right)\left(x_{C}-x_{B}\right)} \\ \beta &=\frac{-\left(x-x_{C}\right)\left(y_{A}-y_{C}\right)+\left(y-y_{C}\right)\left(x_{A}-x_{C}\right)}{-\left(x_{B}-x_{C}\right)\left(y_{A}-y_{C}\right)+\left(y_{B}-y_{C}\right)\left(x_{A}-x_{C}\right)} \\ \gamma &=1-\alpha-\beta \end{aligned}
$$
利用计算出的α，β，γ可以对P点的法线、texCoords、position等进行插值。

- Notice！！：barycentric coordinates are not invariant under projection!

投影前后的重心坐标可能会变化，所以需要在对应时间计算对应的重心坐标来做插值，不能随意复用！ 投影之后计算的重心坐标进行插值还需要进行投影矫正。

问题1: Texture Magnification 纹理太小怎么办 → 插值

- 纹理像素：texel
- 多个pixel映射到了同一个texel
- 解决：
  - Nearest
  - Bilinear
    - Bilinear 插值 lerp
    - 水平+竖直插值→双线性插值
    - 最近的四个点插值
  - Bicubic 双向三次插值
    - 周围16个点做三次插值
    - 运算量更大，结果更好

问题2: Texture Magnification 纹理太大怎么办

- 一个pixel对应了多个texel → 采样频率不足导致 摩尔纹+锯齿（走样）
- 解决：
  - Super Sampling
    - 太浪费！
    - 只需要得到一个范围内的平均值
  - Mipmap
    - overhead: 1/3
    - 如何选取使用Mipmap哪一层，邻pixel的映射uv之间的距离取2的对数
      - 计算出的D是integer 直接用就可以了
      - D不是integer就需要进行Trilinear Interpolation三线性插值
      - 分别在floor(D)和ceil(D)上做Bilinear Interpolation取颜色值之后再插值

## 纹理的应用

各种贴图 ---> 存储 + 范围查询

- General method to bring data to fragment calculations

- 环境光贴图：球面映射、八面体映射、cube map

- 记录微几何

  - Bump Mapping 凹凸贴图，Texture记录了高度移动
    - 不改变几何信息
    - 逐像素扰动法线方向
    - 高度 offset 相对变化，从而改变法线方向
    - 计算法线方向：切线的垂直方向
  - Displacement mapping 位移贴图
    - 输入相同（Texture与Bump Mapping可共用）
    - 改变几何信息，对顶点做位移
    - 相比上更逼真，要求模型足够细致，运算量更高

- 噪声

- 预计算着色

  - 环境遮蔽贴图
    - 计算好的环境光遮蔽贴图
    - 空间换时间

  - 三维体渲染

## Shadow mapping

步骤：

- Redner From Light
  - 来自光源的深度图 → shadow map
- Render from Eye
  - Project to light
    - 将视线中的可见点投射回光线中
    - 采样shadowmap中对应点的深度
    - blocked → shadow

问题：

- 走样、分辨率
- 数值精度问题
- 只能是点光源，面光源不好处理

常见解决方案：

​					自阴影—>bias

​					硬阴影—>PCF—>PCSS—>vsm

##  Curve

### Bézier Curves

  贝塞尔曲线

![image-20220504090356286](..\101Pic\5.png)



![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2c4a2b8-a096-4a3d-af04-22fab18b3d66/Untitled.png](..\101Pic\6.png)

![image-20220504090447424](..\101Pic\7.png)

计算方法：

![image-20220504090548677](..\101Pic\8.png)

一些性质：

![image-20220504090612533](..\101Pic\9.png)

高阶贝塞尔曲线很难控制，所以有分段贝塞尔曲线

Piecewise Bézier Curves

- [Bezier Curve Edit](http://math.hws.edu/eck/cs424/Document\Doc2013/canvas/bezier.html)
- chain many low-order Bézier curve
- 例如：Piecewise cubic Bézier
- 每次四个控制点
- 保证光滑（切线不突变）：内部控制点前后的切线点共线

![image-20220504090652251](..\101Pic\10.png)



## 曲面

### Bézier Surfaces

贝塞尔曲面

用两个维度的时间进行表示T(u,v)

### Mesh

Mesh Operations: Geometry Processing

- 网格细分 上采用
  - 提高分表率
- 网格简化 下采样
  - 降低分辨率
  - 尽量保持形状外观
- 网格正规化
  - 不会出现特别奇怪的三角形
  - 修改样本分布以提高质量

## Loop Subdivision

细分的应用场景

1：Displacement mapping 位移贴图 需要模型足够细致，于是需要细分（最好是动态细分）

需要三角形Mesh

步骤：

1. 创建更多三角形，更多顶点，把每个三角形分成4个
2. 改变三角形的形状，根据权重指定新的顶点位置，新/旧顶点更新方式不同，即新老点分别改变

![image-20220504090804110](..\101Pic\11.png)

![image-20220504090849064](..\101Pic\12.png)

## Catmull-Clark Subdivision (General Mesh)

通用Mesh

Non-quad face：非四边的面

Extraordinary vertex (奇异点)：指(degree != 4)的点

Each subdivision step:

1. Add vertex in each face
2. Add midpoint on each edge
3. Connect all new vertices

![image-20220504091639657](..\101Pic\13.png)

奇异点在第一次细分增加[非四边形面数量]，后续不变；所有非四边形面在第一次细分都会消失

Update Rules (Quad Mesh)：

![image-20220504091729905](..\101Pic\14.png)

## 网格简化

目标：减少网格数量的同时保持整体的形状

应用：移动端、远距离

几何的层次结构

Edge Collapse：顶点合并

- 哪些边合并？如何合并？
  - Quadric Error Metrics，放在二次误差之和最小的地方

Simplification via Quadric Error

- 有问题，一条边的操作会影响其它边，需要更新
  - 数据结构：优先队列 or 堆
- 贪心算法，非全局最优
- 可以有的放矢



## Ray Tracing 

光线追踪 (Whitted Style)

为什么要Ray Tracing?

- 光栅化很难处理全局光照
  - Soft Shadows、Glossy reflection、Indirect Illumination
- 光栅化：快速近似、质量低
- 光线追踪：准确、非常慢

光线追踪相关的假设：

- 假设近似直线传播
- 光线交叉互不影响
- 从光源到人眼～从人眼到光源（光路可逆性）

## Whitted-Style Ray Tracing

Ray Casting:

1. 通过每个像素投射一条光线来生成图像

   从眼睛出发，经过像素点，射向场景

   判断最近碰撞物体，Shading计算颜色

2. 通过向灯光发射光线来检查是否有阴影

![image-20220504091902060](..\101Pic\15.png)

光线弹射多次：递归(Whitted-Style) Ray Tracing

- Shading：每次折射点都计算一次颜色值，最后累加

![image-20220504091933797](..\101Pic\16.png)

Ray equation射线方程:
$$
\mathbf{r}(t)=\mathbf{o}+t \mathbf{d} \quad 0 \leq t<\infty
$$
Ray Intersection:

- with Sphere （c: center, R: 半径）
  $$
  (\mathbf{o}+t \mathbf{d}-\mathbf{c})^{2}-R^{2}=0 
  $$
  
- 推广：隐式表面Geometry
  $$
  \begin{aligned} &\mathbf{p}: f(\mathbf{p})=0\\ &f(\mathbf{o}+t \mathbf{d})=0 \end{aligned}
  $$
  
- 显式表面 - Triangle Mesh

  - 计算每个三角形的相交

    - 太慢了
    - 加速：kdtree, Bounding Box

  - 计算：光线与平面求交+交点是否在三角形内

    - 平面方程与光线方程组合，求t，确定为正实数

  - Möller Trumbore Algorithm: 计算交点

    ![image-20220504092056919](..\101Pic\17.png)

    如与平面有交点：t为正

    判断在三角形内：重心坐标三个系数在[0,1]

## Accelerating Ray-Surface Intersection

原始：每根光线和每个三角形求交（太慢！）

加速：包围盒kdtree

Bounding Volumes 包围盒

- bound complex object with a simple volume ,简单体积包裹复杂对象
- 碰不到包围盒，就肯定碰不到里面的物体

AABB包围盒

- 三对面形成的交集 xmin, xmax, ymin, ymax, zmin, zmax
- 判断求交：光线与三对面的交点：三组(tmin,tmax) 如果有交集（即有同时在三对面内的时间），则与盒子有交
- 原理：The ray enters the box only when it enters all pairs of slabs The ray exits the box as long as it exits any pair of slabs
- If t enter < t exit , we know the ray stays a while in the box (so they must intersect!)
- 需要检查各个t是否为正
- tenter = max{tmin}, texit = min{tmax}
- ray and AABB intersect iﬀ (t enter< t exit && t exit >= 0)
- 垂直时计算更简单：每个t只需要一个`-`和一个`/`

How to use AABB to accelerate ray tracing?

- Uniform grids 均一网格
- Spatial partitions 空间划分

## Spatial Partitions - KD Tree

Oct-tree 八叉树（三维均匀切分）（与维数有关）

**KD-Tree 每次只沿某一个轴划分 二叉树like**

BSP-Tree 每次取一个方向（非横平竖直）将空间分为两部分 （会很麻烦）

KD-Tree Pre-Processing

- 划分空间，存入二叉树

- 内部节点存储以下信息

  split axis: x-, y-, or Z-axis split position: coordinate of split plane along axis children: pointers to child nodes

- 叶节点存储

  list of objects

Traversing a KD-Tree

- 光线从根结点开始向下遍历，若有交点则深入（可能与其子节点都有交点，继续判断），若无交点则离开（不可能与其子节点有交点），碰到叶子节点则与其中所有物体求交

问题1：三角形与Bounding Box的包含情况求解困难（可能出现三角形三个顶点都不在Box内，三角形却有一部分在Box内的情况）

最近渐渐不用KD-Tree了，上述原因为其中一个原因

问题2：一个物体可能存在于多个叶子节点中

## Object Partitions & Bounding Volume Hierarchy (BVH)

以object为单位划分空间

二叉树，两个子节点分别存两部分物体的AABB

一个物体只可能出现在一个包围盒中

如何划分很有讲究，不好的划分会使包围盒重合，降低效率

1. Find bounding box

2. Recursively split set of objects in two subsets

   - Choose a dimension to split
   - Heuristic #1: Always choose the longest axis in node
   - Heuristic #2: Split node at location of median object(中位数）
     - 中位数

3. Recompute the bounding box of the subsets

4. Stop when necessary

   Heuristic: stop when node contains few elements (e.g. 5)

5. Store objects in each leaf node

Internal nodes store

- Bounding box
- Children: pointers to child nodes

Leaf nodes store

- Bounding box
- List of objects

Nodes represent subset of primitives in scene

- All objects in subtree

动态场景不可

Traversal:

- 如果miss，直接返回
- 如果已经到了叶子节点，和其中的object求交找最近
- 如果hit，和下面两个子节点求交，返回最近

Spatial vs Object Partitions

- 后者目前更多应用



# 基础物理量

- Radiant energy（辐射能量）: 电磁辐射的能量
  - 符号：Q
  - 单位：J（Joule焦耳）
- Radiant flux (power) (辐射通量 功率): 单位时间辐射、接收的能量
  - 符号：ϕ （phi）
  - 单位：W（Watt），lm（lumen流明）
  - 单位时间 很重要
- Flux（单位时间内通过传感器的光子数）：单位时间内通过传感器的光子数

重要的光线测量

- Radiant(luminous) **Intensity** （辐射强度）:单位立体角的功率，由点光源发射。

  - 符号定义：

  $$
  I(\omega) = \frac{\mathrm{d} \Phi}{\mathrm{d} \omega}
  $$

  - 单位：W/sr，lm/sr=cd(candela坎德拉)

  - 立体角

    - 弧度：弧长/半径
    - 弧度在三维的延伸→立体角：面积/半径^2，

    $$
    \Omega=\frac{A}{r^{2}}
    $$

    - 单位：sr, steradians

    - 球面=4pi sr
      $$
      \mathrm{d} \omega=\frac{\mathrm{d} A}{r^{2}}=\sin \theta \mathrm{d} \theta \mathrm{d} \phi
      $$

  - 方向性

    - 用 w 来表示方向

  - 点光源：

  $$
  I = \frac{\phi}{4\pi}
  $$

- Irradiance（辐照度）: 单位面积功率（垂直投影）

  - 落在表面上的光能量

  $$
  E(\mathbf{x}) \equiv \frac{\mathrm{d} \Phi(\mathbf{x})}{\mathrm{d} A}
  $$

  - 单位：W/m^2 | lm/m^2 = lux
  - 需要注意，是垂直方向
  - 无方向性

- Radiance（辐射度）:单位立体角的功率，单位投影面积的功率.

  - describes the distribution of light in an environment
  - Light Traveling Along A Ray

  $$
   L(\mathrm{p}, \omega) \equiv \frac{\mathrm{d}^{2} \Phi(\mathrm{p}, \omega)}{\mathrm{d} \omega \mathrm{d} A \cos \theta} 
  $$

  

  - 某个单位面积往某个单位立体角辐射的能量密度

  - Radiance: 每个立体角的辐照度

  - Radiance: **Intensity** per projected unit area

  - 有方向性

  - Incident radiance is the irradiance per unit solid angle arriving at the surface. 某个小面接受来自某个方向的光线
    $$
    L(\mathrm{p}, \omega)=\frac{\mathrm{d} E(\mathrm{p})}{\mathrm{d} \omega \cos \theta} 
    $$
    
  - Exiting surface radiance is the intensity per unit projected area leaving the surface. 某个小面发出向某个方向的光线
    $$
     L(\mathrm{p}, \omega)=\frac{\mathrm{d} I(\mathrm{p}, \omega)}{\mathrm{d} A \cos \theta} 
    $$
    

Irradiance和Radiance之间的区别就在于是否有方向性

- Irradiance: total power received by area dA 四面八方的光线积分起来
- Radiance: power received by area dA from “direction” dω

$$
\begin{aligned} d E(\mathrm{p}, \omega) &=L_{i}(\mathrm{p}, \omega) \cos \theta \mathrm{d} \omega \\ E(\mathrm{p}) &=\int_{H^{2}} L_{i}(\mathrm{p}, \omega) \cos \theta \mathrm{d} \omega \end{aligned} 
$$



- H^2：半球面

## BRDF 

双向反射分布函数：BRDF描述了从某个方向入射到一个点上的光线的能量会怎么反射，在不同的反射方向上会各分布多少能量

反射的理解：光线打到某个点，（被吸收了）然后反弹（发出）到其他地方

Radiance from direction ωi turns into the power E that dA（某个点） receives, Then power E will become the radiance to any other direction ωr

![image-20220504092738965](..\101Pic\18.png)

能量一般指功率，即单位时间内的能量。

某个点接受/发射光线总能量：Irradiance

某个点从某个方向接受/向某个方向发射光线能量：radiance

- Differential irradiance incoming:

$$
 d E\left(\omega_{i}\right)=L\left(\omega_{i}\right) \cos \theta_{i} d \omega_{i} 
$$



- Differential radiance exiting:

$$
d L_{r}\left(\omega_{r}\right) 
$$

BRDF的几个细节：

- 针对一个输入源
- 描述不同方向的输出
- 定义了不同材质

BRDF：represents how much light is reflected into each outgoing direction dL r (! r ) from each incoming direction

![image-20220504092829276](..\101Pic\19.png)
$$
 f_{r}\left(\omega_{i} \rightarrow \omega_{r}\right)=\frac{\mathrm{d} L_{r}\left(\omega_{r}\right)}{\mathrm{d} E_{i}\left(\omega_{i}\right)}=\frac{\mathrm{d} L_{r}\left(\omega_{r}\right)}{L_{i}\left(\omega_{i}\right) \cos \theta_{i} \mathrm{d} \omega_{i}}\left[\frac{1}{\mathrm{sr}}\right] 
$$


## The Reflection Equation 反射方程

- 针对一个输出源（着色点），积分所有方向输入源（光照）的BRDF，获得最后的输出

$$
 L_{r}\left(\mathrm{p}, \omega_{r}\right)=\int_{H^{2}} f_{r}\left(\mathrm{p}, \omega_{i} \rightarrow \omega_{r}\right) L_{i}\left(\mathrm{p}, \omega_{i}\right) \cos \theta_{i} \mathrm{d} \omega_{i}
$$

Challenge: Recursive Equation

- 不只有光源才是输入源，其他物体的反射光也有可能是输入源
- 递归的计算方式，计算量爆炸！

## The Rendering Equation 渲染方程（绘制方程）

**Rendering Equation (Kajiya 86) 跨时代**

Adding an Emission term to the reflection equation to make it general!
$$
 L_{o}\left(p, \omega_{o}\right)=L_{e}\left(p, \omega_{o}\right)+\int_{\Omega^{+}} L_{i}\left(p, \omega_{i}\right) f_{r}\left(p, \omega_{i}, \omega_{o}\right)\left(n \cdot \omega_{i}\right) \mathrm{d} \omega_{i}
$$


- 包括了物体自己会发光的情况，包括了所有光线的传播情况
- assume that all directions are pointing outwards!
- $H^2$和$\Omega^{+}$ 都代表半球
- 怎么解这个方程呢？下节课

单个点光源：

![image-20220504092921790](..\101Pic\20.png)

多个点光源：加起来

![image-20220504092951109](..\101Pic\21.png)

还有面光源：积分起来

![image-20220504093015831](..\101Pic\22.png)

考虑其他物体反射的光线（把光源也包括在内了）：→递归

![image-20220504093044614](..\101Pic\23.png)
$$
 L_{r}\left(x, \omega_{r}\right)=L_{e}\left(x, \omega_{r}\right)+\int_{\Omega} L_{r}\left(x^{\prime},-\omega_{i}\right) f\left(x, \omega_{i}, \omega_{r}\right) \cos \theta_{i} d \omega_{i} 
$$
这个方程是 Fredholm Integral Equation of second kind [extensively studied numerically] with canonical form → 简写为如下形式
$$
I(u)=\theta(u)+\int l(v)K(u,v)dv
$$
通过算符的抽象还可极度简化成如下形式：L = E + KL （K：反射算符）其中L为未知数

- Can be discretized to a simple matrix equation [or system of simultaneous linear equations] (L, E are vectors, K is the light transport matrix)

![image-20220504093156961](..\101Pic\24.png)

![image-20220504093213402](..\101Pic\25.png)

![image-20220504093235110](..\101Pic\26.png)

- 光栅化的着色过程只有直接光照（间接光照需要间接计算）
- 全局光照：直接和间接光照的集合
  - 会收敛到一个亮度

如何解全局光照方程？Monte Carlo Path Tracing (Lec 16)

## Probability Review 概率论回顾

- 随机变量$X$: 可能取很多数值的变量
- 随机变量的分布函数 $X \sim p(x)$: probability density function (PDF) 取不同数值的概率
  - 比如，uniform PDF, 骰子
  - 概率$p_i$的数值非负，和为1，注意和p(x)的概念不一样。是p(x)的输出。
- 期望：The average value that one obtains if repeatedly drawing samples from the random distribution. $E[X]=\sum_{i=1}^{n} x_{i} p_{i}$

连续情况下：

- 随机变量的分布函数 $X \sim p(x)$ 是连续的→**Probability Distribution Function (PDF)**
- 某一个x对应的概率 = p(x)dx （细长条）
- 概率密度函数的性质 $p(x) \geq 0$ and $\int p(x) d x=1$
- 期望：$E[X]=\int x p(x) d x$

随机变量的函数：

- 如果某个随机变量Y是随机变量X的函数：$Y=f(X)$
- 期望的关系：$E[Y]=E[f(X)]=\int f(x) p(x) d x$



